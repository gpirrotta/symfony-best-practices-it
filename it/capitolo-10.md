#Capitolo 10
## I Web Assets

I Web assets sono i fogli di stile CSS, i file JavaScript e le immagini che si utilizzano nel
frontend per renderlo accattivante. Gli sviluppatori Symfony, solitamente, mettono gli asset nella
directory `Resources/public/` di ogni bundle.

#####Best Practice

**Inserisci gli asset nella directory `web/` dell'applicazione.**

Sparpagliare i web assets tra decine di bundle rende il tutto più difficile da gestire.
La vita dei vostri designer sarebbe molto più facile se tutti gli asset dell'applicazione
fossero in un'unica posizione.

Centralizzando gli asset anche i template ne beneficerebbero; i link infatti sarebbero molto più corti:

```
<link rel="stylesheet" href="{{ asset('css/bootstrap.min.css') }}" />
<link rel="stylesheet" href="{{ asset('css/main.css') }}" />

{# ... #}

<script src="{{ asset('js/jquery.min.js') }}"></script>
<script src="{{ asset('js/bootstrap.min.js') }}"></script>
```

Ricordati che la directory `web/` è pubblica e che qualsiasi cosa memorizzata qui dentro
sarà pubblicamente accessibile. Per questo motivo dovresti mettere qui tutti i compilati dei web asset
ma non i loro file sorgente (ad es. i file SASS).

###Usare Assetic

Oggigiorno è praticamente impossibile trovare siti web che utilizzano solamente pochi file statici CSS e Javascript.
E' molto probabile che il tuo progetto utilizzi invece molti file Javascript e diversi file Sass o LESS per la generazione dei CSS.
Per migliorare le perfomance lato client tutti questi file andrebbero quindi raggruppati e minimizzati.

Esistono molti tool per risolvere questi problemi come ad esempio GruntJS, un tool progettato per il frontend (ma che non usa PHP).

#####Best Practice

**Usare Assetic per compilare, raggruppare e minimizzare i web asset, a meno che tu non abbia dimistichezza
con tool come GruntJS.**

[*Assetic*](http://symfony.com/doc/current/cookbook/assetic/asset_management.html) è un asset
manager in grado di compilare asset sviluppati con diverse tecnologie di frontend come LESS, Sass e
CoffeScript. E' possibile raggruppare tutti gli asset con Assetic  "wrappandoli" con un
unico tag Twig:

```
{% stylesheets
    'css/bootstrap.min.css'
    'css/main.css'
    filter='cssrewrite' output='css/compiled/all.css' %}
    <link rel="stylesheet" href="{{ asset_url }}" />
{% endstylesheets %}

{# ... #}

{% javascripts
    'js/jquery.min.js'
    'js/bootstrap.min.js'
    output='js/compiled/all.js' %}
    <script src="{{ asset_url }}"></script>
{% endjavascripts %}
```

###Applicazioni Frontend-Based

Recentemente tecnologie di frontend come AngularJS sono diventate molto popolari nello sviluppo
di applicazioni Web. Tali applicazioni comunicano con il sistema tramite API.

Se stai sviluppando un'applicazione come questa dovresti usare i tool raccomandati dalla tecnologia,
come Bower e GruntJS. Inoltre dovresti sviluppare la tua applicazione frontend in modo del tutto separato dal
backend Symfony (e anche separato dai repository).

###Per saperne di più su Assettic

Assettic è in grado di migliorare la velocità dei siti mininizzarndo asset CSS e Javascript
tramite [*UglifyCSS/UglifyJS*](http://symfony.com/doc/current/cookbook/assetic/uglifyjs.html).
E' possibile anche [comprimere immagini](http://symfony.com/doc/current/cookbook/assetic/jpeg_optimize.html)
riducendone la dimensione prima di essere restituiti nelle richieste.

Per saperne di più su tutte le funzionalità disponibili fare riferimento alla
[documentazione ufficiale](https://github.com/kriswallsmith/assetic).

