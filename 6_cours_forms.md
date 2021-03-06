### OneToMany et ManyToOne
on va créer une entité Catégorie.

```bash
php bin/console doctrine:generate:entity  
AppBundle:Category  
annotation
name
```

on va créer une entité VideoGame.

```bash
php bin/console doctrine:generate:entity  
AppBundle:VideoGame 
annotation
name
# quand vous est demandé les parametres de name mettez unique à true
```

On ne peut choisir qu'un seul type de mapping (yml, xml, annotation) par bundle et les annotations ont la priorité. Si une entité a des annotations de mapping doctrine les autres formes de mapping dans le bundle seront ignorées.

On édite la class AppBundle\Entity\VideoGame et on ajoute la relation avec Category et inversement
```php
//AppBundle\Entity/VideoGame

    /**
     * @var Category $category
     * @ORM\ManyToOne(targetEntity="AppBundle\Entity\Category")
     */
    private $category;
```


Il manque les getter et setters, on demande à doctrine de les faire pour nous. Je préfère car il rajoute les bons commentaires et surtout un return $this; sur les getters
```bash
php bin/console doc:gen:entities AppBundle
```

On applique les changements dans la base de donnée
```bash
php bin/console doctrine:schema:update --force
```
 





Maintenant on va générer un crud et manipuler les forms et les relations.
```bash
php bin/console generate:doctrine:crud
AppBundle:VideoGame
write action : yes


php bin/console generate:doctrine:crud
AppBundle:Category
write action : yes
```

