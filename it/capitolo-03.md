#Capitolo 3
## La Configurazione

La configurazione di un'applicazione solitamente coinvolge diverse parti (ad es. infrastruttura tecnologica, sicurezza,e tc.)
e diversi ambienti (sviluppo, produzione, etc.). Proprio per questo Symfony raccomanda di suddividere la configurazione in tre parti.

#### Configurazione relativa all'infrastruttura

**Best practice**
Definire le opzioni della configurazione relativa all'infrastruttura tecnologica
nel file **app/config/parameters.yml**

Il file di default **parameters.yml** segue questa regola e definisce le opzioni relative al database
e al mail server:

```
parameters:
    database_driver: pdo_mysql
    database_host: 127.0.0.1
    database_port: ~
    database_name: symfony
    database_user: root
    database_password: ~


    mailer_transport: smtp
    mailer_host: 127.0.0.1
    mailer_user: ~
    mailer_password: ~

    # ...

```

Queste opzioni non sono definite nel file **app/config/config.yml** perchè non hanno niente a che fare
con il comportamento dell'applicazione. In altre parole la tua applicazione non si deve preoccupare di
sapere dov'è posizionato il database o come vi accede. L'unica cosa che gli importa sapere
è che il database sia configurato correttamente.

##### Parametri canonici
**Best practice**
Definire tutti i parametri dell'applicazione nel file **app/config/parameters.dist.yml**


Dalla versione 2.3 Symfony include un file di configurazione chiamato **parameters.dist.yml**
che contiene tutti i parametri di configurazione che devono essere
definiti affinchè l'applicazione funzioni correttamente.

Ogni qualvolta viene definito un nuovo parametro di configurazione per l'applicazione, esso dovrebbe
essere aggiunto a questo file e tale modifica dovrebbe essere registrata anche sul vostro
sistema di controllo di versione.
Quando lo sviluppatore aggiornerà il progetto o effettuerà il deploy, Symfony controllerà
eventuali differenze tra il file *canonico* **parameters.dist.yml** e il file locale **parameters.yml**.
In presenza di differenze Symfony chiederà di fornire un valore per il nuovo parametro e lo aggiungerà al file
locale **parameters.yml**.

#### Configurazione relativa all'applicazione

**Best practice**
Definire le opzioni di configurazione relative all'applicazione nel file **app/config/config.yml**.

Il file **config.yml** contiene le opzioni usate dall'applicazione per modificare il suo comportamento, come ad esempio
il mittente delle email, l'abilitazione di [*feature toggles*](http://en.wikipedia.org/wiki/Feature_toggle), etc.
E' possibile definire questi valori anche in **parameters.yml** ma questo aggiungerebbe un livello di
configurazione extra non necessario, perchè solitamente non si vuole che questi valori cambino su ogni server.

##### Costanti o Opzioni di Configurazione
Uno degli errori più comuni nel definire la configurazione dell'applicazione è creare nuovi opzioni per valori
che non cambieranno mai, come ad esempio il numero degli elementi mostrati in una paginazione.

**Best Practice**
Usare costanti per definire opzioni di configurazione che cambieranno raramente

L'approccio tradizionale della definizione delle opzioni di configurazione ha costretto molte applicazioni
Symfony a includere opzioni come il seguente, che controlla il numero di post da mostrare
nell'homepage del blog.


```
# app/config/config.yml
parameters:
    homepage.num_items: 10

```

Se chiedete a voi stessi quando è stata l'ultima volta che avete modificato il valore di una opzione
come questa, è probabile che la risposta sia mai. Creare un'opzione di configurazione per un valore che
non si andrà mai a modificare è da stupidi. La nostra raccomandazione è di definire questi valori
come costanti nell'applicazione. Si potrebbe, ad esempio, definire una costante **NUM_ITEMS**
nell'entità **Post**

```php
// src/AppBundle/Entity/Post.php
namespace AppBundle\Entity;

class Post
{
    const NUM_ITEMS = 10;

    // ...
}
```

Il vantaggio più importante nella definizione di costanti è che si possono utilizzare dappertutto nell'applicazione.
A differenza i parametri sono disponibili solamente tramite il container di Symfony.

Le costanti possono essere usate, per esempio, nei template di Twig grazie alla funzione **constant()**


```php
<p>
    Displaying the {{ constant('NUM_ITEMS', post) }} most recent results.
</p>
```

Così facendo sia le entità di Doctrine che i repository possono accedere facilmente a questi valori mentre le stesse classi non posso accedere ai parametri del container.

```
namespace AppBundle\Repository;

use Doctrine\ORM\EntityRepository;
use AppBundle\Entity\Post;

class PostRepository extends EntityRepository
{
    public function findLatest($limit = Post::NUM_ITEMS)
    {
        // ...
    }
}
```

L'unico svantaggio da considerare nell'utilizzo delle costanti come opzioni di configurazione è che non
possono essere ridefinite facilmente nei test.


#### Non usare la configurazione semantica

**Best Practice**
Non definire nei tuoi bundle una configurazione semantica per il contenitore che inietta le dipendenze (dependency injection).

Come spiegato in [*Come esporre una configurazione semantica per un Bundle*](http://symfony.com/doc/current/cookbook/bundles/extension.html)
è possibile gestire le opzioni di configurazione in un bundle in due modi: la configurazione normale del servizio, attraverso il file
**services.yml** e la configurazione semantica, attraverso una classe speciale di tipo ***Extension**.

Sebbene la configurazione semantica è molto più potente e fornisce interessanti caratteristiche come la validazione
delle opzioni di configurazione, la quantità di lavoro necessaria per la definizione del bundle è notevole
e non vale la pena cimentarsi per bundle non rivolti a terzi.


#### Definire le opzioni di configurazione sensibili al di fuori di Symfony
Quando si lavora con opzioni sensibili, come le credenziali di accesso del database, si raccomanda di spostarle
al di fuori dell'applicazione Symfony e di renderle disponibili tramite le variabili d'ambiente. Impara come farlo
nel seguete articolo: [Come settare parameteri esterni nel Service Container](http://symfony.com/doc/current/cookbook/configuration/external_parameters.html).
