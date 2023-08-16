# Il linguaggio dei pattern dell'architettura a microservizi

L'architettura e il design si basano interamente sulla presa di decisioni. È necessario decidere se l'architettura monolitica o quella a microservizi è la soluzione migliore per la propria applicazione. Quando si prendono tali decisioni, è necessario valutare molte scelte e compromessi. Se si opta per l'architettura a microservizi, sarà necessario affrontare numerose problematiche. Un modo efficace per descrivere le diverse opzioni architettoniche e di design e migliorare il processo decisionale è utilizzare un linguaggio di pattern. Vediamo prima perché abbiamo bisogno di pattern e di un linguaggio di pattern, e poi esploreremo il linguaggio dei pattern dell'architettura a microservizi.

I microservizi non sono immuni al fenomeno della "palla d'argento" (ossia alla tendenza di cercare soluzioni miracolose per risolvere tutti i problemi). Se questa architettura è appropriata per la tua applicazione dipende da molti fattori. Di conseguenza, è un consiglio poco valido suggerire sempre di utilizzare l'architettura a microservizi, ma è altrettanto sbagliato consigliare di non usarla mai. Come per molte cose, dipende.

Un ottimo modo per discutere e descrivere una tecnologia è utilizzare il formato del pattern, perché è oggettivo. Quando si descrive una tecnologia nel formato del pattern, è necessario, ad esempio, descrivere gli svantaggi. Diamo un'occhiata al formato del pattern.

### Il linguaggio dei patterns

Un pattern è una soluzione riutilizzabile a un problema che si verifica in un contesto specifico. È un'idea che ha le sue origini nell'architettura del mondo reale e che si è dimostrata utile nell'architettura e nel design del software. Il concetto di pattern è stato creato da Christopher Alexander, un architetto del mondo reale. Ha anche creato il concetto di un linguaggio di pattern, una collezione di pattern correlati che risolvono problemi all'interno di un determinato dominio.

Un pattern software risolve un problema di architettura o design del software definendo un insieme di elementi software che collaborano tra loro. Immaginiamo, ad esempio, che tu stia costruendo un'applicazione bancaria che deve supportare diverse politiche di scoperto. Ogni politica definisce limiti sul saldo di un conto e le commissioni addebitate per un conto in rosso. Puoi risolvere questo problema usando il pattern Strategy, che è un noto pattern dal libro classico "Design Patterns". La soluzione definita dal pattern Strategy è composta da tre parti:

* Un'interfaccia di strategia chiamata Overdraft che incapsula l'algoritmo di scoperto
* Una o più classi di strategia concrete, una per ciascun contesto particolare
* La classe Account che utilizza l'algoritmo Il pattern Strategy è un pattern di design orientato agli oggetti, quindi gli elementi della soluzione sono classi. Più avanti in questa sezione, descriverò pattern di design ad alto livello, dove la soluzione è composta da servizi che collaborano tra loro.

Una delle ragioni per cui i pattern sono preziosi è che un pattern deve descrivere il contesto in cui si applica. L'idea che una soluzione sia specifica per un contesto particolare e potrebbe non funzionare bene in altri contesti rappresenta un miglioramento rispetto a come la tecnologia veniva solitamente discussa in passato. Ad esempio, una soluzione che risolve il problema a scala di Netflix potrebbe non essere l'approccio migliore per un'applicazione con meno utenti.

Il valore di un pattern, tuttavia, va oltre il fatto di richiederti di considerare il contesto di un problema. Ti costringe a descrivere altri aspetti critici ma spesso trascurati di una soluzione. Una struttura di pattern comunemente utilizzata include tre sezioni particolarmente preziose:

