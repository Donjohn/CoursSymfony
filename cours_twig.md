Twig

Language de templating developpé par Sensio. Tellement puissant que des sociétés se montent sur cette techno et revendent leur savoir-faire.
Comme tout language de templating, il existe les boucles, les if/else etc... ou des variables qui sont passés aux vues.
Généralement vous utilisez Twig dans le rendu de vos controllers. Mais il s'utilise aussi très fréquemment lors de l'envoi de mail. Des mails complexes nécessitent parfois un templating adéquate et on peut aussi utiliser Twig comment moteur de rendu de mail.

- Blocks  
Dans twig on fonctionne à base de block.

- Nommage  
```
AppBundle/Controller/DefaultController::indexAction  
AppBundle/Resources/view/Default/index.twig.html
```
le nom de l'action doit appeler une vue du même nom. Sinon vous allez vite vous perdre.

- Extends / Include / Embbed  
Je créé une action simple qui affiche Hello World !
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
```php
    /**
     * @Route("/", name="home")
     */
    public function indexAction(Request $request)
    {
        return $this->render('default/index.html.twig');
    }
```


Je crée une nouvelle action byeAction
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


Si je veux changer World, je dois éditer 2 templates. Mauvaise solution ! Imaginez-vous avez 200 vues !
Je crée donc un template world.
```twig
{# Resource/views/default/world.html.twig #}

{% block world %}World !{% endblock %}
```
et je l'include dans mes 2 templates initiaux.
```twig
{# Resource/views/default/index.html.twig #}

{% extends 'base.html.twig' %}

{% block body %}
    Hello {% include 'default/world.html.twig' %}
{% endblock %}
```
```twig
{# Resource/views/default/bye.html.twig #}

{% extends 'base.html.twig' %}

{% block body %}
    Bye {% include 'default/world.html.twig' %}
{% endblock %}
```

Une modification du client arrive et il veut désormais "Hello World !" et "Bye Planet !"
Dans ce cas-là on va embbed le template
```twig
{# Resource/views/default/bye.html.twig #}

{% extends 'base.html.twig' %}

{% block body %}
    Bye {% embed 'default/world.html.twig' %}
            {% block world %}Planet !{% endblock %}
        {% endembed %}
{% endblock %}
```

Lors d'inclusion de template, on peut aussi passer des variables. Modifions le template world pour ajouter une phrase
```twig
{# Resource/views/default/world.html.twig #}

{% block world %}{{ message|default('World !') }}{% endblock %}
```
Les templates index et bye deviennent : 
```twig
{# Resource/views/default/index.html.twig #}

{% extends 'base.html.twig' %}

{% block body %}
    Hello {% include 'default/world.html.twig' %}
{% endblock %}
```
```twig
{# Resource/views/default/bye.html.twig #}

{% extends 'base.html.twig' %}

{% block body %}
    Bye {% include 'default/world.html.twig' with {message: ' Planet !'} %}
{% endblock %}
```

- Les filtres/Fonctions
En plus de cette notion d'héritage et d'inclusion de template. Twig offres une pléthore de fonctions manipulant les variables ou permettant de simplifier son code.
Cela va du filtre qui tronque le texte, affiche les dates, interprète ou pas l'html...
https://twig.sensiolabs.org/doc/2.x/

- Les Extensions
Et si cela ne suffit pas, des extensions sont disponibles comme afficher les valeurs en toute lettres (1000 écrit mille), localiser les dates, ...


More ? Vous pouvez coder vous même vos fonctions et brique du twig. Et évidemment des dizaines de bundle rajoutent eux aussi leurs briques (obscurator Email par ex)

