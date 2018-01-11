Les Forms


on va créer une entité Catégorie.

`php bin/console doctrine:generate:entity`  
`AppBundle:Category`  
`name`

on va créer une entité Task.

`php bin/console doctrine:generate:entity`  
`AppBundle:Task`  
`name`


On édite la class Task et on ajoute la relation avec Categorie et inversement
```php
//AppBundle\Entity/Task
    /**
     * @var Category $category
     * @ORM\ManyToOne(targetEntity="AppBundle\Entity\Category", inversedBy="tasks")
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
 
Il manque les getter et setters, on demande à doctrine de les faire pour nous. Je préfère car il rajoute les bons commentaires et surtout un return $this; sur les getters  
`php bin/console doc:gen:entities AppBundle`

On migre la database  
`php bin/console doc:mig:diff`    
`php bin/console doc:mig:mig`

Maintenant on va écrire les forms correspondants.
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
        $builder->add('category');
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => Task::class,
        ));
    }
}
```

On crée ensuite les controllers des Task et Category
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
     * @Route(path="/task_new", name="task_new")
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

        return $this->render('@App/Task/new.html.twig', ['form' => $form->createView()]);

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
{# AppBundle\Resources\views\Task\new.html.twig#}
{% extends 'base.html.twig' %}

{% block body %}
    {{ form(form) }}
{% endblock %}
```
```twig
{# AppBundle\Resources\views\Task\edit.html.twig#}
{% extends '@App/Task/new.html.twig' %}
```


(EXO : Répétez pour Category)  

Si on va sur /category_new on se tape une erreur
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
On retourne sur /categorie_new est on crée une catégorie. Ensuite on va sur /task_new, on renseigne une tache et la catégorie créée et on valide.
Un tour dans la base donnée nous confirme que la Category a été sauvé et que la Task a été sauvée et liée à Category.


On modifie la form de Category
```php
    $builder->add('tasks');
```

On va sur category_new et on essaie d'ajouter une nouvelle category en associant la tache créé à l'étape précédente
Un tour dans la base de donnée nous indique que si la category est bien sauvée, la tâche est toujours assignée à la première category créé

EXO : PK ???
(mapped/inversedBy...)  
Solution : 
```php
//AppBundle\Entity/Category
public function addTask(Task $task){
    $this->tasks[] = $task;
    $task->setCategory($this);
    return $this;
}
```


***Embbed Forms***  
On veut désormais créer la Category à la création de la Task. On modifie le formulaire de Task pour ajouter celui de Category
```php
$builder->add('category', CategoryType::class);
```

Si on valide le formulaire sur task_new. On va avoir en l’état une erreur. Doctrine nous dit qu'il a trouvé une nouvelle entité Category mais alors qu'on lui a dit de sauver Task
 (avec le persist), il ne sait pas quoi faire de celle-là. Pour remédier à cela, on lui ajoute une indication à l'entité Task
 ```php
 //AppBundle\Entity/Task
     /**
      * @var Category $category
      * @ORM\ManyToOne(targetEntity="AppBundle\Entity\Category", inversedBy="tasks", cascade={"persist"})
      */
     private $category;
 ```
 On a rajouté `, cascade={"persist"}` Si une Task est sauvée, toute Category associée est aussi sauvée.


Exo : ajouter tasks à CategoryType
CF doc Collection form embbded avec javascript.