* **Forze (Forces):** La sezione "Forze" di un pattern descrive le forze (questioni) che devi affrontare quando risolvi un problema in un determinato contesto. Le forze possono entrare in conflitto, quindi potrebbe non essere possibile risolverle tutte. Quali forze sono più importanti dipende dal contesto. Devi dare la priorità alla risoluzione di alcune forze rispetto ad altre. Ad esempio, il codice deve essere facile da comprendere e avere buone prestazioni. Il codice scritto in uno stile reattivo ha prestazioni migliori rispetto al codice sincrono, ma spesso è più difficile da comprendere. Elencare esplicitamente le forze è utile perché chiarisce quali questioni devono essere risolte.
*   **Contesto risultante:** La sezione "Contesto risultante" di un pattern descrive le conseguenze dell'applicazione del pattern. È composta da tre parti:

    * Vantaggi: i vantaggi del pattern, tra cui le forze risolte
    * Svantaggi: gli svantaggi del pattern, tra cui le forze irrisolte
    * Problemi: i nuovi problemi introdotti dall'applicazione del pattern

    Il contesto risultante fornisce una visione più completa e meno distorta della soluzione, consentendo decisioni di progettazione migliori.
*   **Pattern correlati**:&#x20;

    La sezione "Pattern correlati" di un pattern descrive la relazione tra il pattern stesso e altri pattern ed è possibile organizzare i pattern che affrontano problemi in un'area specifica in gruppi. La descrizione esplicita dei pattern correlati fornisce una guida preziosa su come risolvere efficacemente un problema particolare. Ci sono cinque tipi di relazioni tra i pattern:

    * _Predecessore_: un pattern predecessore è un pattern che motiva la necessità di questo pattern. Ad esempio, il pattern Architettura a microservizi è il predecessore degli altri pattern nel linguaggio dei pattern, ad eccezione del pattern Architettura monolitica.
    * _Successore_: un pattern che risolve un problema introdotto da questo pattern. Ad esempio, se si applica il pattern Architettura a microservizi, è necessario applicare numerosi pattern successori, tra cui i pattern di scoperta del servizio e il pattern Circuit Breaker.
    * _Alternativa_: un pattern che fornisce una soluzione alternativa a questo pattern. Ad esempio, il pattern Architettura monolitica e il pattern Architettura a microservizi sono modi alternativi di progettare un'applicazione. Si sceglie uno dei due.
    * _Generalizzazione_: un pattern che è una soluzione generale a un problema. Ad esempio, nel capitolo 12 si apprenderà sulle diverse implementazioni del pattern Un singolo servizio per host.
    * _Specializzazione_: una forma specializzata di un particolare pattern.



    <figure><img src="../.gitbook/assets/Screenshot 2023-08-15 alle 10.19.06 (1).png" alt=""><figcaption></figcaption></figure>

### Panoramica del linguaggio di pattern dell'architettura a microservizi

l linguaggio di pattern dell'architettura a microservizi è una raccolta di pattern che ti aiutano a progettare un'applicazione utilizzando l'architettura a microservizi. La mostra la struttura ad alto livello del linguaggio di pattern. Il linguaggio di pattern ti aiuta prima a decidere se utilizzare l'architettura a microservizi.&#x20;

Esso descrive l'architettura monolitica e l'architettura a microservizi, insieme ai loro vantaggi e svantaggi. Successivamente, se l'architettura a microservizi è adatta per la tua applicazione, il linguaggio di pattern ti aiuta a utilizzarla efficacemente risolvendo varie problematiche architetturali e di design.&#x20;

Il linguaggio di pattern è composto da diversi gruppi di pattern. Sulla sinistra c'è il gruppo di pattern dell'architettura dell'applicazione, il pattern dell'architettura monolitica e il pattern dell'architettura a microservizi. Questi sono i pattern di cui abbiamo parlato in precedenza.&#x20;

Il resto del linguaggio di pattern è composto da gruppi di pattern che sono soluzioni per le problematiche introdotte dall'utilizzo del pattern dell'architettura a microservizi. I pattern sono anche suddivisi in tre livelli:&#x20;

