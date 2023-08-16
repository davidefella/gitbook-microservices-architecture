# Architettura a microservizi (in soccorso)

### SLIDE 02-01

Ma cosa sono esattamente i microservizi? Purtroppo, il nome non aiuta perché si mette troppo l'accento sulla dimensione e non sulle funzionalità. Esistono numerose definizioni dell'architettura a microservizi.&#x20;

Alcune interpretazioni prendono il nome troppo letteralmente e sostengono che un servizio dovrebbe essere minuscolo, ad esempio 100 linee di codice. Altre affermano che un servizio dovrebbe richiedere solo due settimane di sviluppo e c'è chi definisce _l'architettura a microservizi come un'architettura orientata ai servizi composta da elementi debolmente accoppiati con contesti delimitati._ Quest'ultima non è una cattiva definizione, ma risulta un po' generica

Una delle definizioni più accettate su cosa sia un'architettura a microservizi è ispirata ad un modello di scalabilità tridimensionale: il cubo della **scalabilità**, il modello definisce tre modi per scalare un'applicazione: X, Y e Z.

<figure><img src="../.gitbook/assets/Screenshot 2023-08-11 alle 21.45.07.png" alt=""><figcaption></figcaption></figure>

### SLIDE 02-02

La scalabilità sull'asse X è un modo comune per scalare un'applicazione monolitica si eseguono molteplici istanze dell'applicazione dietro un load balancer che distribuisce le richieste tra le N istanze identiche dell'applicazione, questo è un buon compromesso per migliorare la prestazioni e la disponibilità di un'applicazione.

<figure><img src="../.gitbook/assets/Screenshot 2023-08-14 alle 20.29.30.png" alt=""><figcaption></figcaption></figure>

### SLIDE 02-03

Anche la scalabilità sull'asse Z esegue più istanze dell'applicazione monolitica, ma a differenza della scalabilità sull'asse X, ogni istanza è responsabile solo di un sottoinsieme dei dati.

Il Router di fronte alle istanze utilizza un _attributo_ della richiesta per instradarla verso l'istanza appropriata. Ad esempio, un'applicazione potrebbe instradare le richieste utilizzando l'identificativo dell'utente (userId). In questo esempio, ogni istanza dell'applicazione è responsabile di un sottoinsieme di utenti. Il router utilizza l'identificativo dell'utente specificato dall'intestazione di autorizzazione della richiesta per selezionare una delle istanze disponibili.&#x20;

In questo esempio, ogni istanza dell'applicazione è responsabile di un sottoinsieme di utenti. Il router utilizza l'identificativo dell'utente (userId) specificato dall'intestazione di autorizzazione della richiesta per selezionare una delle N istanze identiche dell'applicazione. La scalabilità sull'asse Z è un ottimo modo per scalare un'applicazione per gestire volumi crescenti di transazioni e dati.



<figure><img src="../.gitbook/assets/Screenshot 2023-08-14 alle 20.30.04.png" alt=""><figcaption></figcaption></figure>

### SLIDE 02-04



<figure><img src="../.gitbook/assets/Screenshot 2023-08-16 alle 15.34.44.png" alt=""><figcaption></figcaption></figure>

La scalabilità sull'asse X e Z migliorano la capacità e la disponibilità dell'applicazione ma nessuno dei due approcci risolve il problema dell'aumento della complessità dello sviluppo e della nostra applicazione.&#x20;

Per risolvere questi problemi, è necessario applicare la scalabilità sull'asse Y (o decomposizione funzionale). In figura vediamo come funziona la scalabilità sull'asse Y: suddividendo un'applicazione monolitica in un insieme di servizi.

Possiamo definire un servizio come una mini applicazione che implementa funzionalità strettamente focalizzate, come la gestione degli ordini, la gestione dei clienti e così via. Un servizio viene scalato utilizzando la scalabilità sull'asse X, anche se alcuni servizi possono utilizzare anche la scalabilità sull'asse Z. Ad esempio, il servizio degli Ordini è costituito da un insieme di istanze di servizio bilanciate da un load balancer.&#x20;

Arriviamo quindi alla definizione di alto livello dell'architettura a microservizi: uno stile architetturale che scompone funzionalmente un'applicazione in un insieme di servizi, quello che conta è che ogni servizio abbia un insieme di responsabilità focalizzato e coeso.

### SLIDE 02-05

La modularità è fondamentale nello sviluppo di applicazioni grandi e complesse come quella del nostro studio di caso. UrbanEats infatti di base è troppo grande per essere sviluppata da un singolo individuo ed è anche troppo complessa per essere compresa da una sola persona.&#x20;

