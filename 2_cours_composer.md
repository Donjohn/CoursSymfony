** Composer **

=======
 
 - on ouvre le fichier composer.json
 ```json
 {
    "name": "jgn/demodujour",
    "license": "proprietary",
    "type": "project",
    "autoload": {
        "psr-4": {
            "AppBundle\\": "src/AppBundle"
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
        "symfony/symfony": "3.4.*",
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
    `"symfony/swiftmailer-bundle": "^2.3.10",` indique que l'on veut le package symfony/swiftmailer-bundle (gestion des logs) AU MOINS en version 2.3.10. S'il existe une version 3, elle sera installée.  
    Pour savoir ce qui a réellement été installé, on regarde la sortie écran lors de l'exécution de composer ou bien on ouvre le fichier composer.lock et on cherche le package en question.
    
    `"symfony/symfony": "3.4.*",` et plus précis. On demande la version 3.3.* uniquement. Si une version supperieur 3.4 (ex: 4.0) de Symfony sort, vous êtes assuré de ne pas upgrader le core de Symfony par erreur.
      
      `"twig/twig": "^1.0||^2.0"` permet de sélectionner un ou l'autre version, incompatibles entre elles, en fonction de votre environnement, d'autres librairies ou votre version php.
  
  
  
 - Réglages des paramètres  
    Par default, il utilise la version php de votre machine au moment où vous lancez la commande mais parfois il est nécessaire de travailler sur d'autres versions de php. On peut aussi demander à travailler sur des librairies en développement par ex.
    On modifie donc les réglages dans le fichier composer.json. Cela permet d'affiner la selection des packages. Car c'est un des objectifs de composer : télécharger les packages compatibles entre eux
 ```json
 {
    "config": {
        "platform": {
            "php": "7.1.3"
        }
    }
}
```

     
 - composer install / update / require  
 Ce sont les 3 commandes importantes de composer, elle servent à gérer les packages
    - update <packageName|optionnal>  
        Elle permet de demander à composer s'il existe des nouvelles versions des librairies tout en respectant les version marquées dans le fichier composer.json. Apres une création de projet il est sain de lancer cette commande au moins une fois.  
        `composer update`  
        Cette commande met à jour le fichier composer.lock qui sert de reference pour installer les packages voulus.
    - install  
     Cette commande permet d'installer les librairies telles qu'elles ont été déclarées dans le fichier composer.lock (on y revient)    
    - require <packageName> <version|optionnal>
    Permet d'ajouter une librairie à composer.json ou de spécifier une version
    - remove <packageName>
    Permet de supprimer une librairie. (Ne prends pas de numéro de version)
    
    ex :On ne veut pas de Twig en 2 on veut revenir en Twig en version 1  
 `composer require twig/twig ^1.0`
   
    On ouvre à nouveau le fichier composer.json et la ligne a changé 
    `"twig/twig": "^1.0"`
    
     Il est souvent (voir toujours) nécessaire pour un développement de charger une librairie supplémentaire
     `composer require jolicode/gif-exception-bundle`    
      Cela ajoute la librairie au fichier composer.json, vérifie les dépendances et procède à l'installation si tout va bien. Ici, On n'a pas précisé de version. On laisse composer trouver la dernière version compatible avec mon composer.json.      
     
     

 - flux de développement dans une équipe, les problèmes...
     - si tout le monde install les packages dans son coin. 
     - le fichier composer.json comporte des indications de version minimum le plus souvent. Il n'est pas tjs judicieux de mettre à jour les librairies sans le vouloir.
         
    Pour résoudre ces soucis, il y a le fichier composer.lock
    Il contient les informations de toutes les librairies installées via composer. Ce fichier fige les versions installées au dernier update.     
    - le lead met à jour les librairies (composer update|require <package_name>)  
    - il commit le fichier composer.lock  
    `git commit -m "update composer.lock" composer.lock`    
    - les autres devs pull ce fichier depuis le repo et lance simplement composer install et uniquement install. Si vous faites un update, prévenez les autres ou faites un git revert ;)  
    `git pull`  
    `composer install`  
        ```
        Installing dependencies (including require-dev) from lock file
        ```  
    Vous pouvez supprimer le répertoire vendor, tapez la commande et il installera exactement les versions du composer.lock et pas celles du composer.json qui sont "vagues".  
    
    Au pull d'un projet, il est usuel de faire un composer install, surtout si il y a composer.lock de modifié dans les sources récupérées
    
   
  
 Conclusion : 
 - jamais d'update globale sans savoir ce que vous faites.
 - tjrs composer install apres un pull du projet
 - on install des packages avec composer install ou composer remove uniquement.
 
 