* Pattern dell'infrastruttura, risolvono problemi che sono principalmente questioni di infrastruttura al di fuori dello sviluppo
* Infrastruttura dell'applicazione, Sono per problemi di infrastruttura che influenzano anche lo sviluppo
* Pattern dell'applicazione, Risolvono problemi affrontati dagli sviluppatori

Questi pattern sono raggruppati in base al tipo di problema che risolvono

#### Pattern di comunicazione

Un'applicazione costruita utilizzando l'architettura a microservizi è un sistema distribuito. Di conseguenza, la comunicazione tra processi (IPC) è una parte importante dell'architettura a microservizi. Devi prendere una serie di decisioni architettoniche e di design su come i tuoi servizi comunicano tra di loro e con il mondo esterno. La Figura mostra i pattern di comunicazione, che sono organizzati in cinque gruppi:

* _Stile di comunicazione_: quale meccanismo IPC dovresti utilizzare?
* _Scoperta_: come un client di un servizio determina l'indirizzo IP di un'istanza di servizio in modo che, ad esempio, possa effettuare una richiesta HTTP?
* _Affidabilità_: come puoi garantire che la comunicazione tra i servizi sia affidabile anche se i servizi possono non essere disponibili?
* _Messaggistica transazionale_: come dovresti integrare l'invio di messaggi e la pubblicazione di eventi con le transazioni di database che aggiornano i dati aziendali?
* _API esterne_: come comunicano i clienti della tua applicazione con i servizi?

<figure><img src="../.gitbook/assets/Screenshot 2023-08-15 alle 10.25.33.png" alt=""><figcaption></figcaption></figure>

#### Data consistency pattern per le transazioni

Come già accennato in precedenza, al fine di garantire un accoppiamento ridotto, ogni servizio ha il proprio database. Purtroppo, avere un database per servizio introduce alcune problematiche significative e l'approccio tradizionale che utilizza transazioni distribuite (2PC) non è un'opzione valida per un'applicazione moderna. Al contrario, un'applicazione deve mantenere la coerenza dei dati utilizzando il pattern Saga.&#x20;

<figure><img src="../.gitbook/assets/Screenshot 2023-08-15 alle 10.28.58.png" alt=""><figcaption></figcaption></figure>

#### Pattern per interrogare i dati in una architettura a microservizi

Un'altra problematica nell'uso di un database per servizio è che alcune query devono unire dati di proprietà di più servizi. I dati di un servizio sono accessibili solo tramite la sua API, quindi non è possibile utilizzare query distribuite sul suo database. La figura 1.14 mostra alcuni pattern che è possibile utilizzare per implementare le query. A volte è possibile utilizzare il pattern di composizione dell'API, che richiama le API di uno o più servizi e aggrega i risultati. Altre volte, è necessario utilizzare il pattern CQRS (Command Query Responsibility Segregation), che mantiene una o più repliche dei dati facilmente interrogabili.&#x20;

\
&#x20;

<figure><img src="../.gitbook/assets/Screenshot 2023-08-15 alle 10.30.54.png" alt=""><figcaption></figcaption></figure>

#### Service Deployment Patterns

Deploying a monolithic application isn’t always easy, but it is straightforward in the sense that there is a single application to deploy. You have to run multiple instances of the application behind a load balancer.

In comparison, deploying a microservices-based application is much more com- plex. There may be tens or hundreds of services that are written in a variety of lan- guages and frameworks. There are many more moving parts that need to be managed. Figure 1.15 shows the deployment patterns.

The traditional, and often manual, way of deploying applications in a language- specific packaging format, for example WAR files, doesn’t scale to support a microser- vice architecture. You need a highly automated deployment infrastructure. Ideally, you should use a deployment platform that provides the developer with a simple UI (command-line or GUI) for deploying and managing their services. The deployment platform will typically be based on virtual machines (VMs), containers, or serverless technology.&#x20;

