Installer [https://symfony.com/doc/current/setup.html] 
Installer [https://getcomposer.org/download/]
en fonction de votre OS.

```bash
php symfony new DemoDuJour
```

Quel IDE ? NetBeans ou PHPStorm (best)
Dans PHPStorm :
S'assurer que vous ayez les bons plugins.
Settings -> Plugins -> Browse Repositories
PHP Annotations
Symfony Plugin

Puis on créé le projet
New Project ->Source files are in a local directory, no Websever is yet configured
on sélectionne le répertoire DemoDuJour et on lui assigne le statut de Project Root
on en profite pour assigner le statut Resource Root à web et app/Resources
On exclue aussi directement /var et /vendor
Et on met le statut Test Sources à /tests
Finish


Dans les bulles de PHPStorm, on a normalement 2 alertes :
Composer et le plugin de PHPStorm. On active les 2, auto config si possible sinon on renseigne les settings si il y a besoin (le chemin de composer.phar est demandé parfois).

Le projet a créé un bundle AppBundle pour nous. En général, on crée ses propres bundle dans une application complexe pour organiser les fonctionnalités. Aujourd'hui pour faire simple on se sert d’AppBundle.


On édite le fichier app/config/parameters.yml et on renseigne les infos de la base de donnée (database_name: demodujour)

```bash
php bin/console doctrine:database:create
```