Le applicazioni devono essere scomposte in moduli che sono sviluppati e compresi da persone diverse ma in un'applicazione monolitica i moduli vengono definiti utilizzando una combinazione di costrutti e artefatti del linguaggio di programmazione (come i pacchetti Java, JAR e WAR). Il problema è che come abbiamo già visto questo approccio tende a non funzionare bene nella pratica. Le applicazioni monolitiche a lunga durata tendono spesso a degenerare in grandi masse disordinate.

L'architettura a microservizi utilizza i servizi come piccole unità di modularità, una caratteristica chiave dell'architettura a microservizi è proprio che i servizi sono debolmente accoppiati e comunicano solo tramite API. Un modo per ottenere un accoppiamento debole è che ciascun servizio abbia il proprio datastore.&#x20;

Nell'online store, ad esempio, il servizio degli Ordini ha un database che include la tabella ORDERS, mentre il servizio dei Clienti ha il suo database con la tabella CUSTOMERS. In questo modo durante i lavori gli sviluppatori possono modificare lo schema di un servizio senza dover coordinarsi con gli sviluppatori che lavorano su altri servizi. Durante l'esecuzione, i servizi sono isolati l'uno dall'altro; ad esempio, un servizio non verrà mai bloccato perché un altro servizio detiene un lock sul database.

Nota: l requisito che ogni servizio abbia il proprio database non implica necessariamente che debba avere il proprio server di database. Ad esempio, non è necessario spendere 10 volte di più per le licenze di Oracle RDBMS

Diamo un'occhiata veloce a cosa intendiamo per applicare la scalabilità sull'asse Y a questa applicazione. Se applichiamo la decomposizione su asse Y all'applicazione UberEats, otteniamo l'architettura che vedete in figura.&#x20;

L'applicazione decomposta è composta da numerosi servizi frontend e backend. Applicheremmo anche la scalabilità su asse x ed la scalabilità su asse z, in modo che durante l'esecuzione ci siano più istanze di ciascun servizio.

<figure><img src="../.gitbook/assets/Screenshot 2023-08-14 alle 20.39.34.png" alt=""><figcaption></figcaption></figure>

I servizi frontend includono un API gateway e l'interfaccia utente web del ristorante. L'API gateway, che svolge il ruolo di una facade fornisce le API REST utilizzate dalle applicazioni mobili dei consumatori e dei corrieri. L'interfaccia utente web del ristorante implementa l'interfaccia web utilizzata dai ristoranti per gestire i menu e elaborare gli ordini.

La logica di business dell'applicazione è composta da numerosi servizi backend. Ogni servizio backend ha una REST API e il proprio datastore privato, ad esempio:&#x20;

* **OrderService**: Gestisce gli ordini&#x20;
* **DeliveryService**: Gestisce la consegna degli ordini dai ristoranti ai consumatori&#x20;
* **RestaurantService**: Mantiene le informazioni sui ristoranti&#x20;
* **KitchenService**: Gestisce la preparazione degli ordini&#x20;
* **AccountingService**: Gestisce la fatturazione e i pagamenti

Ogni servizio e la sua API sono chiaramente definiti e ciascuno di essi può essere sviluppato, testato, implementato e scalato in modo indipendente preservando comunque la modularità. Uno sviluppatore non può aggirare l'API di un servizio e accedere ai suoi componenti interni.

### SLIDE 02-06

<figure><img src="../.gitbook/assets/Screenshot 2023-08-14 alle 20.44.47.png" alt=""><figcaption></figcaption></figure>

Una delle critiche che si muovono spesso a questo tipo di architettura a microservizi è che in realtà non ci sia niente di veramente nuovo ma che sia una forma di architettura **orientata ai servizi (SOA)** e in effetti ad un livello molto alto, ci sono alcune somiglianze. Sono effettivamente stili architetturali che strutturano un sistema come un insieme di servizi.&#x20;

Detto questo però utilizzano solitamente stack tecnologici diversi. Le applicazioni SOA utilizzano tipicamente tecnologie come SOAP e altri standard di servizi web come il WSDL, spesso utilizzano un _Enterprise Service Bus (ESB)_ per la comunicazione tra le varie parti dell'applicativo e si interpone come intermediario che fa anche da "smart pipe" cioè che contiene logica di business e di elaborazione dei messaggi per integrare i servizi.&#x20;

Le applicazioni costruite utilizzando l'architettura a microservizi invece tendono a utilizzare tecnologie leggere e open source, i servizi comunicano tramite "dumb pipes" (come un semplice canale di comunicazione quindi), come i message broker o protocolli leggeri come REST o gRPC (un framework di comunicazione).

