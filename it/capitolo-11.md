#Capitolo 11
##I Test

Generalmente gli sviluppatori creano due tipi di test: unitari e funzionali.
I test unitari consentono di testare l'input e l'output di funzioni specifiche.
I test funzionali consentono di simulare un "browser", quindi è possibile navigare
le pagine del tuo sito, cliccare sui link, riempire i form e asserire di vedere
certi elementi nella pagina.

###Test Unitari
I test unitari sono usati per testare la tua "logica di business"; essa è
 totalmente indipendente dal framework motivo per cui Symfony non include al suo interno
 nessun tool per i testi unitari. Tuttavia, i tool più conosciuti sono
[PhpUnit](https://phpunit.de/) e [PhpSpec](http://www.phpspec.net/).


###Test Funzionali

Creare buoni test funzionali può essere molto difficile, per questo motivo molti sviluppatori li
ignorano del tutto e non effettuano nessun test. La raccomandazione è di non ignorarli.
Infatti, definendo anche solo qualche semplice test funzionale potrai individuare velocemente
 grandi errori prima di effettuare il deploy:

#####Best Practice

**Definire il test funzionale che almeno controlli il caricamento corretto di tutte pagine
della tua applicazione.**


Un semplice esempio di un test funzionale:

 ```
/** @dataProvider provideUrls */
public function testPageIsSuccessful($url)
{
    $client = self::createClient();
    $client->request('GET', $url);
    $this->assertTrue($client->getResponse()->isSuccessful());
}

public function provideUrls()
{
    return array(
        array('/'),
        array('/posts'),
        array('/post/fixture-post-1'),
        array('/blog/category/fixture-category'),
        array('/archives'),
        // ...
    );
}
 ```

 Il codice controlla che tutti gli URL vengano caricati correttamente, cioè
 che il codice della risposta HTTP sia compreso tra `200` e `299`.
 Apparentemente potrebbe sembrare inutile ma, considerando quanto poco sforzo viene fatto,
 è sempre un bene avere il test nella vostra applicazione.

#### Hardcodare gli URL nei test funzionali

 Alcuni di voi si staranno chiedendo perchè nel precedente test funzionale non viene usato
 il servizio di generazione degli URL.

#####Best Practice

**Hardcodare direttamente gli URL nei test funzionali invece di usare il generatore di URL.**


Considera il seguente test funzionale che usa il servizio `router` per generare l'URL della
pagina testata:

```
public function testBlogArchives()
{
    $client = self::createClient();
    $url = $client->getContainer()->get('router')->generate('blog_archives');
    $client->request('GET', $url);

    // ...
}
```

Il test funzionerà correttamente ma avrà un grande inconveniente. Se per sbaglio uno sviluppatore
modifica il percorso della rotta `blog_archives`, il test continuerà ancora a funzionare, ma
l'URL originale non funzionerà più. Proprio per questo ogni bookmark di quell'URL non sarà
più raggiungibile con conseguenze anche sul page ranking nei motori di ricerca.


### Testare Javascript

Il client fornito da Symfony per i test funzionali funziona molto bene ma non può essere usato per testare
il comportamento di Javascript sulle tue pagine. Se ti serve questa funzionalità considera l'utilizzo della
[libreria Mink](http://mink.behat.org) con PHPUnit.

Ovviamente se la tua applicazione usa Javascript in tutte le sue funzionalità
 dovresti considerare l'uso di tool specificatamente pensati per testare Javascript.


###Saperne di più sui Test Funzionali

Usa le librerie [Faker](https://github.com/fzaninotto/Faker) e
[Alice](https://github.com/nelmio/alice) per la generazione dei dati delle fixture.