on vérifie que tout marche
```bash
php bin/console server:run
```
Et on navigue à [http://127.0.0.1:8001/videogame/]  
On créé un VideoGame

On navigue à [http://127.0.0.1:8001/category/]  
On créé une catégorie

On revient sur  à [http://127.0.0.1:8001/videogame/]  
et édite le jeu créé.

Erreur : pk ?

On corrige en ajoutant la fonction __toString à chaque entité.
```php
//AppBundle\Entity/Category

    public function __toString()
    {
        return $this->name;
    }
```

```php
//AppBundle\Entity/VideoGame

    public function __toString()
    {
        return $this->name;
    }
```

On recharge ça marche mieux !  
On assigne la catégorie créée au VideoGame et on sauve.  
L'information est bien sauvée en base. Le videoGame est lié à la catégorie.  
On en profite pour créer un 2eme VideoGame sur [http://127.0.0.1:8001/videogame/new] et on n'assigne pas de catégory.


On va vouloir ajouter les VideoGame directement dans le formulaire de la category. On édite CategoryType
```php
// src/AppBundle/Form/Type/CategoryType.php

    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('name');
        $builder->add('videogames');
    }
}

// src/AppBundle/Entity/Category.php

    /**
     * @var ArrayCollection
     * @ORM\OneToMany(targetEntity="AppBundle\Entity\VideoGame")
     */
    private $videogames;
    
}
```
```bash
php bin/console doctrine:generate:entities AppBundle
error
[Doctrine\ORM\Mapping\MappingException]
OneToMany mapping on field 'videogames' requires the 'mappedBy' attribute.
```
Parce qu’on a une relation OneToMany et ManyToOne, on doit définir les mappedBy et inversedBy.  
Category ne possède pas physiquement d'information sur les VideoGame auxquels elle est rattachée. Le champ sql categorie_id se trouve dans la table de l'entité VideoGame. De ce fait on dit que VideoGame est le Owning Side de la relation. C'est lui qui possède l'information de la relation. En liant Category à VideoGame Doctrine a besoin de savoir où est le champ "maitre" de la relation et qui est l'inverse Side. C'est le but de ces paramètres.

```php
// src/AppBundle/Entity/Category.php

    /**
     * @var ArrayCollection
     * @ORM\OneToMany(targetEntity="AppBundle\Entity\VideoGame", mappedBy="category")
     */
    private $videogames;    
}
// src/AppBundle/Entity/VideoGame.php

    /**
     * @var Category $category
     * @ORM\ManyToOne(targetEntity="AppBundle\Entity\Category", inversedBy="videogames")
     */
    private $category;   
}
```
```bash
php bin/console doctrine:generate:entities AppBundle
```

On édite la category créé et on assigne le 2eme jeu
[http://127.0.0.1:8001/category/1/edit]


On sauve et la category est resté sur le premier jeu.  

PK ?  
On sauve une category. Elle n'est pas maître de la relation. Elle ne la met donc pas à jour. Il faut sauver l'objet videogame pour que l'effet ait lieu. C'est le OwningSide/InverSide effect.    

Le maitre de la relation est toujours VideoGame hors, nous avons simplement remplis un ArrayCollection de l'objet Category et sauvé l'objet Category. VideoGame doit être sauvé aussi pour mettre à jour la relation vu que c'est le maitre de la relation. Comment faire ?  
On corrige et on met à jour l'objet VideoGame lorsqu’on change la liste dans l'entité Category
```php
//AppBundle\Entity/Category
    public function addVideogame(\AppBundle\Entity\VideoGame $videogame)
    {
        $videogame->setCategory($this);
        $this->videogames[] = $videogame;

        return $this;
    }

    /**
     * Remove videogame
     *
     * @param \AppBundle\Entity\VideoGame $videogame
     */
    public function removeVideogame(\AppBundle\Entity\VideoGame $videogame)
    {
        $videogame->setCategory(null);
        $this->videogames->removeElement($videogame);
    }
```

On teste ! Tjs pas !  
Pk ?  
Le formulaire au moment où il lie la requête à l'objet Category utilise les getters et les setters. Hors dans le cas d'une relation OneToMany il n'y a pas de setter. Il faut donc modifier le formulaire pour ajouter une option.  
 
```php
//AppBundle\Form\CategoryType
    /**
     * {@inheritdoc}
     */
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('name');
        $builder->add('videogames', null, ['by_reference' => false]);
    }
```
On re-teste  
Miracle, ca marche !  

Bcp de changement pour finalement assigner des VideoGame depuis la category. Mais il faut comprendre que sans ce mécanisme. Doctrine pourrait partir en boucle infinie : Si les 2 cotés donnent l'ordre mettre à j'our l'objet lié.

### ManyToMany
On va créer une entity Tag et on va créer une relation ManyToMany entre Tag et VideoGame 
 
 ```bash
php bin/console doctrine:generate:entity  
AppBundle:Tag 
annotation
name
# quand vous est demandé les parametres de name mettez unique à true
```
```php
//src/AppBundle/Entity/VideoGame.php

    /**
     * @ORM\ManyToMany(targetEntity="AppBundle\Entity\Tag")
     */
    protected $tags;
}

//src/AppBundle/Entity/Tag.php

    //**
     * @ORM\ManyToMany(targetEntity="AppBundle\Entity\VideoGame")
     */
    protected $videogames;
    
    public function __toString(){
        return $this->name;
    }
    
//src/AppBundle/Form/Type/VideoGameType.php

    /**
     * {@inheritdoc}
     */
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('name')
            ->add('category')
            ->add('tags');
    }
}
```
```bash
php bin/console doctrine:generate:entities AppBundle
php bin/console doctrine:schema:update --force
php bin/console generate:doctrine:crud
AppBundle:Tag
write action : yes
```
On cree un tag et on lui assigne un ou plusieurs jeu directement  
Pk ca marche ??  
Car la relation ManyToMany est en fait une relation ManyToOne OneToMany et ManyToOne OneToMany automatique avec la table de liaison.    
Dans le cas d'un ManyToMany, doctrine construit une table intermediare, invisible dans le code. Ccette table est le owningSide. Vos entités VideoGame et Category sont les 2 inversedSide de la relation. Doctrine sait donc mettre à jour tout seul la relation entre les 2 objets.
et pk dans un sens et pas dans l'autre ??

Forcons les inversedBy et mappedBy. Dans un sens, la mise à jour marche masi pas dans l'autre. Si on inverse, le resultat s'inverse aussi.
Si on veut un bi directionnel, il faut reproduire le comportement vu plus tot en respectant le sesn de mise à jour.   



### Cascade
On a fait une erreur et on doit supprimer la category. On édite la Category et on delete.

Erreur : PK ?


On a créé une dépendance entre 2 tables. Il existe donc une contrainte d'intégrité. Elle empêche qu'un category_id dans la table VideoGame n'existe plus dans la table Category.  
Ce qui est le cas si on supprime la category. Les videoGame ont une relation obsolète. Par default c'est protégé. On va dire à doctrine de mettre à jour le champ dans la table VideoGame si on supprime la category, et comme on n’a pas de valeur, on la met à null.

```php
// src/AppBundle/Entity/VideoGame.php

    /**
     * @var Category $category
     * @ORM\ManyToOne(targetEntity="AppBundle\Entity\Category", inversedBy="videogames", cascade={"persist"})
     * @ORM\JoinColumn(onDelete="SET NULL")
     */
    private $category;   
}
```
```bash
php bin/console doctrine:schema:update --force
```

Si vous n'avez pas de VideoGame relié à un Category, recréez en une.
Puis on va supprimer ce VideoGame directement. On s'attend à ce que la category soit supprimé aussi.

Elle ne l'est pas, pk ?

Il manque l'ordre à doctrine.
```php
// src/AppBundle/Entity/VideoGame.php

    /**
     * @var Category $category
     * @ORM\ManyToOne(targetEntity="AppBundle\Entity\Category", inversedBy="videogames", cascade={"persist", "remove"})
     * @ORM\JoinColumn(onDelete="SET NULL")
     */
    private $category;
```

Désormais si on supprime la VideoGame on supprime la category.



Si on cree 2 VideoGame en rentrant 2 fois la même category, On créé 2 categories. Normal, on a rien empêché pour ca. On a même pas mis que name pouvait être unique.
On corrige
```php
//AppBundle/Entity/Category
    /**
     * @var string
     *
     * @ORM\Column(name="name", type="string", length=255, unique=true)
     */
    private $name;
```
```bash
php bin/console doctrine:schema:update --force
```
S'il pose des erreurs c'est surement que vous avez déjà 2 categories avec le même nom. Supprimez-les et relancez la commande.

Exo
Relions 2 VideoGame à une même category. Supprimons un jeu.
Trouver comment supprimer un jeu relié à 2 castegorie sans supprimer la category mais si on supprime le dernier jeu relié à cette category, cela supprime cette category.
Reponse : orphanRemoval


### Cascade Persist et Embedded Form
Retirons le champ videogames de CategoryType
```php
// src/AppBundle/Form/Type/CategoryType.php

    /**
     * {@inheritdoc}
     */
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('name');
    }
}
// src/AppBundle/Form/Type/VideoGameType.php

use 

    /**
     * {@inheritdoc}
     */
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('name')
            ->add('category', CategoryType::class);
    }
}
```
Et dans VideoGameType, ajoutons le formulaire de category directement.  
Editons un des VideoGame  
Rentrons une nouvelle category et on teste.  

Erreur : PK ?  

Doctrine ne sait pas quoi faire de cet objet. on lui a donné aucune indication. On a seulement persist l'object VideoGame. On doit donc lui dire quoi en faire.
```php
// src/AppBundle/Entity/VideoGame.php

    /**
     * @var Category $category
     * @ORM\ManyToOne(targetEntity="AppBundle\Entity\Category", inversedBy="videogames", cascade={"persist"})
     */
    private $category;   
}
```
On teste, ça marche, le VideoGame est relié à un nouvelle category


