   

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