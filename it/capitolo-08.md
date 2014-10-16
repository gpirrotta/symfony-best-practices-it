#Capitolo 8
## L'Internazionalizzazione

L'internazionalizzazione e la localizzazione hanno come obiettivo quello di adattare
 l'applicazione e il suo contenuto ad una specifica nazione o alla lingua dei suoi utenti.
In Symfony questa funzionalità è opzionale e per poter essere utilizzata bisogna prima abilitarla.
Per fare questo togliere il commento all'opzione di configurazione `translator` settando la
la località di default dell'applicazione:

```
# app/config/config.yml
framework:
    # ...
    translator: { fallback: "%locale%" }

    # app/config/parameters.yml
    parameters:
        # ...
        locale: en
```

#### Formato dei file di traduzione

Il componente `Translation` supporta diversi formati di file di traduzione: PHP, Qt, .po,
.mo, JSON, CSV, INI, etc.

**Best Practice**
Usa il formato XLIFF per i tuoi file di traduzione.

Fra tutti i formati di traduzione disponibili solo XLIFF e gettext sono ampiamente
supportati nei tool usati dai traduttori professionali. Poichè XLIFF è basato su XML
è possibile validare il contenuto del file appena viene creato.

Symfony 2.6 ha aggiunto il supporto per le note e commenti all'interno dei file XLIFF,
rendendoli più user-friendly per i traduttori.
Dato che una buona traduzione dipende fortemente dal contesto le note del formato XLIFF
rappresentano il posto migliore per contenere queste informazioni.


Il bundle [*JMSTranslationBundle*](https://github.com/schmittjoh/JMSTranslationBundle),
sotto licenza Apache, fornisce un'interfaccia Web per la visualizzazione e la modifica
dei file di traduzione. Inoltre dispone di uno strumento avanzato in grado di leggere il tuo
progetto, estrarre il testo da tradurre dai template e automaticamente aggiornare i file XLIFF.

#### La posizione dei file di traduzione

**Best Practice**
Mettere i file di traduzione nella directory `app/Resources/translations/`

Solitamente gli sviluppatori Symfony mettono questi file nella directory `Resources/translations/`
di ogni bundle.

Poichè la directory `app/Resources/` è considerata la posizione globale delle risorse
dell'applicazione, mettendo le traduzioni in `app/Resources/translations` esse risulteranno
centralizzate e prioritarie su ogni altro file di traduzione. Questo consentirà di effettuare
l'override delle traduzioni definite nei bundle di terze parti.

#### Definizione di chiavi per le traduzioni

**Best Practices**
Per le traduzioni usare sempre chiavi invece di contenuti stringa.

Usare le chiavi semplifica la gestione dei file di traduzione poichè è possibile
modificare il contenuto della lingua originale senza la necessità di aggiornare tutti i file
di tutte le lingue.

Le chiavi dovrebbero sempre descrivere il loro scopo e non la loro posizione. Per esempio
se un form ha un campo con l'etichetta "Username", una chiave idonea sarà `label.username`
e non `edit_form.label.username`.

#### Esempio di file di traduzione

Applicando tutte le best practice precedenti, un file di traduzione di esempio per la lingua inglese sarà:

```
<!-- app/Resources/translations/messages.en.xliff -->
<?xml version="1.0"?>
<xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2">
    <file source-language="en" target-language="en" datatype="plaintext">
        <body>
            <trans-unit id="1">
                <source>title.post_list</source>
                <target>Post List</target>
            </trans-unit>
        </body>
    </file>
</xliff>
```
