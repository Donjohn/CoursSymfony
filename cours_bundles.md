- librairies != bundle  

 Une bundle est une librairie php dédiée à Symfony2/3.  
 Guzzle est une librairie php qu'on peut utiliser dans n'importe quel projet php comme du Laravel  
 
jolicode/GifExceptionBundle


Les bundles importants: 
- https://symfony.com/doc/bundles/
- FOSUserBundle  
  Permet la gestion des utilisateurs stockés dans la base de données. Il offre tout ce qu'il faut pour identifier, authentifier, valider, attribuer des droits à des utilisateurs et stocke le tout dans la base de donnée. Ce qui n'est pas par default dans Symfony.
  C'est un bundle essentiel, on l'utilise quasiment dans tous les projets. C'est assez rare de ne pas gérer de liste d'utilisateurs.
- SonataAdminBundle / EasyAdminBundle  
  Je préfère EasyAdmin mais j'ai beaucoup utilisé SonataAdmin. Les commandes CrudGenerator permet de faire à minima le travail pour obtenir de quoi éditer/lister/supprimer les entités. Mais Vous ne voulez pas refaire l'interface complète à chaque fois. Ces bundles offrent des moyens rapides (moins pour Sonata) d'avoir un backend rapide, responsive, et qui offre toutes les options nécessaires. 
- StofDoctrineExtensionsBundle  
  Permet d'automatiser des comportements sur les entités. Timestampable, Traduction, ...
- LiipImagineBundle  
  Permet de gérer les images en rajoutant des filtres. Très important dans le cas où vous devez afficher des images sur votre site. Vous ne voulez pas gérer les 10 formats nécessaires...
- IvoryCKEditorBundle      
  Intègre le fameux CKEditor à Symfony
  Remplace les Textarea par un éditeur de texte complet.
- KnpMenuBundle  
  Gestion de menus, permet de simplifier la création, la navigation dans les menus que vous créez sur votre site.
  
  
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
                csrf_token_generator: security.csrf.token_manager
            logout:       true
            anonymous:    true

    access_control:
        - { path: ^/login$, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/register, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/resetting, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/admin/, role: ROLE_ADMIN }
````
Identification != Authentification. Pourtant ça se gère au même endroit car les 2 définissent un firewall.
Dans cet exemple, 'main' est notre firewall. On indique à SF d'activer ce firewall sur toutes les urls qui respectent la pattern ^/

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
                csrf_token_generator: security.csrf.token_manager
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
Des lors qu'il ya un firewall, il faut un User afin de connaitre ses droits.
Vous pouvez accepter les Anonymous User comme dans l'exemple, son rôle est automatique et fournit par Symfony.
Une personne non loggué peut donc acceder à l'url /login située derrière le firewall main car on a accepté les anonymous et que la page /login demande un des rôles les plus bas de la hierarchie : IS_AUTHENTICATED_ANONYMOUSLY

Derniere étape, recuperer les routes fournies par FOSUser
```yaml
# app/config/routing.yml
fos_user:
    resource: "@FOSUserBundle/Resources/config/routing/all.xml"
```
Ouvrons le, il fournit bcp de pages et même une page de profil de l'utilisateur !!


### Surcharge
1) héritage de bundle
2) juste les assets
3) dans la config

