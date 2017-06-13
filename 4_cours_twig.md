Twig

Language de templating developpé par Sensio. Tellement puissant que des sociétés se montent sur cette techno revendent leur savoir faire.
Comme tout language de templating, il existe les boucles, les if/else etc... des variables qui sont passés aux vues.
Généralement vous utilisez Twig dans le rendu de vos controllers. Mais il s'utilise aussi très frequement lors de l'envoi de mail. Des mails complexes nescessitent parfois un templating adequate et on peut aussi utiliser Twig comment moteur de rendu de mail.

- Blocks  
Dans twig on fonctionne à base de block.

- Extends/Include/Embbed  
AppBundle/Controller/DefaultController  
indexAction
le nom de l'action doit appeller une vue du même nom. Sinon vous allez vite vous perdre.

```twig
{# Resource/views/default/index.html.twig #}

{% extends 'base.html.twig' %}

{% block body %}
    Hello World !
{% endblock %}
```

```twig
{# Resource/views/base.html.twig #}
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>{% block title %}Welcome!{% endblock %}</title>
        {% block stylesheets %}{% endblock %}
        <link rel="icon" type="image/x-icon" href="{{ asset('favicon.ico') }}" />
    </head>
    <body>
        {% block body %}{% endblock %}
        {% block javascripts %}{% endblock %}
    </body>
</html>
```


Je cree une nouvelle action
```php
    /**
     * @Route("/bye", name="bye")
     */
    public function byeAction(Request $request)
    {
        return $this->render('default/bye.html.twig');
    }
```
et ma vue
```twig
{# Resource/views/default/bye.html.twig #}

{% extends 'base.html.twig' %}

{% block body %}
    Bye World !
{% endblock %}
```


Si je veux changer World, je dois éditer 2 templates. Mauvaise solution ! Imaginez vous avez 200 vues !
Je cree donc un template world.
```twig
{# Resource/views/default/world.html.twig #}

{% block world %}World !{% endblock %}
```
et je l'include dans mes 2 templates initiaux.
```twig
{# Resource/views/default/index.html.twig #}

{% extends 'base.html.twig' %}

{% block body %}
    Hello {% include 'world.html.twig' %}
{% endblock %}
```
```twig
{# Resource/views/default/bye.html.twig #}

{% extends 'base.html.twig' %}

{% block body %}
    Bye {% include 'world.html.twig' %}
{% endblock %}
```

Une modification du client arrive et il veut desormais "Hello World !" et "By World ! See you soon !"
Dans ce cas là on va embbed le template world dans bye
```twig
{# Resource/views/default/bye.html.twig #}

{% extends 'base.html.twig' %}

{% block body %}
    Bye {% embed 'world.html.twig' %}
            {% block world %}{{ parent()}} See you soon !{% endblock %}
        {% endembed %}
{% endblock %}
```

Lors d'inclusion de template, on peut aussi passer des variables. Modifions le template world
```twig
{# Resource/views/default/world.html.twig #}

{% block world %}World !{{ textMore|default('') }}{% endblock %}
```
Les templates index et bye deviennent : 
```twig
{# Resource/views/default/index.html.twig #}

{% extends 'base.html.twig' %}

{% block body %}
    Hello {% include 'world.html.twig' %}
{% endblock %}
```
```twig
{# Resource/views/default/bye.html.twig #}

{% extends 'base.html.twig' %}

{% block body %}
    Bye {% include 'world.html.twig' with {textMore: ' See you soon !'} %}
{% endblock %}
```

- Les filtres/Fonctions
En plus de cette notion d'heritage et d'inclusion de template. Twig offres une plétore de fonction manipulant les variables ou permettant de simplifier son code.
Cela va du filtre qui tronque le texte, affiche les dates, interprete ou pas l'html...
https://twig.sensiolabs.org/doc/2.x/

- Les Extensions
Et si cela ne suffit pas, des extensions sont disponibles comme d'afficher les valeurs en toute lettres, localiser les dates, ...


More ? Vous pouvez coder vous meme vos fonctions et brique du twig. Et evidemment des dizaines de bundle rajoutent eux aussi leurs briques (obscurator)