SOA e l'architettura a microservizi differiscono anche nel modo in cui trattano i dati. Le applicazioni SOA hanno tipicamente un modello di dati globale e condividono database.Nell'architettura a microservizi ogni servizio ha il proprio database dedicato e ogni servizio di solito è considerato avere il proprio modello di dominio.

Un'altra differenza chiave tra SOA e l'architettura a microservizi è la dimensione dei servizi. SOA è tipicamente utilizzato per integrare applicazioni monolitiche grandi e complesse. Anche se i servizi in un'architettura a microservizi non sono sempre piccoli, sono quasi sempre molto più piccoli. Di conseguenza, un'applicazione SOA di solito è composta da alcuni servizi grandi, mentre un'applicazione basata su microservizi di solito è composta da decine o centinaia di servizi più piccoli.

### SLIDE 02-07

L'architettura a microservizi presenta diversi benefici



Tra i benefici più importanti dell'architettura a microservizi c'è la possibilità di avere una reale **delivery continua** di applicazioni anche grandi e complesse. Come vedremo più avanti questa attività fa parte della figura di DevOps che si occupa di pratiche per la consegna rapida, frequente e affidabile del software. Ci sono tre modi in cui l'architettura a microservizi consente la consegna/il rilascio continui:

1. Poiché ogni servizio in un'architettura a microservizi è relativamente piccolo, i **test automatizzati** sono molto più facili da scrivere e più veloci da eseguire e di conseguenza, l'applicazione avrà meno bug.
2. Ogni servizio può essere distribuito **indipendentemente** dagli altri servizi, di conseguenza è molto più facile rilasciare frequentemente le modifiche in produzione.
3. **Consente alle squadre di sviluppo di essere autonome e debolmente accoppiate**: Si possono strutturare i team come una collezione di piccole squadre, ognuna delle quali è unicamente responsabile dello sviluppo e del rilascio di uno o più servizi correlati, di conseguenza la velocità di sviluppo è molto più elevata.

Questa capacità si traduce in altri benefici:

1. Capacità dell'azienda di reagire rapidamente ai **feedback** dei clienti, riducendo il tempo necessario per portare nuove funzionalità sul mercato richieste
2. L'azienda può fornire un servizio **affidabile** che i clienti si aspettano e questo contribuisce a costruire la fiducia e la soddisfazione del cliente.
3. Aspetto da non sottovalutare i dipendenti sono più **soddisfatti** poiché dedicano più tempo a sviluppare funzionalità di valore anziché a gestire situazioni d'emergenza continue.

Di conseguenza, l'architettura a microservizi è diventata una componente fondamentale per qualsiasi azienda che dipenda dalla tecnologia del software.





Un altro grande vantaggio dell'architettura a microservizi è che ogni servizio è **relativamente** **piccolo** e questo porta a scrivere codice più comprensibile, la dimensione ridotta del codice non rallenta l'IDE, aumentando la produttività degli sviluppatori. Inoltre, ogni servizio di solito si avvia molto più velocemente rispetto a un grande monolite, il che migliora anche la produttività degli sviluppatori e accelera i rilasci.





Un altro beneficio, ogni servizio in un'architettura a microservizi può essere **scalato in modo indipendente** dagli altri servizi utilizzando il cloning sull'ass X e la suddivisione dell'asse Z come abbiamo visto poco fa. Inoltre, ogni servizio può essere distribuito su hardware che si adatta meglio ai suoi requisiti di risorse. Questo è molto diverso dall'utilizzo di un'architettura monolitica, in cui componenti con requisiti di risorse molto diversi, ad esempio intensivi per la CPU rispetto a intensivi per la memoria, devono essere distribuiti insieme.





L'architettura a microservizi offre una migliore **isolamento dei guasti** perchè ad esempio una perdita di memoria in un servizio influisce solo su quel servizio. Gli altri servizi continueranno a gestire le richieste normalmente. In confronto, un componente mal funzionante in un'architettura monolitica farà collassare l'intero sistema.





Ultimo ma non meno importante, l'architettura a microservizi elimina o comunque riduce notevolmente il **lock** a lungo termine verso un particolare **stack tecnologico**. In linea di principio, quando si sviluppa un nuovo servizio, gli sviluppatori sono liberi di scegliere il linguaggio e i framework più adatti per quel servizio. In molte organizzazioni ha senso limitare le scelte chiaramente ma il punto chiave è che non si è vincolati dalle decisioni del passato. Inoltre, dato che i servizi sono piccoli, riscriverli utilizzando linguaggi e tecnologie migliori diventa fattibile. Se il tentativo di una nuova tecnologia fallisce, è possibile scartare quel lavoro senza mettere a rischio l'intero progetto. Questo è molto diverso dall'utilizzo di un'architettura monolitica, dove le scelte tecnologiche iniziali limitano gravemente la capacità di utilizzare linguaggi e framework diversi in futuro.

