** Composer **
 - creer un projet  
 Télécharger le script https://symfony.com/doc/current/setup.html    
 `php symfony new Folder <version>`
 - on ouvre le fichier composer.json
 ```json
 {
    "name": "cours/demo",
    "license": "proprietary",
    "type": "project",
    "autoload": {
        "psr-4": {
            "": "src/"
        },
        "classmap": [
            "app/AppKernel.php",
            "app/AppCache.php"
        ]
    },
    "autoload-dev": {
        "psr-4": {
            "Tests\\": "tests/"
        },
        "files": [
            "vendor/symfony/symfony/src/Symfony/Component/VarDumper/Resources/functions/dump.php"
        ]
    },
    "require": {
        "php": ">=5.5.9",
        "doctrine/doctrine-bundle": "^1.6",
        "doctrine/orm": "^2.5",
        "incenteev/composer-parameter-handler": "^2.0",
        "sensio/distribution-bundle": "^5.0.19",
        "sensio/framework-extra-bundle": "^3.0.2",
        "symfony/monolog-bundle": "^3.1.0",
        "symfony/polyfill-apcu": "^1.0",
        "symfony/swiftmailer-bundle": "^2.3.10",
        "symfony/symfony": "3.3.*",
        "twig/twig": "^1.0||^2.0"
    },
    "require-dev": {
        "sensio/generator-bundle": "^3.0",
        "symfony/phpunit-bridge": "^3.0"
    },
    "scripts": {
        "symfony-scripts": [
            "Incenteev\\ParameterHandler\\ScriptHandler::buildParameters",
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::buildBootstrap",
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::clearCache",
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::installAssets",
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::installRequirementsFile",
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::prepareDeploymentTarget"
        ],
        "post-install-cmd": [
            "@symfony-scripts"
        ],
        "post-update-cmd": [
            "@symfony-scripts"
        ]
    },
    "config": {
        "sort-packages": true
    },
    "extra": {
        "symfony-app-dir": "app",
        "symfony-bin-dir": "bin",
        "symfony-var-dir": "var",
        "symfony-web-dir": "web",
        "symfony-tests-dir": "tests",
        "symfony-assets-install": "relative",
        "incenteev-parameters": {
            "file": "app/config/parameters.yml"
        },
        "branch-alias": null
    }
} 
 ```
 - il faut savoir lire les numéros de version
  `"symfony/monolog-bundle": "^3.1.0",` indique que l'on veut le package symfony/monolog-bundle (gestion des logs) AU MOINS en version 3.1.0. S'il existe une version 4, elle sera installé  
  `"symfony/symfony": "3.3.*",` et plus précis. On demande la version 3.3.* uniquement. Si une version 3.4 de Symfony sort, vous etes assuré de ne pas upgrader le core de Symfony.
  
 - gestion de la version de php, permet d'affiner la selection des packages. Car c'est aussi un des objectifs de composer : télécharger les packages compatibles entre eux
 ```
    "config": {
        "platform": {
            "php": "5.6.25"
        }
    }
```
 - install / update / require
    - install  
     Cette commande permet d'installer les librairies telles qu'elle ont été declarés dans le fichier composer.lock (on y revient)
    - update  
    Elle premet de demander à composer s'il existe des nouvelles versions des librairies tout en respectant les version marquées dans le fichier composer.json
    - require  
    Permet d'ajouter une librairies à composer.json ou de spécifier une version
    - remove
    Permet de supprimer une librairie. (Ne prends aps de numero de version)
 ex :On ne veut pas de Symfony en 3.3 on veut revenir en Symfony 3.2
 `composer require symfony/symfony 3.2.*`
   
 On ouvre à nouveau le fichier composer.json et la ligne a changé 
    `"symfony/symfony": "3.2.*"`
 
 Par contre si je mets "^3.2.*" il me téléchargera la derniere version (3.3.2)

 - flux de developpement dans une equipe  
     - Probleme
         - si tout le monde install les packages dans son coin. 
         - le fichier composer.json comporte des indications de version minimum le plus souvent. Il n'est pas tjs judicieux de mettre à jour les librairies sans le vouloir.
         
 Pour resoudre ces soucis, il y a le fichier composer.lock
 Il contient les informations de toutes les librairies installées via composer. Ce fichier fige les versions installées au dernier update.   
    - le lead met à jour les librairies (composer update|require <package_name>)  
    - il commit le fichier composer.lock  
    `git commit -m "update composer.lock" composer.lock`    
    - les autres devs pull ce fichier depuis le repo et lance simplement composer install et uniquement install. Si vous faites un update, prévénez les autres ou faites un git revert ;)
    `git pull`  
    `composer install`  
    - souvent au pull d'un projet, il est usuel de faire un composer install, surtout si il y a composer.lock de modifié dans les sources recupérées
    
 - il est souvent (voir toujours) néscessaire pour un developpement de charger une librairie supplémentaire    
 `composer require guzzlehttp/guzzle`  
  Cela ajoute la librairie au fichier composer.json, verifie les dependances et procède à l'installation si tout va bien. Ici, On n'a pas précisé de version. On laisse composer trouver la derniere version compatible avec mon composer.json.
  (Guzzle est une librairie php qui offre des clients de connection)
 
 
 - librairies != bundle  
 Une bundle est une librairie php dédié à Symfony2. Guzzle est une librairie php qu'on peut utiliser dans n'importe quel projet php comme du Laravel  
 symfony/assetic-bundle est quant à elle spécifique à Symfony. Elle s'installe pourtant de la même manière.
 ```
composer require symfony/assetic-bundle
    Using version ^2.8 for symfony/assetic-bundle
    ./composer.json has been updated
    Package operations: 2 installs, 0 updates, 0 removals
      - Installing kriswallsmith/assetic (v1.4.0) 
      - Installing symfony/assetic-bundle (v2.8.1) 
    ...
    Writing lock file
    Generating autoload files
    ...
```
 Si on observe ce que fait composer il télécharge en fait 2 librairies kriswallsmith/assetic en version 1.4 et symfony/assetic-bundle en 2.8 alors qu'on a demandé un seul package.
 Ce sont les dépendances du dit packages. Lui meme est capable d'indique à composer quelle sont ses propres dépendances. Dans le cas present la librairie kriswallsmith/assetic, utilisable dans n'importe quel projet php. Pour la rendre compatible et l'utiliser dans symfony de manière transparente, on utilise le bundle qui l'intègre pour nous.
 C'est un procédé très frequent.
 
 Notre bundle n'est tjs pas utilisable dans Symfony. Il va falloir l'ajouter au moteur, le kernel.
 Ouvrez le fichier app/AppKernel.php et ajoutez la ligne  
 `new Symfony\Bundle\AsseticBundle\AsseticBundle(),` à la liste des bundles utilisés.
  Certain bundle ont besoin de configuration pour etre opérationnel. C'est le cas d'assetic, ouvrez le fichier app/config/config.yml et ajoutez
  ```yaml
assetic:
    debug:          '%kernel.debug%'
    use_controller: '%kernel.debug%'
    filters:
        cssrewrite: ~
```
 
  
 Conclusion : 
 - jamais d'update globale sans savoir ce que vous faites.
 - tjs composer install apres un pull du projet
 - on install des packages avec composer install ou composer remove uniquement.
 - on lit le README de la librairies pour l'installation !!!
 
 