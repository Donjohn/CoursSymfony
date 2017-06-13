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
 Ce sont les dépendances du dit packages. Lui meme est capable d'indiquer à composer quelle sont ses propres dépendances. Dans le cas present la librairie kriswallsmith/assetic, utilisable dans n'importe quel projet php. Pour la rendre compatible et l'utiliser dans symfony de manière transparente, on utilise le bundle qui l'intègre pour nous.
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
Assetic est un gestionnaire de css et js. Tout comme gulp/grunt et d'autres. Depuis la 2.8 il a été retiré car il n'est pas indispensable. Néanmoins, historiquement il continue d'être très utilisé notamment pour changer les chemins des fichiers dans vos css lors de la mise en ligne de votre application. 

**_Exo : installer doctrine/migrations-bundle_**
`php bin/console doctrine:database:create`
`php bin/console doctrine:migration:generate`

Je teste mon projet rapidement en cleanant le cache
`php bin/console ca:cl`

Les bundles importants: 
- https://symfony.com/doc/bundles/
- FOSUserBundle  
  Premet la gestion des utilisateurs stockés dans la base de données. Il offre tout ce qu'il faut pour identifier, authentifier, valider, attribuer des droits à des utilisateurs et stocke le tout dans la base de donnée. Ce qui n'est pas par default dans Symfony.
  C'est un bundle essentiel, on l'utilise quasiement dans tous les projets. C'est assez rare de ne pas gérer de liste d'utilisateurs.
- SonataAdminBundle / EasyAdminBundle  
  Je prefere EasyAdmin mais j'ai beaucoup utilisé SonataAdmin. Les commandes CrudGenerator permet de faire a minima le travail pour obtenir de quoi editer/lister/supprimer les entités. Mais Vous ne voulez pas refaire l'interface complete à chaque fois. Ces bundl offrent des moyens rapides (moins pour Sonata) d'avoir un backend rapide, reponsive, et qui offre toutes les options nescessaires. 
- StofDoctrineExtensionsBundle  
  Permet d'automatiser des comportements sur les entités. Timestampable, Traduction, ...
- LiipImagineBundle  
  Permet de gérer les images en rajoutant des filtres. Tres important dans le cas ou vous devez afficher des images sur votre site. Vous ne voulez pas gérer les 10 formats nescessaires...
- IvoryCKEditorBundle  	
  Intégre le fameux CKEditor à Symfony
  Remplace les Textarea par un editeur de texte complet.
- KnpMenuBundle  
  Gestion de menus, permet de simplifier la creation, la navigation dans les menus que vous créez sur votre site.
  
  
FOSUSER en détail :
    Installation : 
    https://symfony.com/doc/current/bundles/FOSUserBundle/index.html  
    
    `composer require friendsofsymfony/user-bundle`  
    
    `new FOS\UserBundle\FOSUserBundle(),`  
```php
    //src/AppBundle/Entity/User.php
    
    namespace AppBundle\Entity;
    
    use FOS\UserBundle\Model\User as BaseUser;
    use Doctrine\ORM\Mapping as ORM;
    
    /**
     * @ORM\Entity
     * @ORM\Table(name="cours_user")
     */
    class User extends BaseUser
    {
        /**
         * @ORM\Id
         * @ORM\Column(type="integer")
         * @ORM\GeneratedValue(strategy="AUTO")
         */
        protected $id;
    }
```
```yaml
fos_user:
    db_driver: orm
    firewall_name: main
    user_class: AppBundle\Entity\User
    from_email:
            address: "cours@demo.com"
            sender_name: "Demo !"
```
`php bin/console doc:mig:diff`    
`php bin/console doc:mig:mig`  
 
- Le security.yml
```yaml
security:
    encoders:
        FOS\UserBundle\Model\UserInterface: bcrypt

    role_hierarchy:
        ROLE_ADMIN:       ROLE_USER
        ROLE_SUPER_ADMIN: ROLE_ADMIN

    providers:
        fos_userbundle:
            id: fos_user.user_provider.username

    firewalls:
        main:
            pattern: ^/
            form_login:
                provider: fos_userbundle
                csrf_provider: security.csrf.token_manager
            logout:       true
            anonymous:    true

    access_control:
        - { path: ^/login$, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/register, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/resetting, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/admin/, role: ROLE_ADMIN }
````
Identification != Authentification. Pourtant ca se gère au meme endroit car les 2 definissent un firewall.
main est notre firewall. On indique à SF d'activer ce firewall toutes les urls qui respectent la pattern ^/

Identification :
```yaml
    encoders:
            FOS\UserBundle\Model\UserInterface: bcrypt
    providers:
        fos_userbundle:
            id: fos_user.user_provider.username

    firewalls:
        main:
            form_login:
                provider: fos_userbundle
                csrf_provider: security.csrf.token_manager
            logout:       true
            anonymous:    true
```
Authentification :
```yaml
    role_hierarchy:
        ROLE_ADMIN:       ROLE_USER
        ROLE_SUPER_ADMIN: ROLE_ADMIN

    firewalls:
        main:
            pattern: ^/                

    access_control:
        - { path: ^/login$, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/register, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/resetting, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/admin/, role: ROLE_ADMIN }
```
Des lors qu'il ya un firewall, il faut un User afin de comparer ses droits avec la route en cours.
Vous pouvez accepter les Anonymous User comme dans l'exemple, son role est automatique et fournit par Symfony.
Une personne non loggué peut donc acceder à l'url /login située derriere le firewall main car on a accepté les anonymous et que la page /login demande un des roles les plus bas de la hierarchie : IS_AUTHENTICATED_ANONYMOUSLY

Derniere étape, recuperer les routes fournies par FOSUser
```yaml
# app/config/routing.yml
fos_user_security:
    resource: "@FOSUserBundle/Resources/config/routing/security.xml"

fos_user_profile:
    resource: "@FOSUserBundle/Resources/config/routing/profile.xml"
    prefix: /profile

fos_user_register:
    resource: "@FOSUserBundle/Resources/config/routing/registration.xml"
    prefix: /register

fos_user_resetting:
    resource: "@FOSUserBundle/Resources/config/routing/resetting.xml"
    prefix: /resetting

fos_user_change_password:
    resource: "@FOSUserBundle/Resources/config/routing/change_password.xml"
    prefix: /profile
```
Il fournit meme une page de profil de l'utilisateur !!