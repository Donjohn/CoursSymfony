Les embbeded Forms


on va creer un entité Catégorie.

```bash
php bin/console doctrine:generate:entity  
AppBundle:Category  
annotation
name
```

on va creer un entité VideoGame.

```bash
php bin/console doctrine:generate:entity  
AppBundle:VideoGame 
annotation
name
# quand vous est demandé les parametres de name mettez unique à true
```

On ne peut choisir qu'un seul type de mapping (yml, xml, annotation) par bundle et les annotations ont la pririoté. Si une entité a des annotations de mapping doctrine les autres formes de mapping dans le bundle seront ignorés.

On edite la class AppBundle\Entity\VideoGame et on ajoute la relation avec Categorie et inversement
```php
//AppBundle\Entity/VideoGame

    /**
     * @var Category $category
     * @ORM\ManyToOne(targetEntity="AppBundle\Entity\Category", inversedBy="videogames", cascade={"persist"})
     */
    private $category;
```
```php
//AppBundle\Entity/Category

    /**
     * @var VideoGame $videogames
     * @ORM\OneToMany(targetEntity="AppBundle\Entity\VideoGame", mappedBy="category")
     */
    private $videogames;
```

Il manque les getter et setters, on demande à doctrine de les faire pour nous. Je prefere car il rajoute les bons commentaires et surtout un return $this; sur les getters
```bash
php bin/console doc:gen:entities AppBundle
```

On applique les changements dans la base de donnée
```bash
php bin/console doctrine:schema:update --force
```


ICI : laius sur inversedBy, mappedBy
 





Maintenant on va generer un crud et manipuler les forms et les relations.
```bash
php bin/console generate:doctrine:crud
AppBundle:VideoGame
write action : yes


php bin/console generate:doctrine:crud
AppBundle:Category
write action : yes
```

on verifie que tout marche
```bash
php bin/console server:run
```
Et on navigue à [http://127.0.0.1:8001/videogame/]

// src/AppBundle/Form/Type/CategoryType.php
namespace AppBundle\Form\Type;

use AppBundle\Entity\Category;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class CategoryType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('name');
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => Category::class,
        ));
    }
}
```
On migre la database
`php bin/console doc:mig:diff`    
`php bin/console doc:mig:mig`  
```php
// src/AppBundle/Form/Type/CategoryType.php
namespace AppBundle\Form\Type;

use AppBundle\Entity\Category;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class TaskType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('name');
        $builder->add('category', CategoryType::class);
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => Category::class,
        ));
    }
}
```
**_EXO : Pourquoi on ne met pas `$builder->add('tasks')` dans le CategoryType ?_**

On cree ensuite le controller des Tasks
```php
//AppBundle\Controller\TaskController;
namespace AppBundle\Controller;


use AppBundle\Entity\VideoGame;
use AppBundle\Form\Type\TaskType;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing\Annotation\Route;

class TaskController extends Controller
{
    /**
     * @param Request $request
     * @return \Symfony\Component\HttpFoundation\Response
     * @Route(path="/task_new", name="task_create")
     */
    public function createAction(Request $request)
    {
        $task = new VideoGame();
        $form = $this->createForm(TaskType::class, $task);
        $form->add('valider', SubmitType::class);

        $form->handleRequest($request);
        if ($form->isSubmitted() && $form->isValid()) {

            $this->get('doctrine.orm.default_entity_manager')->persist($task);
            $this->get('doctrine.orm.default_entity_manager')->flush();

            return $this->redirectToRoute('task_edit', ['VideoGame' => $task->getId()]);

        }

        return $this->render('@App/VideoGame/create.html.twig', ['form' => $form->createView()]);

    }