###

### SLIDE 02-08

<figure><img src="../.gitbook/assets/Screenshot 2023-08-14 alle 20.53.11.png" alt=""><figcaption></figcaption></figure>

### SLIDE 02-09

Nessuna tecnologia è una soluzione miracolosa (come si dice "silver bullet"), e l'architettura a microservizi presenta una serie di notevoli svantaggi e problematiche.&#x20;



Una sfida nell'utilizzo dell'architettura a microservizi è che non esiste un algoritmo concreto e ben definito per **decomporre** un sistema in servizi e per complicare ulteriormente le cose se si decompone un sistema in modo errato si costruirà un monolite distribuito cioè un sistema composto da servizi accoppiati che devono essere però deployati insieme. Un monolite distribuito presenta gli svantaggi sia dell'architettura monolitica che dell'architettura a microservizi.

####

#### I sistemi distribuiti sono complessi.

Un altro problema nell'utilizzo dell'architettura a microservizi è che gli sviluppatori devono affrontare l'ulteriore complessità di creare un **sistema distribuito**, infatti I servizi devono utilizzare un meccanismo di comunicazione tra processi e chiaramente questo è più complesso rispetto a una semplice chiamata di metodo. Inoltre, un servizio deve essere progettato per gestire fallimenti parziali e gestire il servizio remoto che potrebbe essere non disponibile o mostrare alta latenza.

Ogni servizio ha il proprio database, il che rende difficile implementare transazioni e interrogazioni che coinvolgono diversi servizi e come vedremo più avanti un'applicazione basata su microservizi deve utilizzare pattern noti come SAGA per mantenere la coerenza dei dati tra i servizi. Un'applicazione basata su microservizi non può recuperare dati da diversi servizi tramite semplici interrogazioni. Deve invece implementare interrogazioni utilizzando la composizione di API o viste CQRS.

Gli IDE e altri strumenti di sviluppo sono maggiormente focalizzati sulla costruzione di applicazioni monolitiche e non forniscono un reale supporto esplicito per lo sviluppo di applicazioni distribuite. Scrivere test automatizzati che coinvolgono più servizi distribuiti non è banale.&#x20;





L'architettura a microservizi introduce anche una significativa **complessità operativa**, perché abbiamo molte più istanze di diversi tipi di servizio che devono essere gestite in produzione con un alto livello di automazione. Questo comporta la conoscenza di diverse tecnologie come:

* Strumenti di distribuzione automatizzata
* Una piattaforma PaaS pronta all'uso
* Una piattaforma di orchestrazione come Docker Swarm o Kubernetes

Sempre nell'ottica della complessità operativa va detto che rilascio di funzionalità che coinvolgono più servizi richiede una coordinazione attenta tra i vari team di sviluppo. È necessario creare un piano di distribuzione che ordina i rilasci dei servizi in base alle dipendenze tra di essi.





Un altro problema nell'utilizzo dell'architettura a microservizi è la decisione su **quando** durante il ciclo di vita dell'applicazione si dovrebbe utilizzare questa architettura. Nella fase di sviluppo della prima versione di un'applicazione, spesso non si hanno i problemi che questa architettura risolve. Inoltre, l'uso di un'architettura elaborata e distribuita rallenterà lo sviluppo iniziale che deve essere invece molto rapido. Questo può essere un grande dilemma per le startup, dove il problema più grande è di solito come far evolvere rapidamente il modello di business e l'applicazione correlata. L'utilizzo dell'architettura a microservizi rende molto più difficile iterare rapidamente. Una startup dovrebbe quasi certamente iniziare con un'applicazione monolitica.

In seguito, però, quando il problema è come gestire la complessità, è a quel punto che ha senso scomporre funzionalmente l'applicazione in un insieme di microservizi. Potresti trovare difficile il refactoring a causa delle dipendenze intricate.





Come puoi vedere, l'architettura a microservizi offre molti vantaggi, ma ha anche alcuni drawback significativi e a causa di questi problemi l'adozione di un'architettura a microservizi non dovrebbe essere presa alla leggera. Tuttavia, per applicazioni complesse è di solito la scelta giusta. Siti ben noti come eBay, Amazon, Groupon sono tutti evoluti da un'architettura monolitica a un'architettura a microservizi.

