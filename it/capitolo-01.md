#Capitolo 1
## Best Practices del framework Symfony

Symfony è conosciuto per essere un framework molto flessibile; esso è infatti utilizzato in diversi ambiti come ad esempio piccoli siti web, applicazioni enterprise con miliardi di richieste ma anche come base per altri framework.

Da quando è stato pubblicato nel luglio 2011, la comunità Symfony ha percorso un lungo cammino di apprendimento, sia su ciò che Symfony è in grado di fare, sia su qual è il modo migliore per farlo.


Le diverse risorse create dalla comunità, da articoli di blog a presentazioni a conferenze, hanno creato una serie di raccomandazioni e pratiche ufficiose per lo sviluppo di applicazioni Symfony.

Purtroppo molte di queste raccomandazioni sono sbagliate e oltre a complicare lo sviluppo delle stesse applicazioni non sono allineate con la filosofia originale e pragmatica dei creatori di Symfony.

### In cosa consiste questa guida?

Lo scopo di questa guida è quello di risolvere il problema sopra menzionato stabilendo una serie di buone pratiche ufficiali per lo sviluppo di applicazioni web con il framework Symfony. Queste pratiche sono quelle che meglio si adattano alla filosofia immaginata dal creatore originale del framework *Fabien Potencier*.

Sappiamo che le vecchie abitudini sono dure a morire e qualcuno potrebbe meravigliarsi o non essere d'accordo con alcune di queste regole. Nonostante ciò, crediamo che seguendo questi consigli potrete sviluppare le applicazioni più velocemente, con una minore complessità e con la stessa o perfino una migliore qualità. Inoltre le raccomandazioni saranno continuamente aggiornate e migliorate.


In ogni caso, considerate che le raccomandazioni sono opzionali e che voi e il vostro team potete adottarle come no. Se preferite continuare a sviluppare utilizzando le vostre pratiche e metodologie continuate a farlo. Symfony è abbastanza flessibile da adattarsi alle vostre necessità e questo non cambierà mai.


### A chi è rivolto il libro? (Nota: non è un Tutorial)
Questa guida è pensata per qualsiasi sviluppatore Symfony, sia esperto che principiante. Non essendo un tutorial passo passo, però, è necessaria una conoscenza basilare di Symfony, per seguire la guida al meglio. Se sei proprio agli inizi, benvenuto nella comunità. Inizia però prima con il tutorial [*The Quick Tour*](http://symfony.com/doc/current/quick_tour/the_big_picture.html). 

Inoltre abbiamo deciso di mantenere questa guida più piccola possibile, per essere molto facile da leggere. Non ci soffermeremo quindi su spiegazioni che si possono trovare nella documentazione ufficiale di Symfony, come discussioni sulla **dependency injection** o sul **front controller**. Ci soffermeremo unicamente a spiegare come fare meglio ciò che già conosci.

## L'applicazione di esempio
Insieme a questa guida troverete un'applicazione di esempio sviluppata seguendo le best practices. **L'applicazione è un semplice blog** e ci consentirà di concentrarci su ciò che ci lega a Symfony senza sconfinare in particolarità proprie dell'applicazione.

Invece di sviluppare l'applicazione passo passo, troverete snippet di codice all'interno dei singoli capitoli. Fare riferimento all'ultimo capitolo di questa guida per maggiori dettagli sull'applicazione e le istruzioni per installarla.


