    /**
     * @param Request $request
     * @param VideoGame $task
     * @return \Symfony\Component\HttpFoundation\Response
     * @Route(path="/task_edit/{VideoGame}", name="task_edit")
     */
    public function editAction(Request $request, VideoGame $task)
    {
        $form = $this->createForm(TaskType::class, $task);
        $form->add('valider', SubmitType::class);

        $form->handleRequest($request);
        if ($form->isSubmitted() && $form->isValid()) {

            $this->get('doctrine.orm.default_entity_manager')->persist($task);
            $this->get('doctrine.orm.default_entity_manager')->flush();

        }

        return $this->render('@App/VideoGame/edit.html.twig', ['form' => $form->createView()]);

    }
}
```
Et on créé 2 templates
```twig
{# AppBundle\Resources\views\VideoGame\create.html.twig#}
{% extends 'base.html.twig' %}

{% block body %}
    {{ form(form) }}
{% endblock %}
```
```twig
{# AppBundle\Resources\views\VideoGame\edit.html.twig#}
{% extends '@App/VideoGame/create.html.twig' %}
```
On va sur /task_new, on renseigne un tache et un catégorie et on valide.
Un tour dans la base donnée nous confirme que la VideoGame et la Category ont été sauvé.
Par contre, on a jamais sauvé Category explicitement... pk elle est dans la base de donnée. pk... ??

Car on a rajouté `, cascade={"persist"}` Si une VideoGame est sauvée, toute Category associé est aussi sauvée.

PK pas dans l'autre sens ?
- si on avait les 2 sens, on aurait une boucle infinie.
- on a un champ category_id dans VideoGame. On doit donc modifier la table VideoGame pour mettre un jour la relation qu'on modifie VideoGame ou Category. C'est donc VideoGame qui "possede" une Category. C'est lui le boss, le master. C'est donc lui qui donne le sens pour sauver les relations entre les entités.

Si on fait un form des Category avec les tasks et on decide de sauver cette category. Aucune VideoGame ne sera assigné à cette catégorie sans intervention de notre part.

```php
//AppBundle\Controller\TaskController;
namespace AppBundle\Controller;


use AppBundle\Entity\Category;
use AppBundle\Entity\VideoGame;
use AppBundle\Form\Type\CategoryType;
use AppBundle\Form\Type\TaskType;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing\Annotation\Route;

class CategoryController extends Controller
{
    /**
     * @param Request $request
     * @return \Symfony\Component\HttpFoundation\Response
     * @Route(path="/category_new", name="category_create")
     */
    public function createAction(Request $request)
    {
        $category = new Category();
        $form = $this->createForm(CategoryType::class, $category);
        $form->add('tasks');
        $form->add('valider', SubmitType::class);

        $form->handleRequest($request);
        if ($form->isSubmitted() && $form->isValid()) {

            $this->get('doctrine.orm.default_entity_manager')->persist($category);
            $this->get('doctrine.orm.default_entity_manager')->flush();

            return $this->redirectToRoute('category_edit', ['category' => $category->getId()]);

        }

        return $this->render('@App/Category/create.html.twig', ['form' => $form->createView()]);

    }

    /**
     * @param Request $request
     * @param VideoGame $task
     * @return \Symfony\Component\HttpFoundation\Response
     * @Route(path="/category_edit/{category}", name="category_edit")
     */
    public function editAction(Request $request, Category $category)
    {
        $form = $this->createForm(CategoryType::class, $category);
        $form->add('tasks');
        $form->add('valider', SubmitType::class);

        $form->handleRequest($request);
        if ($form->isSubmitted() && $form->isValid()) {

            $this->get('doctrine.orm.default_entity_manager')->persist($category);
            $this->get('doctrine.orm.default_entity_manager')->flush();

        }

        return $this->render('@App/Category/edit.html.twig', ['form' => $form->createView()]);

    }
}
```
Si on va sur /category_name on se tape une erreur
`Catchable Fatal Error: Object of class AppBundle\Entity\VideoGame could not be converted to string`

Il essaie d'afficher les tasks possibles, et pour cela il a besoin d'un label pour chaque VideoGame, or on a oublié de dire à l'entité quel est son label pour l'instance en cours
```php
//AppBundle\Entity\VideoGame
    ...
    /**
     * @return string
     */
    public function __toString()
    {
        return $this->name;
    }
    ...
```
On revient, et on créé une category et on l'assigne à la tache précedemment sauvé. On valide et on regarde en base de donnée : la VideoGame est toujours relié à la premiere catégory créée  
=> explanation...

Pour que le slave donne l'ordre au master de sauver la relation il faut modifier l'entité Category
```php
    /**
     * @var VideoGame $tasks
     * @ORM\OneToMany(targetEntity="AppBundle\Entity\VideoGame", mappedBy="category", cascade={"persist"})
     */
    private $tasks;

...

    /**
     * Add VideoGame
     *
     * @param \AppBundle\Entity\VideoGame $task
     *
     * @return Category
     */
    public function addTask(\AppBundle\Entity\VideoGame $task)
    {
        $this->tasks[] = $task;
        $task->setCategory($this);

        return $this;
    }
```
Le cascade indique qu'il faut traiter les entités VideoGame associés à cette Category. Hors sans le `$task->setCategory($this);` aucune VideoGame n'est mise à jour. 