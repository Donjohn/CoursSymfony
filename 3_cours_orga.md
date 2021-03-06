Comment organiser son code !

parler de la scruture des reps.

Il existe plusieurs manières : 
- AppBundle  
    c'est le bundle créé par default à l'initialisation du projet. Suffit dans le cas de très petits projets. Il disparaitra avec la version 4 de Symfony.
- Back/Front  
    C'est l'organisation logique au premier abord et surtout hérité de méthode plus ancienne de développement (avec un répertoire /admin). Sauf que ça viole une règle Symfony : Un bundle doit se suffire à lui-même.  
    Dans le cas de l'organisation BackBundle et FrontBundle, se pose la question de savoir ou mettre les entités, ou mettre les services, etc. On se retrouve vite avec un des 2 bundle qui a quasiment tout le code métier et un autre bundle quasi vide. Ou 2 bundles foutoirs et où la seule différence vient des Controller/views...
- Par fonctionnalité  
    C'est peut-être la plus complexe mais c'est la plus pratique sur le long terme. Un bundle = une fonctionnalité/partie du site.
    
    Ex : Site Jeux Video  
    - JeuVideo
    - Users
    - Forum
    - eShop
    - Actu
    - Dossiers
      
    Ou placer mes entités JeuVideo que je veux présenter en front mais que je veux alimenter par mon back ?
    Dans le cas 2, j'en ai autant besoin dans l'un et l'autre bundle.
    La bonne solution est de faire un bundle qui possède l'admin et le front, donc les controllers, formulaires ainsi que tous les assets qui servent à afficher les JeuVideo
    
    Attention : le nom du répertoire doit être en accord avec le namespace de la classe.
```
src/
    JeuVideoBundle/
        /Controller
            JeuVideoAdminController.php
            JeuVideoController.php
        /Entity
            JeuVideo.php
        /Repository
            JeuVideoRepository.php
        /Resources
            /public
                /css
                    admin.css
                    front.css
                /images
                    background.jpg
                /js
                    effects.js
            /views
                /JeuVideoAdmin
                    index.html.twig
                    edit.html.twig
                    list.html.twig
                    delete.html.twig
                /JeuVideo
                    index.html.twig
                    list.html.twig
                    search.html.twig
        JeuVideoBundle.php
    UserBundle/
    ForumBundle/
    ShopBundle/
    NewsBundle/
    DocuBundle/
    CoreBundle/
        /Manager/
            BaseManager.php
        /Listener
            MailListener.php
        /Resources            
            /views
                layout.html.twig
```
 



Ou placer ses assets dans les bundle !  
`Resource/public/js|images|css`  

Les assets font parties de vos sources, elle ne doivent pas etre disponibles directement mais servit.
On peut tout mettre dans web, mais niveau sécu, ca laisse une faille. Les droits sur le répertoire web ne sont pas les mêmes que dans le répertoire src. Souvent parce que vous autorisez l'upload de fichier par ex.
Vous protégez vos sources sur le serveur en les mettant autre part que dans /web, donc dans /src et c'est le front_controller (app.php) qui va exécuter symfony. Il faut donc le même mécanisme pour les assets : les protéger dans src et les servir dans web

Symfony possède un mécanisme qui va mettre en ligne les assets dans web. C'est la commande  
`php bin/console assets:install web --symlink`


Mais now ya Gulp/Node/etc... donc on gere directement dans web