#Capitolo 2
## La Creazione del Progetto

####Installazione di Symfony

C'è un solo modo raccomandato per installare Symfony

**BEST PRACTICE**  
**Utilizzare sempre [*Composer*](https://getcomposer.org) per installare Symfony**

Composer è un dependency manager per applicazioni moderne in PHP. Grazie a questo tool, aggiungere o togliere requisiti al
tuo progetto così come aggiornare librerie di terze parti utilizzate dal tuo codice è un gioco da ragazzi.

##### Dependency Management con Composer
Prima di installare Symfony, accertati di aver installato Composer globalmente. Apri il tuo terminale
(chiamato anche console dei comandi) e digita il seguente comando:

```
$ composer --version
Composer version 1e27ff5e22df81e3cd0cd36e5fdd4a3c5a031f4a 2014-08-11 15:46:48
```
Probabilmente vedrete un identificatore di versione diverso. Questo perchè Composer viene aggiornato
continuamente e non importa la sua versione.

##### Installazione di Composer globalmente
Nel caso tu non abbia installato Composer globalmente, digita i seguenti due comandi se usi
Linux o Mac OSX (il secondo comando vi chiederà la password utente)

```
$ curl -sS https://getcomposer.org/installer | php
$ sudo mv composer.phar /usr/local/bin/composer
```

A seconda della distribuzione Linux potrebbe essere necessario digitare il comando **su** invece di **sudo**.

Se usi un sistema Windows, scarica l'installer dalla [pagina di download di Composer](https://getcomposer.org/download/) e segui i passi per l'installazione.

#### Creazione dell'Applicazione Blog
Adesso che tutto è correttamente configurato, possiamo creare un nuovo progetto basato su Symfony.
Nella tua console, spostati in una directory dove hai permessi per creare file ed esegui i seguenti comandi:

```
$ cd projects/
$ composer create-project symfony/framework-standard-edition blog/
```

Il comando creerà una nuova directory chiamata **blog** contenente l'ultima release stabile di Symfony disponibile.

##### Il controllo dell'installazione
Ultimata l'installazione spostarsi nella directory **blog** e controllare la correttezza dell'installazione di Symfony eseguendo i seguenti comandi:

```
$ cd blog/
$ php app/console --version
Symfony version 2.6.* - app/dev/debug
```

Se appare la versione installata di Symfony tutto funziona come previsto. Altrimenti è possibile
eseguire uno *script* per controllare cosa impedisce al vostro sistema di eseguire correttamente l'applicazione:

```
php app/check.php
```

Eseguendo lo script *check.php*, a seconda del sistema, è possibile visualizzare due elenchi differenti.
Il primo mostra i requisiti obbligatori che il sistema deve soddisfare per far funzionare l'applicazione Symfony.
Il secondo mostra i requisiti opzionali suggeriti per un'esecuzione optimale dell'applicazione:

```
Symfony2 Requirements Checker
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

> PHP is using the following php.ini file:
/usr/local/zend/etc/php.ini

> Checking Symfony requirements:
   .....E.........................W.....

[ERROR]
Your system is not ready to run Symfony2 projects

Fix the following mandatory requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* date.timezone setting must be set
  > Set the "date.timezone" setting in php.ini* (like Europe/Paris).

Optional recommendations to improve your setup
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* short_open_tag should be disabled in php.ini
  > Set short_open_tag to off in php.ini*.
```

Per motivi di sicurezza le release di Symfony sono firmate digitalmente. Per verificare l'integrità della vostra installazione
date uno sguardo al [repository pubblico dei checksum](https://github.com/sensiolabs/checksums) e seguite
[questi passi](http://fabien.potencier.org/article/73/signing-project-releases) per verificare le firme.

#### Strutturare l'Applicazione
Dopo aver creato l'applicazione, spostandosi nella directory **blog/**, si vedrà il seguente insieme di file e directory, generato automaticamente:

```
blog/
├─ app/
│ ├─ console
│ ├─ cache/
│ ├─ config/
│ ├─ logs/
│ └─ Resources/
├─ src/
│ └─ AppBundle/
├─ vendor/
└─ web/
```

Questa gerarchia di file e directory è la convenzione proposta da Symfony per organizzare la vostra applicazione.
Lo scopo di ciascuna directory è il seguente:

* **app/cache/** contiene tutti i file di cache generati dall'applicazione;
* **app/config/** contiene la configurazione definita per ciascun ambiente;
* **app/logs/** contiene tutti i file di log generati dall'applicazione;
* **app/Resources/** contiene tutti i file template e di traduzione dell'applicazione;
* **src/AppBundle/** contiene codice specifico per Symfony (controller e router), codice di dominio (ad es. le classi Doctrine)
e tutta la logica di business;
* **vendor/** in questa directory Composer installa tutte le dipendenze dell'applicazione; non si dovrebbe mai modificare il suo contenuto;
* **web/** contiene il front controller e tutti i Web asset, come i fogli di stile, i file Javascript e le immagini.


##### I Bundle dell'Applicazione
Quando Symfony 2.0 è stato rilasciato la maggior parte di sviluppatori hanno adottato, in modo naturale, lo stesso approccio usato
in symfony 1.x suddividendo l'applicazione in moduli logici. Proprio per questo, molte applicazioni Symfony
definiscono bundle come:  **UserBundle**, **ProductBundle**, **InvoiceBundle**, etc.

Tuttavia i bundle sono stati concepiti come moduli software da riutilizzare in maniera autonoma.
Se **UserBundle** non può essere riusato "così com'è" in un'altra applicazione Symfony, allora non
è più un bundle. Inoltre **InvoiceBundle** dipende da **ProductBundle**, quindi non esiste alcun vantaggio
 ad avere due bundle separati.

**BEST PRACTICE**
**Crea solamente un bundle, chiamato AppBundle, nella tua applicazione**

Implementando solamente il bundle **AppBundle** nel tuo progetto renderai il tuo codice più conciso e facile
da capire. A partire da Symfony 2.6 la documentazione ufficiale di Symfony mostrerà gli esempi
 con il bundle **AppBundle**

Non è necessario aggiungere il prefisso della vostra azienda (*vendor*) ad **AppBundle** (es. **AcmeAppBundle**) dato
che il questo bundle, specifico dell'applicazione, non verrà mai condiviso con terzi.

Con tutto ciò la struttura di directory raccomandata di un'applicazione Symfony è la seguente:

```
blog/
├─ app/
│ ├─ console
│ ├─ cache/
│ ├─ config/
│ └─ logs/
│ └─ Resources/
├─ src/
│ └─ AppBundle/
├─ vendor/
└─ web/
├─ app.php
└─ app_dev.php
```

Se stai usando Symfony 2.6 o una versione successiva, il bundle **AppBundle** è già presente.
Se stai usando una versione Symfony più vecchia, puoi generarlo con questo comando:

```
$ php app/console generate:bundle --namespace=AppBundle --dir=src --format=annotation --no-interaction
```

#### Estendere la Struttura delle Directory
Se il tuo progetto o la tua infrastruttura richiede alcune modifiche alla struttura delle directory di default,
è possibile effettuare l'[overriding della posizione delle principali directory](http://symfony.com/doc/current/cookbook/configuration/override_dir_structure.html):
**cache/**, **logs/** and **web/**.

Inoltre, Symfony3 userà una struttura di directory leggermente diversa quando sarà rilasciato:

```
blog-symfony3/
├─ app/
│ ├─ config/
│ └─ Resources/
├─ bin/
│ └─ console
├─ src/
├─ var/
│ ├─ cache/
│ └─ logs/
├─ vendor/
└─ web/
```

Le modifiche sono piuttosto superficiali ma, per ora, si consiglia di utilizzare la struttura di directory di Symfony2.
