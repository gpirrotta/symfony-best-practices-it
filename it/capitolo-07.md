#Capitolo 7
## I form

I form sono uno dei componenti più abusati di Symfony; questo è dovuto sia al suo vasto campo
di applicazione sia alla sua lista infinita di funzionalità. In questo capitolo,
mostreremo alcune best practices in modo da poterli sfruttare al meglio.


### Creazione dei form

##### Best Practices

**Definire i form come classi PHP.**

Il componente `Form` consente di creare form direttamente dal controller.
A meno che non si voglia riusare il form da qualche altra parte, quest'abitudine
non è del tutto sbagliata.
Nonostante ciò, per form più complessi da poter riutilizzare in altri controller
si raccomanda di definire ogni form nella propria classe PHP.

```
namespace AppBundle\Form;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolverInterface;

class PostType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('title')
            ->add('summary', 'textarea')
            ->add('content', 'textarea')
            ->add('authorEmail', 'email')
            ->add('publishedAt', 'datetime')
        ;
    }

    public function setDefaultOptions(OptionsResolverInterface $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => 'AppBundle\Entity\Post'
        ));
    }

    public function getName()
    {
        return 'post';
    }
    }
```

Per creare il form, chiamare il metodo `createForm` e instanziare la nuova classe:

```
use AppBundle\Form\PostType;
// ...

public function newAction(Request $request)
{
    $post = new Post();
    $form = $this->createForm(new PostType(), $post);

    // ...
}
```

####Registrazione dei Form come Servizi
È possibile [registrare i tipi di form come servizi](http://symfony.com/doc/current/cookbook/form/create_custom_field_type.html#creating-your-field-type-as-a-service), anche se non si consiglia di farlo a meno che non si pianifichi di riusare lo stesso form in altri posti
o di incorporarlo all'interno di altri form usando il
[tipo collection](http://symfony.com/doc/current/reference/forms/types/collection.html).

Per la maggior parte dei casi in cui il form viene usato solo per editare o creare qualcosa, la registrazione come
servizio è eccessiva e rende più difficile capire esattamente quale classe viene usata nel controller.


###Configurazione dei Button

Le classe dei form dovrebbe essere agnostica rispetto a dove viene utilizzata. In questo modo
i form possono essere riusati più facilmente.

##### Best Practice

**Aggiungi i button direttamente nel template, non nelle classi dei form o nei controller.**

A partire da Symfony 2.5 è possibile aggiugere campi button all'interno del form.
Il vantaggio è che si semplifica il codice del template che visualizza il form.
Lo svantaggio è che aggiungere il bottone alla classe form ne limita la sua riusabilità.

```
class PostType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            // ...
            ->add('save', 'submit', array('label' => 'Create Post'))
        ;
    }

    // ...
}
```

In questo esempio il form è stato progettato per la creazione di post, ma se si volesse riusarlo
anche per la modifica la label del button sarebbe sbagliata.
Alcuni sviluppatori configurano invece i button del form nel controller:

```
namespace AppBundle\Controller\Admin;

use Symfony\Component\HttpFoundation\Request;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use AppBundle\Entity\Post;
use AppBundle\Form\PostType;

class PostController extends Controller
{
    // ...

    public function newAction(Request $request)
    {
        $post = new Post();
        $form = $this->createForm(new PostType(), $post);
        $form->add('submit', 'submit', array(
            'label' => 'Create',
            'attr' => array('class' => 'btn btn-default pull-right')
        ));

        // ...
    }
}
```

Anche questa soluzione è sbagliata perché si sta mischiando codice markup relativo
alla presentazione (etichette, classi CSS, etc) con puro codice PHP.
La separazione delle competenze è una buona regola da seguire sempre.
Proprio per questo tutto ciò che è relativo alla vista deve essere messo nel *view* layer:

```
<form method="POST" {{ form_enctype(form) }}>
    {{ form_widget(form) }}

    <input type="submit" value="Create"
           class="btn btn-default pull-right" />
</form>
```


###Renderizzare il Form
Symfony mette a disposizione diversi modi per renderizzare il form;
essi spaziano dal renderizzare tutto il form con un unico comando al
renderizzare ogni singolo campo in modo indipendente.

Il modo migliore dipende dalla quantità di personalizzazione necessaria nel form.

Il modo più semplice, utile specialmente durante lo sviluppo, è creare manualmente i tag
`<form></form>` e utilizzare la funzione `form_widget()`
per renderizzare tutti i campi:

```
<form method="POST" {{ form_enctype(form) }}>
    {{ form_widget(form) }}
</form>
```

#####Best Practice

**Non usare le funzioni `form()` o `form_start()` per renderizzare l'apertura e
la chiusura dei tag del form.**

Gli sviluppatori più esperti si saranno accorti che nell'esempio precedente
abbiamo creato manualmente i tag `<form>` invece di usare le funzioni `form_start()` o `form()`.
Nonostante la comodità di queste ultime funzioni, esse non portano molti vantaggi
perché riducono la chiarezza e la leggibilità del form stesso apportando quindi solo pochi benefici.


L'unica eccezione è il form `delete` perché, essendo costituito solamente da un bottone, può
beneficiare delle funzionalità messe a disposizione dal framework.

Se si ha bisogno di un controllo più preciso sulla renderizzazione del form
non usare la funzione `form_widget(form)` ma renderizzare i campi
individualmente.
Per maggiori informazioni su come renderizzare i form e su come impostare un tema in modo globale
consultare il seguente articolo
[*Come personalizzare il rendering dei form*](http://symfony.com/doc/current/cookbook/form/form_customization.html).

###Gestione del submit
La gestione di un form in Symonfy generalmente segue la seguente struttura:

```
public function newAction(Request $request)
{
    // build the form ...

    $form->handleRequest($request);

    if ($form->isSubmitted() && $form->isValid()) {
        $em = $this->getDoctrine()->getManager();
        $em->persist($post);
        $em->flush();

    return $this->redirect($this->generateUrl(
        'admin_post_show',
        array('id' => $post->getId())
        ));
    }

    // render the template
}
```
Nel codice precedente è importante evidenziare due cose.

In primo luogo si raccomanda di usare un'unica action sia per renderizzare
il form che per la gestione del submit.

In realtà potresti avere una `newAction` che renderizza il form e una `createAction` che
processa solo il submit.
Dato che entrambe le azioni però sono quasi identiche è più semplice lasciare che sia `newAction` a gestire il tutto.

In secondo luogo si raccomand di usare `$form->isSubmitted()`
nello statement `if` per rendere il codice più chiaro.

Tecnicamente non è necessario dato che `isValid()`  esegue prima `isSubmitted()`; senza
questo, tuttavia, il flusso risulterebbe un po strano
 e il form sembrerebbe *sempre* processato, anche per le richieste GET.



