Les embbeded Forms


on va creer un entité Catégorie.

`php bin/console doctrine:generate:entity`  
`AppBundle:Category`  
`name`

on va creer un entité Task.

`php bin/console doctrine:generate:entity`  
`AppBundle:Task`  
`name`


On edite la class Task et on ajoute la relation avec Categorie et inversement
```php
//AppBundle\Entity/Task
    /**
     * @var Category $category
     * @ORM\ManyToOne(targetEntity="AppBundle\Entity\Category", inversedBy="tasks", cascade={"persist"})
     */
    private $category;
```
```php
//AppBundle\Entity/Category
    /**
     * @var Task $tasks
     * @ORM\OneToMany(targetEntity="AppBundle\Entity\Task", mappedBy="category")
     */
    private $tasks;
```
ICI : laius sur inversedBy, mappedBy
 
Il manque les getter et setters, on demande à doctrine de les faire pour nous. Je prefere car il rajoute les bons commentaires et surtout un return $this; sur les getters
`php bin/console doc:gen:entities AppBundle`

Maintenant on va generer les forms correspondants.
```php
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


use AppBundle\Entity\Task;
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
        $task = new Task();
        $form = $this->createForm(TaskType::class, $task);
        $form->add('valider', SubmitType::class);

        $form->handleRequest($request);
        if ($form->isSubmitted() && $form->isValid()) {

            $this->get('doctrine.orm.default_entity_manager')->persist($task);
            $this->get('doctrine.orm.default_entity_manager')->flush();

            return $this->redirectToRoute('task_edit', ['task' => $task->getId()]);

        }

        return $this->render('@App/Task/create.html.twig', ['form' => $form->createView()]);

    }

    /**
     * @param Request $request
     * @param Task $task
     * @return \Symfony\Component\HttpFoundation\Response
     * @Route(path="/task_edit/{task}", name="task_edit")
     */
    public function editAction(Request $request, Task $task)
    {
        $form = $this->createForm(TaskType::class, $task);
        $form->add('valider', SubmitType::class);

        $form->handleRequest($request);
        if ($form->isSubmitted() && $form->isValid()) {

            $this->get('doctrine.orm.default_entity_manager')->persist($task);
            $this->get('doctrine.orm.default_entity_manager')->flush();

        }

        return $this->render('@App/Task/edit.html.twig', ['form' => $form->createView()]);

    }
}
```
Et on créé 2 templates
```twig
{# AppBundle\Resources\views\Task\create.html.twig#}
{% extends 'base.html.twig' %}

{% block body %}
    {{ form(form) }}
{% endblock %}
```
```twig
{# AppBundle\Resources\views\Task\edit.html.twig#}
{% extends '@App/Task/create.html.twig' %}
```
On va sur /task_new, on renseigne un tache et un catégorie et on valide.
Un tour dans la base donnée nous confirme que la Task et la Category ont été sauvé.
Par contre, on a jamais sauvé Category explicitement... pk elle est dans la base de donnée. pk... ??

Car on a rajouté `, cascade={"persist"}` Si une Task est sauvée, toute Category associé est aussi sauvée.

PK pas dans l'autre sens ?
- si on avait les 2 sens, on aurait une boucle infinie.
- on a un champ category_id dans Task. On doit donc modifier la table Task pour mettre un jour la relation qu'on modifie Task ou Category. C'est donc Task qui "possede" une Category. C'est lui le boss, le master. C'est donc lui qui donne le sens pour sauver les relations entre les entités.

Si on fait un form des Category avec les tasks et on decide de sauver cette category. Aucune task ne sera assigné à cette catégorie sans intervention de notre part.

```php
//AppBundle\Controller\TaskController;
namespace AppBundle\Controller;


use AppBundle\Entity\Category;
use AppBundle\Entity\Task;
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
     * @param Task $task
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
`Catchable Fatal Error: Object of class AppBundle\Entity\Task could not be converted to string`

Il essaie d'afficher les tasks possibles, et pour cela il a besoin d'un label pour chaque Task, or on a oublié de dire à l'entité quel est son label pour l'instance en cours
```php
//AppBundle\Entity\Task
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
On revient, et on créé une category et on l'assigne à la tache précedemment sauvé. On valide et on regarde en base de donnée : la task est toujours relié à la premiere catégory créée  
=> explanation...

Pour que le slave donne l'ordre au master de sauver la relation il faut modifier l'entité Category
```php
    /**
     * @var Task $tasks
     * @ORM\OneToMany(targetEntity="AppBundle\Entity\Task", mappedBy="category", cascade={"persist"})
     */
    private $tasks;

...

    /**
     * Add task
     *
     * @param \AppBundle\Entity\Task $task
     *
     * @return Category
     */
    public function addTask(\AppBundle\Entity\Task $task)
    {
        $this->tasks[] = $task;
        $task->setCategory($this);

        return $this;
    }
```
Le cascade indique qu'il faut traiter les entités Task associés à cette Category. Hors sans le `$task->setCategory($this);` aucune Task n'est mise à jour. 