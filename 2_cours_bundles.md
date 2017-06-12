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

Les bundles importants: 
- https://symfony.com/doc/bundles/
- FOSUserBundle
  Premet la gestion des utilisateurs stockes dans la base de données 
- SonataAdminBundle / EasyAdminBundle
  Je prefere EasyAdmin mais j'ai beaucoup utilisé SonataAdmin
- StofDoctrineExtensionsBundle
  Permet d'automatiser des comportements sur les entités
- LiipImagineBundle
  Permet de gérer les images en rajoutant des filtres
- IvoryCKEditorBundle	
  Intégre le fameux CKEditor à Symfony
- KnpMenuBundle
  Gestion de menus
  
  
FOSUSER en détail :
    Installation : 
    