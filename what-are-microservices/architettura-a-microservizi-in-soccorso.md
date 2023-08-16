# Architettura a microservizi in soccorso

### SLIDE 1

Ma cosa sono esattamente i microservizi? Purtroppo, il nome non aiuta perché si mette troppo l'accento sulla dimensione e non sulle funzionalità. Esistono numerose definizioni dell'architettura a microservizi.&#x20;

Alcune interpretazioni prendono il nome troppo letteralmente e sostengono che un servizio dovrebbe essere minuscolo, ad esempio 100 linee di codice. Altre affermano che un servizio dovrebbe richiedere solo due settimane di sviluppo e c'è chi definisce _l'architettura a microservizi come un'architettura orientata ai servizi composta da elementi debolmente accoppiati con contesti delimitati._ Quest'ultima non è una cattiva definizione, ma risulta un po' generica

### Scale cube e microservizi

Una delle definizioni più accettate su cosa sia un'architettura a microservizi è ispirata dal libro di Martin Abbott e Michael Fisher, "The Art of Scalability" (2015). Questo libro descrive un modello di scalabilità tridimensionale utile: il cubo della scalabilità, illustrato nella. Il modello definisce tre modi per scalare un'applicazione: X, Y e Z.

<figure><img src="../.gitbook/assets/Screenshot 2023-08-11 alle 21.45.07.png" alt=""><figcaption></figcaption></figure>



* La scalabilità sull'asse X è un modo comune per scalare un'applicazione monolitica. La figura mostra come funziona la scalabilità sull'asse X. Si eseguono molteplici istanze dell'applicazione dietro un bilanciamento del carico. Il bilanciamento del carico distribuisce le richieste tra le N istanze identiche dell'applicazione. Questo è un ottimo modo per migliorare la capacità e la disponibilità di un'applicazione.

<figure><img src="../.gitbook/assets/Screenshot 2023-08-14 alle 20.29.30.png" alt=""><figcaption></figcaption></figure>

* Anche la scalabilità sull'asse Z esegue più istanze dell'applicazione monolitica, ma a differenza della scalabilità sull'asse X, ogni istanza è responsabile solo di un sottoinsieme dei dati. La prossima figura mostra come funziona la scalabilità sull'asse Z. Il router di fronte alle istanze utilizza un attributo della richiesta per instradarla verso l'istanza appropriata. Ad esempio, un'applicazione potrebbe instradare le richieste utilizzando l'identificativo dell'utente (userId). In questo esempio, ogni istanza dell'applicazione è responsabile di un sottoinsieme di utenti. Il router utilizza l'identificativo dell'utente specificato dall'intestazione di autorizzazione della richiesta per selezionare una delle istanze disponibili. In questo esempio, ogni istanza dell'applicazione è responsabile di un sottoinsieme di utenti. Il router utilizza l'identificativo dell'utente (userId) specificato dall'intestazione di autorizzazione della richiesta per selezionare una delle N istanze identiche dell'applicazione. La scalabilità sull'asse Z è un ottimo modo per scalare un'applicazione per gestire volumi crescenti di transazioni e dati.



<figure><img src="../.gitbook/assets/Screenshot 2023-08-14 alle 20.30.04.png" alt=""><figcaption></figcaption></figure>

*   La scalabilità sull'asse X e Z migliora la capacità e la disponibilità dell'applicazione. Tuttavia, nessuno dei due approcci risolve il problema dell'aumento della complessità dello sviluppo e dell'applicazione. Per risolvere questi problemi, è necessario applicare la scalabilità sull'asse Y, o decomposizione funzionale. La prossima figura mostra come funziona la scalabilità sull'asse Y: suddividendo un'applicazione monolitica in un insieme di servizi.

    Un servizio è quindi una mini applicazione che implementa funzionalità strettamente focalizzate, come la gestione degli ordini, la gestione dei clienti e così via. Un servizio viene scalato utilizzando la scalabilità sull'asse X, anche se alcuni servizi possono utilizzare anche la scalabilità sull'asse Z. Ad esempio, il servizio degli Ordini è costituito da un insieme di istanze di servizio bilanciate. La definizione di alto livello dell'architettura a microservizi (microservices) è uno stile architetturale che scompone funzionalmente un'applicazione in un insieme di servizi. Nota che questa definizione non dice nulla sulla dimensione. Quello che conta è che ogni servizio abbia un insieme di responsabilità focalizzato e coeso.

### Microservizi come forma di modularità

La modularità è essenziale nello sviluppo di applicazioni grandi e complesse. Un'applicazione moderna come UrbanEats è troppo grande per essere sviluppata da un singolo individuo. È anche troppo complessa per essere compresa da una sola persona. Le applicazioni devono essere scomposte in moduli che sono sviluppati e compresi da persone diverse. In un'applicazione monolitica, i moduli vengono definiti utilizzando una combinazione di costrutti del linguaggio di programmazione (come i pacchetti Java) e artefatti di compilazione (come i file JAR di Java). Tuttavia, come hanno scoperto gli sviluppatori di UrbanEats, questo approccio tende a non funzionare bene nella pratica. Le applicazioni monolitiche a lunga durata tendono spesso a degenerare in grandi masse disordinate.

