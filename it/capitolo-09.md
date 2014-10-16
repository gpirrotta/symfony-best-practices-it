#Capitolo 9
## La Sicurezza

####Autenticazione e Firewall
#####(Recuperare le credenziali dell'utente)

Per l'autenticazione degli utenti è possibile configurare Symfony con qualsiasi metodo
così come è possibile caricare le informazioni degli utenti da qualsiasi fonte. E' un argomento
abbastanza complesso e si rimanda al [CookBook, sezione sicurezza](http://symfony.com/doc/current/cookbook/security/index.html),
che contiene molte informazioni in merito.

A prescindere dalle tue necessità l'autenticazione è configurata in `security.yml`,  sotto
la chiave `firewalls`.

**Best Practice**
A meno che non si abbia due meccanismi di autenticazione differenti (ad esempio il form login
per il sito principale e un sistema a token per le API), si raccomanda di definire un unico
firewall con l'opzione `anonymous` abilitata.

La maggior parte delle applicazioni utilizza solamente un meccanismo di autenticazione e
 un insieme di utenti. Per questa tipologia di applicazioni basta soltanto un unico firewall.
 Ovviamente esistono delle eccezioni ad esempio quando nel tuo sito devi proteggere delle API dalla sezione web.
 L'importante è mantenere le cose semplici.

Si dovrebbe inoltre abilitare sempre l'opzione `anonymous` nel tuo firewall. Se hai bisogno che gli utenti
accedano a sezioni differenti del tuo sito utilizza la configurazione dell'opzione `access_control`.

**Best Practice**
Usare **bcrypt** per codificare le password degli utenti.

Se memorizzi le password degli utenti nel tuo sistema si raccomanda di usare l'encoder **bcrypt**,
invece della tradizionale codifica SHA-512. I vantaggi più importanti
di **bcrypt** sono l'inclusione di un valore *salt* per la protezione contro gli attacchi di tipo *rainbow table*
e la sua natura adattiva che consente di rallentare la sua esecuzione e resistere meglio agli attacchi di forza bruta.


Detto questo ecco un esempio di autenticazione della nostra applicazione che usa un form login per caricare
gli utenti dal database:

```
security:
    encoders:
        AppBundle\Entity\User: bcrypt

    providers:
        database_users:
            entity: { class: AppBundle:User, property: username }

    firewalls:
        secured_area:
            pattern: ^/
            anonymous: true
            form_login:
                check_path: security_login_check
                login_path: security_login_form

        logout:
            path: security_logout
            target: homepage

# ... access_control exists, but is not shown here
```

Il codice sorgente dell'applicazione di prova include commenti che spiegheranno
dettagliatamente ogni parte del file.


#### Autorizzazione
##### (Negare l'accesso)

Symfony definisce vari modi per configurare l'autorizzazione e restringere l'accesso degli utenti alle risorse.
In particolare è possibile configurare l'opzione `access_control` in [*security.yml*](http://symfony.com/doc/current/reference/configuration/security.html),
l'*annotation @Security* e usare il metodo *isGranted* del servizio `security.context`.

**Best Practice**
* Per la protezione di pattern URL molto grandi usa `access_control`
* Quando possibile usa l'annotazione *@Security* - (dalla versione inglese)
* Per proteggere singole risorse usa l'annotazione *@Security* -  (dalla traduzione spagnola)
* Per logiche di sicurezza più complesse usa direttamente il servizio *security.context*

Esistono anche diversi modi per centralizzare la logica di autorizzazione, come i
votanti e le ACL o lista di controllo degli accessi.

**Best Practice**
* Personalizzare un votante per definire restrizioni a grana fine
* Usare le ACL per definire logiche di sicurezza complesse, per gestire l'accesso di ogni oggetto da ogni
utente attraverso un'interfaccia admin.


#### L'annotazione @Security

Per controllare l'accesso su un controller usa l'annotazione `@Security`;
oltre ad essere di facile lettura essa è collocata sempre sopra ogni action.

Nella nostra applicazione di prova per creare un nuovo post è necessario disporre del ruolo `ROLE_ADMIN`.
Usando l'annotazione `@Security` il codice del controller sarà:


```
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Security;
// ...

/**
* Displays a form to create a new Post entity.
*
* @Route("/new", name="admin_post_new")
* @Security("has_role('ROLE_ADMIN')")
*/
public function newAction()
{
    // ...
}
```

##### Usare le Espressioni per Restrizioni di Sicurezza più Complesse

Se la tua logica di sicurezza è più complessa è possibile usare
le [espressioni](http://symfony.com/doc/current/components/expression_language/introduction.html)
dentro `@Security`. Nel seguente esempio l'utente potrà accedere al controller solamente se la sua email
corrisponde al valore ritornato dal metodo `getAuthorEmail` dell'oggetto `Post`:

```
use AppBundle\Entity\Post;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Security;

/**
* @Route("/{id}/edit", name="admin_post_edit")
* @Security("user.getEmail() == post.getAuthorEmail()")
*/
public function editAction(Post $post)
{
    // ...
}
```

Questa configurazione richiede l'uso del [*ParamConverter*](http://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/annotations/converters.html)
che automaticamente caricherà l'oggetto `Post` mettendolo nell'argomento `$post`.
Grazie a questa funzionalità è possibile usare la variabile `post` nell'espressione.

Lo svantaggio delle espressioni nelle annotazioni è che non possono essere
riusate facilmente in altre parti dell'applicazione. Si immagini di voler aggiungere un link in
un template visibile solo dagli autori.
Per ottenere questo comportamento si dovrà ripetere il codice dell'espressione usando la sintassi Twig:

```
{% if app.user and app.user.email == post.authorEmail %}
    <a href=""> ... </a>
{% endif %}
```

La soluzione più facile - se la tua logica è abbastanza semplice - è aggiungere un nuovo metodo
all'entità `Post` per controllare se un certo utente è l'autore del post.

```
// src/AppBundle/Entity/Post.php
// ...

class Post
{
    // ...

    /**
    * Is the given User the author of this Post?
    *
    * @return bool
    */
    public function isAuthor(User $user = null)
    {
        return $user && $user->getEmail() == $this->getAuthorEmail();
    }
}
```

Adesso è possibile riusare il metodo sia nel template che nell'espressione.

```
use AppBundle\Entity\Post;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Security;

/**
    * @Route("/{id}/edit", name="admin_post_edit")
    * @Security("post.isAuthor(user)")
*/
public function editAction(Post $post)
{
    // ...
}
```

```
{% if post.isAuthor(app.user) %}
    <a href=""> ... </a>
{% endif %}
```

####Controllare i permessi senza @Security

L'esempio visto sopra con `@Security` funziona perchè stiamo usando *ParamConverter* che dà
all'espressione l'accesso alla variabile `post`. Se invece *ParamConverter* non viene usato, o in presenza di
casi d'uso più avanzati, è sempre possibile effettuare il controllo da codice PHP:

```
/**
 * @Route("/{id}/edit", name="admin_post_edit")
 */
public function editAction($id)
{
    $post = $this->getDoctrine()->getRepository('AppBundle:Post')
        ->find($id);

    if (!$post) {
        throw $this->createNotFoundException();
    }

    if (!$post->isAuthor($this->getUser())) {
        throw $this->createAccessDeniedException();
    }

    // ...
}
```

#### I Votanti

Se la logica di sicurezza è complessa e non può essere centralizzata in un metodo come `isAuthor()`
si dovrebbe creare un votante personalizzato. Gestire la sicurezza con i votanti risulta più semplice
rispetto alle [ACL](http://symfony.com/doc/current/cookbook/security/acl.html) e fornisce la flessibilità
richiesta in quasi tutti gli scenari.

Inizialmente creiamo una classe votante.
Il seguente esempio mostra la classe che implementa la stessa
logica del metodo `getAuthorEmail` vista sopra:

```
namespace AppBundle\Security;

use Symfony\Component\Security\Core\Authorization\Voter\AbstractVoter;
use Symfony\Component\Security\Core\User\UserInterface;

// AbstractVoter class requires Symfony 2.6 or higher version
class PostVoter extends AbstractVoter
{
    const CREATE = 'create';
    const EDIT = 'edit';

    protected function getSupportedAttributes()
    {
        return array(self::CREATE, self::EDIT);
    }

    protected function getSupportedClasses()
    {
        return array('AppBundle\Entity\Post');
    }

    protected function isGranted($attribute, $post, $user = null)
    {
        if (!$user instanceof UserInterface) {
            return false;
        }

        if ($attribute == self::CREATE && in_array(ROLE_ADMIN, $user->getRoles())) {
            return true;
        }

        if ($attribute == self::EDIT && $user->getEmail() === $post->getAuthorEmail()) {
            return true;
        }

        return false;
    }
}
```

Per abilitare il votante nell'applicazione definire un nuovo servizio:

```
# app/config/services.yml
services:
    # ...
    post_voter:
        class: AppBundle\Security\PostVoter
        public: false
        tags:
            - { name: security.voter }
```

E' adesso possibile usare il votante dentro `@Security`:

```
/**
 * @Route("/{id}/edit", name="admin_post_edit")
 * @Security("is_granted('edit', post)")
 */
public function editAction(Post $post)
{
    // ...
}
```

E' possibile usare il votante direttamente dal controller tramite il servizio `security.context`:

```
/**
 * @Route("/{id}/edit", name="admin_post_edit")
 */
public function editAction($id)
{
    $post = // query for the post ...

    if (!$this->get('security.context')->isGranted('edit', $post)) {
        throw $this->createAccessDeniedException();
    }
}
```

####Saperne di più

Il bundle [*FOSUserBundle*](https://github.com/FriendsOfSymfony/FOSUserBundle), sviluppato dalla comunità
di Symfony, aggiunge il supporto alla gestione utenti memorizzati in una base di dati. Il bundle implementa
la gestione di task comuni come la registrazione utente e la funzionalità di recupero password.
Per consentire agli utenti di loggarsi solo una volta senza dover reinserire
 la password ogni volta che visitano il tuo sito , abilita la
funzionalità [*Remember Me*](http://symfony.com/doc/current/cookbook/security/remember_me.html)

Nel fornire assistenza ai clienti a volte è necessario accedere all'applicazione come *altri* utenti in modo
da poter riprodurre il problema. Symfony fornisce l'abilità di
[*impersonificare gli utenti*](http://symfony.com/doc/current/cookbook/security/impersonating_user.html).

Se la vostra azienda usa un metodo di login non supportato da Symfony è possibile sviluppare il
[*proprio user provider*](http://symfony.com/doc/current/cookbook/security/custom_provider.html) e il
[*proprio authentication provider*](http://symfony.com/doc/current/cookbook/security/custom_authentication_provider.html).
