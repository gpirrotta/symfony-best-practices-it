#Capitolo 6
## I Template

Quando è nato il PHP, 20 anni fa, gli sviluppatori rimasero incantati dalla sua semplicità
e dalla possibilità di mescolare facilamente HTML e codice dinamico.
Con il passare del tempo sono nati tanti linguaggi per i template - come [Twig](http://twig.sensiolabs.org/) -
in grado di gestire i template dell'applicazione in modo migliore.

#####Best Practice

**Usare Twig per i template**

Generalmente parlando, i template in PHP sono più prolissi di Twig, per
la mancanza di supporto nativo a molte caratteristiche necessarie nei template moderni,
come l'ereditarietà, l'escape automatico, i filtri.

Twig è il formato predefinito per i template in Symfony e può contare sul supporto della più grande
comunità di utenti fra tutti gli engine non PHP (è usato in progetti molto importanti, come Drupal 8).

Inoltre, Twig è l'unico formato con supporto garantito in Symfony 3.0.
Il formato PHP potrebbe non essere più supportato ufficialmente.


###Posizione dei template

##### Best Practice

**Inserire tutti i template dell'applicazione nella directory app/Resource/views/**.

Solitamente gli sviluppatori Symfony mettono i template dell'applicazione nella directory
**Resources/views/** di ciascun bundle. Per riferirsi ad essi usano il nome logico
(ad es. **AcmeDemoBundle:Default:index.html.twig**).

Anche se per i bundle a terzi quest'abitudine è corretta, è molto più conveniente, invece,
inserire i template dell'applicazione nella directory **app/Resources/views/**.

Innanzitutto questo semplifica drasticamente il nome logico dei template:

| **Template dentro i bundle**                  | **Template dentro app/**       |
|-----------------------------------------------|--------------------------------|
|`AcmeDemoBunde:Default:index.html.twig`        |`default/index.html.twig`       |
|`::layout.html.twig`                           |`layout.html.twig`              |
|`AcmeDemoBundle::index.html.twig`              |`index.html.twig`               |
|`AcmeDemoBundle:Default:subdir/index.html.twig`|`default/subdir/index.html.twig`|
|`AcmeDemoBundle:Default/subdir:index.html.twig`|`default/subdir/index.html.twig`|

Un altro vantaggio è che centralizzare i template semplifica il lavoro dei designer.
Essi non dovranno cercare più i template in tante directory sparpagliate fra i bundle.

###Estensioni Twig

#####Best Practice

**Definire le estensioni di Twig nella directory `AppBundle/Twig` configurandole
nel file `app/config/services.yml`**

All'applicazione serve un filtro Twig personalizzato `m2html` in modo da poter
trasformare il contenuto di ogni post da Markdown in HTML.

Per fare questo, per prima cosa, installiamo l'ottimo parser Markdown
[*ParseDown*](http://parsedown.org/) come nuova dipendenza del progetto:

```
$ composer require erusev/parsedown
```

Quindi creiamo un nuovo servizio chiamato `Markdown` che sarà usato successivamente
dall' estensione Twig. La definizione del servizio richiede solamente di specificare
il percorso della classe.

```
# app/config/services.yml
services:
    # ...
    markdown:
        class: AppBundle\Utils\Markdown
```

La classe `Markdown`  definisce un unico metodo per trasformare il contenuto
Markdown in contenuto HTML:

```
namespace AppBundle\Utils;

class Markdown
{
    private $parser;

    public function __construct()
    {
        $this->parser = new \Parsedown();
    }

    public function toHtml($text)
    {
        $html = $this->parser->text($text);

        return $html;
    }
}
```

Quindi creiamo un'estensione Twig e definiamo un nuovo filtro chiamato `md2html`
utilizzando la classe  `Twig_SimpleFilter`. Iniettiamo il nuovo servizio appena
definito `markdown` nel costruttore dell'estensione Twig:

```
namespace AppBundle\Twig;

use AppBundle\Utils\Markdown;

class AppExtension extends \Twig_Extension
{
    private $parser;

    public function __construct(Markdown $parser)
    {
        $this->parser = $parser;
    }

    public function getFilters()
    {
        return array(
            new \Twig_SimpleFilter(
                'md2html',
                array($this, 'markdownToHtml'),
                array('is_safe' => array('html'))
            ),
        );
    }

    public function markdownToHtml($content)
    {
        return $this->parser->toHtml($content);
    }

    public function getName()
    {
        return 'app_extension';
    }
}
```
Per poter utilizzare l'estensione di Twig nell'applicazione definiamo un nuovo
servizio, assegnandogli il tag `twig.extension` (il nome del servizio è irrilevante, perché
non verrà mai usato nel codice).

```
# app/config/services.yml
services:
    app.twig.app_extension:
    class: AppBundle\Twig\AppExtension
    arguments: ["@markdown"]
    tags:
    - { name: twig.extension }
```