L'architettura a microservizi utilizza i servizi come unità di modularità. Un servizio ha un'API, che rappresenta un confine impermeabile difficile da violare. Non è possibile aggirare l'API e accedere a una classe interna come si può fare con un pacchetto Java. Di conseguenza, è molto più semplice preservare la modularità dell'applicazione nel tempo. Ci sono altri vantaggi nell'utilizzare i servizi come mattoni fondamentali, inclusa la capacità di distribuirli e scalarli in modo indipendente.

Una caratteristica chiave dell'architettura a microservizi è che i servizi sono debolmente accoppiati e comunicano solo tramite API. Un modo per ottenere un accoppiamento debole è che ciascun servizio abbia il proprio datastore.&#x20;

Nell'online store, ad esempio, il servizio degli Ordini ha un database che include la tabella ORDERS, mentre il servizio dei Clienti ha il suo database con la tabella CUSTOMERS. Durante lo sviluppo, gli sviluppatori possono modificare lo schema di un servizio senza dover coordinarsi con gli sviluppatori che lavorano su altri servizi. Durante l'esecuzione, i servizi sono isolati l'uno dall'altro; ad esempio, un servizio non verrà mai bloccato perché un altro servizio detiene un lock sul database.

Nota: l requisito che ogni servizio abbia il proprio database non implica necessariamente che debba avere il proprio server di database. Ad esempio, non è necessario spendere 10 volte di più per le licenze di Oracle RDBMS

### UrbanEats microservice architecture

Diamo un'occhiata veloce a cosa significa applicare la scalabilità Y-axis a questa applicazione. Se applichiamo la decomposizione Y-axis all'applicazione FTGO, otteniamo l'architettura mostrata nella seguente L'applicazione decomposta è composta da numerosi servizi frontend e backend. Applicheremmo anche la scalabilità X-axis e, eventualmente, la scalabilità Z-axis, in modo che durante l'esecuzione ci siano più istanze di ciascun servizio.

<figure><img src="../.gitbook/assets/Screenshot 2023-08-14 alle 20.39.34.png" alt=""><figcaption></figcaption></figure>

I servizi frontend includono un API gateway e l'interfaccia utente web del ristorante. L'API gateway, che svolge il ruolo di una facciata e viene descritto dettagliatamente nel capitolo 8, fornisce le API REST utilizzate dalle applicazioni mobili dei consumatori e dei corrieri. L'interfaccia utente web del ristorante implementa l'interfaccia web utilizzata dai ristoranti per gestire i menu e elaborare gli ordini.

La logica di business dell'applicazione FTGO è composta da numerosi servizi backend. Ogni servizio backend ha una REST API e il proprio datastore privato. I servizi backend includono i seguenti:

* **OrderService**: Gestisce gli ordini&#x20;
* **DeliveryService**: Gestisce la consegna degli ordini dai ristoranti ai consumatori&#x20;
* **RestaurantService**: Mantiene le informazioni sui ristoranti&#x20;
* **KitchenService**: Gestisce la preparazione degli ordini&#x20;
* **AccountingService**: Gestisce la fatturazione e i pagamenti

Molti servizi corrispondono ai moduli descritti in precedenza in questo capitolo. Ciò che è diverso è che ogni servizio e la sua API sono chiaramente definiti. Ciascuno può essere sviluppato, testato, implementato e scalato in modo indipendente. Inoltre, questa architettura riesce bene a preservare la modularità. Uno sviluppatore non può aggirare l'API di un servizio e accedere ai suoi componenti interni.

### UrbanEats microservice architecture

Some critics of the microservice architecture claim it’s nothing new—it’s service- oriented architecture (SOA). At a very high level, there are some similarities. SOA and the microservice architecture are architectural styles that structure a system as a set of services. Alcuni critici dell'architettura a microservizi affermano che non sia niente di nuovo, ma una forma di architettura orientata ai servizi (SOA). A un livello molto alto, ci sono alcune somiglianze. SOA e l'architettura a microservizi sono stili architetturali che strutturano un sistema come un insieme di servizi.

<figure><img src="../.gitbook/assets/Screenshot 2023-08-14 alle 20.44.47.png" alt=""><figcaption></figcaption></figure>

L'architettura SOA e l'architettura a microservizi utilizzano solitamente stack tecnologici diversi. Le applicazioni SOA utilizzano tipicamente tecnologie pesanti come SOAP e altri standard WS\*. Spesso utilizzano un ESB, un "smart pipe" che contiene logica di business e di elaborazione dei messaggi per integrare i servizi. Le applicazioni costruite utilizzando l'architettura a microservizi tendono a utilizzare tecnologie leggere e open source. I servizi comunicano tramite "dumb pipes", come i message broker o protocolli leggeri come REST o gRPC.

SOA e l'architettura a microservizi differiscono anche nel modo in cui trattano i dati. Le applicazioni SOA hanno tipicamente un modello di dati globale e condividono database. In contrasto, come accennato in precedenza, nell'architettura a microservizi ogni servizio ha il proprio database. Inoltre, come descritto nel capitolo 2, ogni servizio di solito è considerato avere il proprio modello di dominio.

Un'altra differenza chiave tra SOA e l'architettura a microservizi è la dimensione dei servizi. SOA è tipicamente utilizzato per integrare applicazioni monolitiche grandi e complesse. Anche se i servizi in un'architettura a microservizi non sono sempre piccoli, sono quasi sempre molto più piccoli. Di conseguenza, un'applicazione SOA di solito è composta da alcuni servizi grandi, mentre un'applicazione basata su microservizi di solito è composta da decine o centinaia di servizi più piccoli.
