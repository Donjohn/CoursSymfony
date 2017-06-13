Comment organiser son code !
  
/src

- AppBundle  
    c'est le bundle créé par default à l'initialisation du projet. Suffit dans le case de tres petites projets
- Back/Front  
    C'est l'organisation logique au premier abord. Sauf que ca viole un règle Symfony : Un bundle doit se suffir à lui même.  
    Dans le case de l'organisation BackBundle et FrontBundle, se pose la question de savoir ou mettre les entités, ou mettre les services, etc.. On se retrouve vite avec un des 2 bundle qui a quasiement tout le code métier et un auter bundle quasi vide. Ou 2 bundles foutoirs et où la seule difference vient des controllers/views...
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
```
    
(ouvrir GFI et montrer l'arborescence et expliquer rapidement)


Ou placer ses assets dans les bundle !  
`Resource/public/js|images|css`  

Les assets font parties de vos sources, elle ne doivent pas etre disponibles directement mais servit.
On peut tout mettre dans web, mais niveau sécu, ca laisse une faille. Les droits sur le repertoire web ne sont pas les meme que dans le repetoire src. Souvent parce que vous autorisez l'upload de fichier par ex.
Vous protégez vos sources sur le serveur en les mettant autre part que dans /web, donc dans /src et c'est le front_controller (app.php) qui va executer symfony. Il faut donc le même mecanisme pour les assets : les protéger dans src et les servir dans web

Symfony possède un mecanisme qui va mettre en ligne les assets dans web. C'est la commande  
`php bin/console assets:install web --symlink`


Si je place mes fichiers dans AppBundle/Resources/public/css et mes images dans AppBundle/Resources/public/css, si je ne veux pas que mon ide m'indique une erreure je dois ecrire dans mes css :  
```css
.background {
    background: url('../images/background.jpg');
}
```
 
Par contre, si je demande '../images/background.jpg' depuis le le front controller, il ne trouvera pas l'image.  
Car apres mise à disposition des assets dans le repertoire web leurs urls ne sont plus 
`../images/background.jpg` mais `bundles/app/images/background.jpg`  
Elles ne sont plus servies du meme endroit, et donc elles n'ont plus la meme url
C'est le job d'AsseticBundle grace au filtre cssrewite, qui va corriger le chemin des assets lorsqu'il va traiter les css lors des appels coté front controller.