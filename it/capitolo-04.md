#Capitolo 4
## Organizzare la logica di business

La logica di business è "the part of the program that encodes the real-world
business rules that determine how data can be created, displayed, stored, and changed".
(leggi la [definizione completa da Wikipedia](http://en.wikipedia.org/wiki/Business_logic))

Nelle applicazioni Symfony la logica di business comprende tutto il codice implementato per l'applicazione
non relativo al framework. Le classi di dominio, le entità Doctrine
e classiche classi PHP utilizzate come servizi rappresentano buoni esempi di logica di business.

Nella maggior parte dei progetti la logica di business dovrebbe essere inserita dentro **AppBundle**.
All'interno del bundle è possibile creare qualsiasi gerarchia di directory come struttura organizzativa.


```
symfoy2-project/
├─ app/
├─ src/
│ └─ AppBundle/
│ └─ Utils/
│ └─ MyClass.php
├─ vendor/
└─ web/
```

### Creare classi fuori dal bundle

Non vi è alcuna limitazione tecnica che ci impedisca di mettere la logica di business
fuori dal bundle. Se si vuole si può creare il proprio namespace
dentro **src/** e mettere tutto là dentro:

```
symfoy2-project/
├─ app/
├─ src/
│ ├─ Acme/
│ │ └─ Utils/
│ │ └─ MyClass.php
│ └─ AppBundle/
├─ vendor/
└─ web/
```

La raccomandazione di usare il bundle `AppBundle` è giustificata dal voler rendere
tutto più facile da gestire. Se sei così esperto da sapere cosa è necessario mettere
dentro un bundle e cosa mettere invece fuori, sentiti libero di farlo.


### Nomi e formati dei servizi

La nostra applicazione blog ha bisogno di una utility in grado di trasformare il titolo di ogni post
(ad es. "Ciao Mondo") nel suo relativo slug (ad es. "ciao-mondo").
Lo slug verrà quindi usato come parte dell'URL del post.

Creiamo una classe **Slugger** dentro **src/AppBundle/Utils/** e aggiungiamo il metodo
**slugify()**:

```php
// src/AppBundle/Utils/Slugger.php
namespace AppBundle\Utils;

class Slugger
{
    public static function slugify($string)
    {
        return preg_replace(
            '/[^a-z0-9]/', '-', strtolower(trim(strip_tags($string)))
        );
    }
}
```

Definiamo quindi un nuovo servizio per quella classe.

```php
# app/config/services.yml
services:
    # keep your service names short
    slugger:
        class: AppBundle\Utils\Slugger
```

Per la definizione dei nomi dei servizi solitamente si sceglie
di utilizzare il nome e la posizione della classe per evitare collisioni di nomi.
Pertanto il servizio dovrebbe chiamarsi **app.utils.slugger**.
Tuttavia se si usano nomi dei servizi brevi il codice risulterà più facile da leggere e da usare.

##### Best Practice

**Il nome dei servizi dovrebbe essere il più breve possibile, idealmente solo una piccola parola.**

Adesso è possibile usare lo slugger da ogni controller, come ad es. **AdminController**:

```
public function createAction(Request $request)
{
    // ...

    if ($form->isSubmitted() && $form->isValid()) {
        $slug = $this->get('slugger')->slugify($post->getTitle()));
        $post->setSlug($slug);

        // ...
    }
}
```

### Il formato del file di configurazione YAML

Per la definizione del servizio, nella sezione precedente, è stato usato il formato YAML.

##### Best Practice

**Per la definizione dei propri servizi usare il formato YAML.**

Sappiamo che questa raccomandazione è molto controversa.
Dalla nostra esperienza sappiamo che sia il formato YAML che il formato XML è
ugualmente utilizzato tra gli sviluppatori, con una leggere preferenza verso YAML.
Entrambi i formati hanno le stesse performance, quindi la scelta di quale utilizzare
è una questione di gusti personali.

Noi raccomandiamo di usare YAML perchè risulta più semplice da gestire dai nuovi
programmatori sia più conciso. Ovviamente puoi usare il formato che preferisci.

### Non definire parametri per le classi dei servizi

Avrete probabilmente notato che nella definizione del servizio precedente non abbiamo creato
un parametro di configurazione per definire la classe di servizio

```
# app/config/services.yml

# service definition with class namespace as parameter
parameters:
    slugger.class: AppBundle\Utils\Slugger

services:
    slugger:
        class: "%slugger.class%"
```

Quest'abitudine risulta scomoda e completamente non necessaria per i propri servizi:

##### Best Practice

**Non definire parametri di configurazione per le classi dei servizi.**


Quest'abitudine trae la sua origine dai bundle di terze parti.
Se si sviluppa un bundle da condividere è possibile allora definire parametri di configurazione
per le classi. Ma se si sviluppa un servizio per la propria applicazione, non c'è bisogno
che le sue classi siano configurabili.


### Utilizzare lo strato di persistenza

Symfony è un framework HTTP che si preoccupa solo di generare una risposta HTTP
per ogni richiesta HTTP. Questo è il motivo per cui Symfony non prevede una
sua modalità per comunicare con lo strato di persistenza (ad es. database, API esterne)
E' possibile quindi scegliere la libreria o strategia preferita
per colmare questa carenza.

In pratica però molte applicazioni Symfony usano [Doctrine](http://www.doctrine-project.org/)
per definire il loro modello tramite entity e repository. Così come per la logica di business
si raccomanda di creare le entity di Doctrine nella directory **AppBundle**

Le tre entity definite dalla nostra applicazione blog sono un buon esempio di come rappresentare
le nostre classi:

```
symfony2-project/
├─ ...
└─ src/
   └─ AppBundle/
      └─ Entity/
         ├─ Comment.php
         ├─ Post.php
         └─ User.php
```

Se sei uno sviluppatore esperto, puoi creare le tue classi nel tuo namespace in **src/**.

### Il Mapping di Doctrine
Le entità doctrine sono semplici classi PHP le cui informazioni vengono memorizzate in qualche "database".
Le uniche informazioni conosciute da Doctrine su queste entità sono informazioni di
mapping di metadati sul vostro modello.
Doctrine supporta quattro formati per definire queste informazioni: YAML, XML, PHP e annotazioni.

##### Best Practice

**Usare le annotazioni per definire il mapping delle entità Doctrine**


Le annotazioni sono di gran lunga il modo più conveniente e agile per definire e cercare
le informazioni di mapping:

```php
namespace AppBundle\Entity;

use Doctrine\ORM\Mapping as ORM;
use Doctrine\Common\Collections\ArrayCollection;

/**
    * @ORM\Entity
*/
class Post
{
    const NUM_ITEMS = 10;
    /**
    * @ORM\Id
    * @ORM\GeneratedValue
    * @ORM\Column(type="integer")
    */
    private $id;

    /**
    * @ORM\Column(type="string")
    */
    private $title;

    /**
    * @ORM\Column(type="string")
    */
    private $slug;

    /**
    * @ORM\Column(type="text")
    */
    private $content;

    /**
    * @ORM\Column(type="string")
    */
    private $authorEmail;

    /**
    * @ORM\Column(type="datetime")
    */
    private $publishedAt;

    /**
    * @ORM\OneToMany(
    * targetEntity="Comment",
    * mappedBy="post",
    * orphanRemoval=true
    * )
    * @ORM\OrderBy({"publishedAt" = "ASC"})
    */
    private $comments;

    public function __construct()
    {
        $this->publishedAt = new \DateTime();
        $this->comments = new ArrayCollection();
    }

    // getters and setters ...
}
```

Tutti i formati hanno la stessa performance; la scelta su quale formato
usare dipende, ancora una volta, dai gusti personali.


#### Data Fixture

In Symfony il supporto alle fixture non è abilitato di default per cui, per installare
il bundle di gestione delle fixture in Doctrine, è necessario eseguire il seguente
comando:

```
$ composer require "doctrine/doctrine-fixtures-bundle":"~2"
```


Quindi è necessario abilitare il bundle in **AppKernel.php**, ma solo per gli ambienti **dev** e **test**:

```
use Symfony\Component\HttpKernel\Kernel;

class AppKernel extends Kernel
{
    public function registerBundles()
    {
        $bundles = array(
            // ...
        );

        if (in_array($this->getEnvironment(), array('dev', 'test'))) {
            // ...
            $bundles[] = new Doctrine\Bundle\FixturesBundle\DoctrineFixturesBundle(),
        }

        return $bundles;
    }
    // ...
}
```

Per semplicità si raccomanda di creare solamente [*una classe fixture*](http://symfony.com/doc/master/bundles/DoctrineFixturesBundle/index.html#writing-simple-fixtures)
anche se è consentito averne di più se questa classe diventa troppo grande.

Assumendo di avere almeno una classe fixture e che l'accesso del database sia configurato correttamente,
è possibile caricare il tutto eseguendo il seguendo comando:

```
$ php app/console doctrine:fixtures:load

Careful, database will be purged. Do you want to continue Y/N ? Y
    > purging database
    > loading AppBundle\DataFixtures\ORM\LoadFixtures
```

### Coding Standard
Il codice sorgente di Symfony rispetta gli standard [*PSR-1*](http://www.php-fig.org/psr/psr-1/)
e [*PSR-2*](http://www.php-fig.org/psr/psr-2/) definiti dalla comunità PHP.
Per saperne di più clicca su [*Symfony Code Standard*](http://symfony.com/doc/current/contributing/code/standards.html).

Inoltre è possibile usare il tool [*PHP-CS-Fixer*](https://github.com/fabpot/PHP-CS-Fixer), una utility
a riga di comando in grado di riformattare tutto il codice sorgente
dell'applicazione in pochi secondi.





