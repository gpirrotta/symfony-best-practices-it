#Capitolo 2
## La creazione del progetto

###Installazione di Symfony

Fra i tutti modi di installare Symfony ne raccomandiamo solo uno.

##### BEST PRACTICE
**Utilizzare sempre [*Composer*](https://getcomposer.org) per installare Symfony**

Composer è un dependency manager per applicazioni moderne in PHP. Grazie a questo strumento, aggiungere o togliere requisiti a
un progetto, così come aggiornare librerie di terze parti utilizzate dal codice, è un gioco da ragazzi.

##### Dependency Management con Composer
Prima di installare Symfony, accertarsi di aver installato Composer globalmente. Aprire un terminale
(chiamato anche console dei comandi) e digitare il seguente comando:

```
$ composer --version
Composer version 1e27ff5e22df81e3cd0cd36e5fdd4a3c5a031f4a 2014-08-11 15:46:48
```
Probabilmente si vedrà un identificatore di versione diverso. Questo perché Composer viene aggiornato
continuamente e non importa la sua versione.

#### Installazione di Composer globalmente
Nel caso non si abbia installato Composer globalmente, digitare i seguenti due comandi, se si usa
Linux o Mac OSX (il secondo comando chiederà la password utente)

```
$ curl -sS https://getcomposer.org/installer | php
$ sudo mv composer.phar /usr/local/bin/composer
```

A seconda della distribuzione Linux, potrebbe essere necessario digitare il comando **su** invece di **sudo**.

Se si usa un sistema Windows, scaricare l'installer dalla [pagina di download di Composer](https://getcomposer.org/download/) e seguire i passi per l'installazione.

### Creazione dell'applicazione Blog
Adesso che tutto è correttamente configurato, si può creare un nuovo progetto basato su Symfony.
Spostarsi nella console in una directory in cui si hanno i permessi per creare file ed eseguire i seguenti comandi:

```
$ cd projects/
$ composer create-project symfony/framework-standard-edition blog/
```

Il comando creerà una nuova directory chiamata **blog**, contenente l'ultima release stabile di Symfony disponibile.

#### Il controllo dell'installazione
Ultimata l'installazione, spostarsi nella directory **blog** e controllare la correttezza dell'installazione di Symfony, eseguendo i seguenti comandi:

```
$ cd blog/
$ php app/console --version
Symfony version 2.6.* - app/dev/debug
```

Se appare la versione installata di Symfony, tutto funziona come previsto. Altrimenti è possibile
eseguire uno *script* per controllare cosa impedisce al sistema di eseguire correttamente l'applicazione:

```
php app/check.php
```

Eseguendo lo script *check.php*, a seconda del sistema, è possibile visualizzare due elenchi differenti.
Il primo mostra i requisiti obbligatori che il sistema deve soddisfare per far funzionare l'applicazione Symfony.
Il secondo mostra i requisiti opzionali suggeriti per un'esecuzione ottimale dell'applicazione:

```
Symfony2 Requirements Checker
`~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

> PHP is using the following php.ini file:
/usr/local/zend/etc/php.ini

   > Checking Symfony requirements:
   .....E.........................W.....

[ERROR]
Your system is not ready to run Symfony2 projects

Fix the following mandatory requirements
`~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* date.timezone setting must be set
  > Set the "date.timezone" setting in php.ini* (like Europe/Paris).

Optional recommendations to improve your setup
`~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* short_open_tag should be disabled in php.ini
  > Set short_open_tag to off in php.ini*.
```

Per motivi di sicurezza, le release di Symfony sono firmate digitalmente. Per verificare l'integrità di un'installazione,
dare uno sguardo al [repository pubblico dei checksum](https://github.com/sensiolabs/checksums) e seguire
[questi passi](http://fabien.potencier.org/article/73/signing-project-releases) per verificare le firme.

### Strutturare l'applicazione
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

Questa gerarchia di file e directory è la struttura proposta da Symfony per organizzare un'applicazione.
Lo scopo di ciascuna directory è il seguente:

* **app/cache/** contiene tutti i file di cache generati dall'applicazione;
* **app/config/** contiene la configurazione definita per ciascun ambiente;
* **app/logs/** contiene tutti i file di log generati dall'applicazione;
* **app/Resources/** contiene tutti i file template e di traduzione dell'applicazione;
* **src/AppBundle/** contiene codice specifico per Symfony (controller e router), codice di dominio (ad es. le classi Doctrine)
e tutta la logica di business;
* **vendor/** in questa directory Composer installa tutte le dipendenze dell'applicazione; non si dovrebbe mai modificare il suo contenuto;
* **web/** contiene il front controller e tutti i Web asset, come i fogli di stile, i file Javascript e le immagini.


#### I bundle dell'applicazione
Quando è stato rilasciato Symfony 2.0, la maggior parte degli sviluppatori ha adottato, in modo naturale, lo stesso approccio usato
in symfony 1.x, suddividendo l'applicazione in moduli logici. Proprio per questo, molte applicazioni Symfony
definiscono bundle come:  **UserBundle**, **ProductBundle**, **InvoiceBundle**, ecc.

Tuttavia, i bundle sono stati concepiti come moduli software da riutilizzare in maniera autonoma.
Se **UserBundle** non può essere riusato "così com'è" in un'altra applicazione Symfony, allora non
è più un bundle. Inoltre **InvoiceBundle** dipende da **ProductBundle**, quindi non esiste alcun vantaggio
ad avere due bundle separati.

##### BEST PRACTICE
**Creare solamente un bundle, chiamato AppBundle, in un'applicazione**

Implementando solamente il bundle **AppBundle** in un progetto, si renderà il codice più conciso e facile
da capire. A partire da Symfony 2.6, la documentazione ufficiale di Symfony mostrerà gli esempi
con il bundle **AppBundle**

Non è necessario aggiungere il prefisso dell'azienda (*vendor*) ad **AppBundle** (es. **AcmeAppBundle**), dato
che questo bundle, specifico dell'applicazione, non verrà mai condiviso con terzi.

Detto questo, la struttura di directory raccomandata di un'applicazione Symfony è la seguente:

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

Se si sta usando Symfony 2.6 o una versione successiva, il bundle **AppBundle** è già presente.
Se si sta usando una versione Symfony più vecchia, lo si può generare con questo comando:

```
$ php app/console generate:bundle --namespace=AppBundle --dir=src --format=annotation --no-interaction
```

### Estendere la struttura delle directory
Se un progetto o un'infrastruttura richiedono alcune modifiche alla struttura predefinita delle directory,
è possibile effettuare l'[overriding della posizione delle principali directory](http://symfony.com/doc/current/cookbook/configuration/override_dir_structure.html): ad es.
**cache/**, **logs/** e **web/**.

Symfony3, inoltre, userà una struttura di directory leggermente diversa quando sarà rilasciato:

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