<figure><img src="../.gitbook/assets/Screenshot 2023-08-15 alle 10.32.50 (1).png" alt=""><figcaption></figcaption></figure>

#### Pattern di osservabilità

na parte fondamentale nell'operare un'applicazione è comprendere il suo comportamento in esecuzione e risolvere problemi come richieste fallite e alta latenza. Sebbene capire e risolvere i problemi di un'applicazione monolitica non sia sempre semplice, aiuta il fatto che le richieste vengano gestite in modo semplice e diretto. Ogni richiesta in ingresso viene bilanciata su un'istanza specifica dell'applicazione, che effettua alcune chiamate al database e restituisce una risposta. Ad esempio, se è necessario capire come è stata gestita una particolare richiesta, si guarda il file di registro dell'istanza dell'applicazione che ha gestito la richiesta.

In contrasto, capire e diagnosticare i problemi in un'architettura a microservizi è molto più complicato. Una richiesta può passare attraverso più servizi prima che una risposta venga restituita al client. Di conseguenza, non c'è un unico file di registro da esaminare. Allo stesso modo, i problemi di latenza sono più difficili da diagnosticare perché ci sono più possibili colpevoli.

Puoi utilizzare i seguenti pattern per progettare servizi osservabili:

* API di controllo dello stato di salute: esponi un punto finale che restituisce lo stato di salute del servizio.
* Aggregazione dei log: registra l'attività del servizio e scrivi i log in un server di registrazione centralizzato, che fornisce ricerca e avvisi.
* Tracciamento distribuito: assegna a ciascuna richiesta esterna un ID univoco e traccia le richieste mentre fluiscono tra i servizi.
* Monitoraggio delle eccezioni: segnala le eccezioni a un servizio di monitoraggio delle eccezioni, che deduplica le eccezioni, avvisa gli sviluppatori e tiene traccia della risoluzione di ciascuna eccezione.
* Metriche dell'applicazione: mantieni metriche come contatori e indicatori e esponile a un server di metriche.
* Audit logging: registra le azioni dell'utente.

#### Pattern per l'automazione dei test

L'architettura a microservizi rende i singoli servizi più facili da testare perché sono molto più piccoli rispetto all'applicazione monolitica. Tuttavia, allo stesso tempo, è importante testare che i diversi servizi funzionino insieme evitando di utilizzare test end-to-end complessi, lenti e fragili che testano più servizi insieme. Ecco alcuni pattern per semplificare i test testando i servizi in isolamento:

* Test di contratto guidato dal consumatore: verifica che un servizio soddisfi le aspettative dei suoi clienti.
* Test di contratto lato consumatore: verifica che il client di un servizio possa comunicare con il servizio.
* Test di componente di servizio: testa un servizio in isolamento.

#### Cross-cutting concerns patterns&#x20;

Nell'architettura a microservizi, ci sono numerose preoccupazioni che ogni servizio deve implementare, tra cui i pattern di osservabilità e i pattern di scoperta. Deve anche implementare il pattern Configurazione Esteriorizzata, che fornisce parametri di configurazione come credenziali del database a un servizio in fase di esecuzione. Quando si sviluppa un nuovo servizio, reimplementare da zero queste preoccupazioni richiederebbe troppo tempo. Un approccio molto migliore è applicare il pattern Microservice Chassis e costruire i servizi su un framework che gestisca queste preoccupazioni.&#x20;

#### Security Pattern

In un'architettura a microservizi, gli utenti vengono tipicamente autenticati dalla API gateway. Deve quindi passare informazioni sull'utente, come identità e ruoli, ai servizi che invoca. Una soluzione comune è applicare il pattern Token di Accesso. La API gateway passa un token di accesso, come JWT (JSON Web Token), ai servizi, che possono convalidare il token e ottenere informazioni sull'utente. Non sorprendentemente, i pattern nel linguaggio dei pattern dell'architettura a microservizi sono focalizzati sulla risoluzione di problemi architettonici e di progettazione.\
