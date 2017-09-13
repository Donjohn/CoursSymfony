Comment organiser son code !
  

Il existe plusieurs manieres : 
- AppBundle  
    c'est le bundle créé par default à l'initialisation du projet. Suffit dans le cas de tres petits projets. Il disparaitra avec la version 4 de Symfony.
- Back/Front  
    C'est l'organisation logique au premier abord et surtout hérité de méthode plus ancienne de developpement (avec un repertoire /admin). Sauf que ca viole une règle Symfony : Un bundle doit se suffire à lui même.  
    Dans le cas de l'organisation BackBundle et FrontBundle, se pose la question de savoir ou mettre les entités, ou mettre les services, etc.. On se retrouve vite avec un des 2 bundle qui a quasiement tout le code métier et un autre bundle quasi vide. Ou 2 bundles foutoirs et où la seule difference vient des controllers/views...
- Par fonctionnalité  
    C'est peut etre la plus complexe mais c'est la plus pratique sur le long terme. Un bundle = une fonctionnalité/partie du site.
    
    Ex : Site Jeux Video  
    - JeuVideo
    - Users
    - Forum
    - eShop
    - Actu
    - Dossiers
      
    Ou placer mes entitées JeuVideo que je veux présenter en front mais que je veux alimenter par mon back ?
    Dans le cas 2, j'en ai autant besoin dans l'un et l'autre bundle..
    La bonne solution est de faire un bundle qui possède l'admin et le front, donc les controllers, formulaires ainsi que tous les assets qui servent à afficher les JeuVideo
    
    Attention : le nom du repertoire doit être en accord avec le namespace de la classe.
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
 