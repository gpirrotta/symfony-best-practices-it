#Capitolo 5
## I Controller

Symfony segue la filosofia "*thin controllers and fat models*".
Ciò significa che i controller dovrebbero contenere solo il codice strettamente necessario (**glue-code**)
per coordinare le diversi parti dell'applicazione.

La regola d'oro per i controller è sintetizzata nella tripla 5-10-20.
Ogni controller dovrebbe definire al massimo 5 variabili, contenere al massimo 10 azioni,
ciascuna delle quali formate al massimo da 20 righe di codice. Anche se possono
esserci eccezioni occasional alla regola, essa aiuta ad orientarsi su quando
iniziare a rifattorizzare il codice del controller per spostarlo in un servizio.

**Best Practice**
Estendi il tuo controller dalla classe base `Controller` fornita dal bundle
**FrameworkBundle** e usa le annotazioni per configurare rotte, cache
e sicurezza, quando possibile.


L'accoppiamento dei controller al framework sottostante consente di sfruttare tutte
le funzionalità del framework stesso aumentando la vostra produttività.

E poichè i vostri controller dovrebbero essere **sottili** e contenere
niente di più che poche linee di *glue-code*, spendere ore provando a disaccoppiarlo
dal framework non porterà grandi benefici nel lungo periodo.
La quantità di tempo *sprecato* non vale il beneficio.

Inoltre usare le annotazioni per le rotte, la cache e la sicurezza semplifica
enormemente la configurazione dell'applicazione.
Non sarà necessario esplorare decine di file di formati diversi
(YAML, XML, PHP): tutta la configurazione è lì dove ti serve e in un solo formato.

Complessivamente quindi si dovrebbe disaccoppiare totalmente la logica di business
dal framework e nello stesso tempo accoppiare totalmente al framework controller e rotte
in modo da ottenere il massimo da Symfony.


####Configurazione delle Rotte
Per caricare tutte le rotte definite nelle annotazioni dei controller del bundle
`AppBundle` aggiungete la seguente configurazione al file principale delle rotte:


```
# app/config/routing.yml
app:
    resource: "@AppBundle/Controller/"
    type: annotation
```

Questa configurazione caricherà le annotazioni da ogni controller presente sia nella
directory **src/AppBundle/Controller/** che nelle sue sottodirectory. Se la tua
applicazione definisce molti controller è perfettamente lecito organizzare il
tutto in sottodirectory.

```
<your-project>/
├─ ...
└─ src/
   └─ AppBundle/
      ├─ ...
      └─ Controller/
         ├─ DefaultController.php
         ├─ ...
         ├─ Api/
         │  ├─ ...
         │  └─ ...
         └─ Backend/
            ├─ ...
            └─ ...
```

####Configurazione del Template

**Best Practice**

Non usare l'annotazione **@Template()** per configurare il template usato dal controller

Anche se l'annotazione **@Template()** è utile essa implica qualche *magia*. Per questo motivo
si raccomanda di non usarla.

La maggior parte delle volte **@Template** è usato senza parametri il che rende più difficile
sapere quale template viene renderizzato. Il suo utilizzo inoltre rende meno ovvio
 ai principianti che un controller deve sempre ritornare un oggetto Response (a meno che non si usi
un view layer).

Infine l'annotazione **@Template** usa la classe **TemplateListener** per ascoltare
l'evento **kernel.view** del framework. L'ascoltatore ha un impatto non trascurabile
sulle prestazioni dell'applicazione. Nell'esempio dell'applicazione blog il rendering
dell'homepage impiega 5 millisecondi usando il metodo **$this->render()** mentre ben
26 millisecondi usando l'annotazione **@Template**.


####Come dovrebbe essere il Controller

Considerando tutto, ecco un esempio di come dovrebbe essere il controller
per l'homepage della nostra applicazione:

```
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

class DefaultController extends Controller
{
    /**
    * @Route("/", name="homepage")
    */
    public function indexAction()
    {
        $em = $this->getDoctrine()->getManager();
        $posts = $em->getRepository('App:Post')->findLatest();

        return $this->render('default/index.html.twig', array(
            'posts' => $posts
        ));
    }
}
```

####Usare il ParamConverter

Se la tua applicazione usa Doctrine è possibile usare *opzionalmente* il
[*ParamConverter*](http://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/annotations/converters.html)
per effettuare la ricerca dell'entity in modo automatico e passarla
come argomento del controller.

**Best Practice**
Usa il ParamConverter per caricare automaticamente le entity Doctrine
nei casi più semplici.

Per esempio:

```
/**
* @Route("/{id}", name="admin_post_show")
*/
public function showAction(Post $post)
{
    $deleteForm = $this->createDeleteForm($post);

    return $this->render('admin/post/show.html.twig', array(
        'post' => $post,
        'delete_form' => $deleteForm->createView(),
    ));
}
```

Solitamente ci si aspetterebbe un argomento **$id** nel metodo **showAction**.
Invece, creando un nuovo argomento (**$post**) e specificando il tipo di classe
**Post** (che è un'entity Doctrine), il ParamConverter cercherà automaticamente
un oggetto la cui proprietà **$id** corrisponde al valore **{id}**. Nel
caso in cui non venga trovato alcun **Post** verrà mostrato la pagina 404.


##### Esecuzione di ricerche più avanzate

Nell'esempio precedente tutto funziona senza nessuna configurazione
perchè il nome della wildcard **{id}** corrisponde esattamente al nome della
proprietà dell'entity. Quando questo non succede, o se si ha perfino una logica
più complessa, la cosa più facile da fare è cercare l'entity manualmente.
Questo è per esempio quello che succede nella classe `CommentController`  della
nostra applicazione:

```
/**
* @Route("/comment/{postSlug}/new", name = "comment_new")
*/
public function newAction(Request $request, $postSlug)
{
    $post = $this->getDoctrine()
        ->getRepository('AppBundle:Post')
        ->findOneBy(array('slug' => $slug));

    if (!$post) {
        throw $this->createNotFoundException();
    }

    // ...
}
```

Naturalmente è possibile configurare il **@ParamConverter** in modo più avanzato
perchè è abbastanza flessibile:

```
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\ParamConverter;

/**
  * @Route("/comment/{postSlug}/new", name = "comment_new")
  * @ParamConverter("post", options={"mapping": {"postSlug": "slug"}})
*/
public function newAction(Request $request, Post $post)
{
    // ...
}
```

Possiamo infine dire che la scorciatoia del ParamConverter è buona nelle situazioni semplici.
Nonostante ciò non si dovrebbe dimenticare che la ricerca diretta di entity è sempre
molto facile.

#### Esecuzione di codice prima e dopo del controller

Se si ha la necessità di eseguire del codice prima o dopo l'esecuzione dei controller
è possibile usare il componente *EventDispatcher*
[configurando i filtri prima/dopo](http://symfony.com/doc/current/cookbook/event_dispatcher/before_after_filters.html).

