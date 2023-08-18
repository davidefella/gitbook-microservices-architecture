# Il linguaggio dei pattern dell'architettura a microservizi

## SLIDE 03-01

Come dicevamo quindi è necessario decidere se l'architettura monolitica o quella a microservizi è la soluzione migliore per la propria applicazione. Quando si prendono tali decisioni, è necessario valutare molte scelte e compromessi. Se si opta per l'architettura a microservizi, sarà comunque necessario affrontare numerose problematiche come abbiamo visto.&#x20;

Un modo efficace per descrivere le diverse opzioni architettoniche e di design e migliorare il processo decisionale è utilizzare un linguaggio di pattern. Vediamo prima perché abbiamo bisogno di pattern e di un linguaggio di pattern, e poi esploreremo il linguaggio dei pattern dell'architettura a microservizi.

I microservizi non sono immuni al fenomeno della "pallottola d'argento", se questa architettura è appropriata per la nostra applicazione dipende da molti fattori. Di conseguenza, è un consiglio poco valido suggerire sempre di utilizzare l'architettura a microservizi, ma è altrettanto sbagliato consigliare di non usarla mai. Come per molte cose nel nostro mondo IT la risposta è "dipende".

Un ottimo modo per discutere e descrivere una tecnologia è utilizzare il formato del pattern, perché è oggettivo. Quando si descrive una tecnologia nel formato del pattern, è necessario, ad esempio, descrivere gli svantaggi. Diamo un'occhiata al formato del pattern.

Un **pattern** è una soluzione riutilizzabile a un problema che si verifica in un contesto specifico. È un'idea che ha le sue origini nell'architettura del mondo reale e che si è dimostrata utile nell'architettura e nel design del software.&#x20;

Un **pattern software** quindi ci aiuta a risolvere un problema definendo un insieme di elementi che interagiscono tra loro e una delle ragioni per cui i pattern sono preziosi è che un pattern deve descrivere il contesto in cui si applica.&#x20;

L'idea che una soluzione sia specifica per un contesto particolare e potrebbe non funzionare bene in altri contesti rappresenta un miglioramento rispetto a come la tecnologia veniva solitamente discussa in passato. Ad esempio, una soluzione che risolve il problema di scalabilità di Netflix potrebbe non essere l'approccio migliore per un'applicazione con meno utenti.

Un Pattern inoltre ti costringe a **descrivere** altri aspetti critici ma spesso trascurati di una soluzione. In genere una struttura di pattern include tre sezioni:

* **Forze (Forces):** La sezione che descrive le forze (questioni) che devi affrontare quando risolvi un problema in un determinato contesto e che possono entrare in conflitto, quindi potrebbe non essere possibile risolverle tutte quali sono più importanti dipende dal contesto. Ad esempio, il codice deve essere facile da comprendere e avere buone prestazioni. Il codice scritto in uno stile reattivo ha prestazioni migliori rispetto al codice sincrono, ma spesso è più difficile da comprendere. Elencare esplicitamente le forze è utile perché chiarisce quali questioni devono essere risolte.
*   **Contesto risultante:** qui descriviamo le conseguenze dell'applicazione del pattern. È composta da tre parti:

    * Vantaggi: i vantaggi del pattern, tra cui le forze risolte
    * Svantaggi: gli svantaggi del pattern, tra cui le forze irrisolte
    * Problemi: i nuovi problemi introdotti dall'applicazione del pattern

    Il contesto risultante fornisce una visione più completa e reale
*   **Pattern correlati**:&#x20;

    Qui descriviamo la relazione tra il pattern stesso e altri pattern così da poterli organizzare per aree specifiche in gruppi. Ci sono cinque tipi di relazioni tra i pattern:

    * _Predecessore_: un pattern predecessore è un pattern che motiva la necessità di questo pattern.&#x20;
    * _Successore_: un pattern che risolve un problema introdotto da questo pattern.&#x20;
    * _Alternativa_: un pattern che fornisce una soluzione alternativa a questo pattern.&#x20;
    * _Generalizzazione_: un pattern che è una soluzione generale a un problema.
    * _Specializzazione_: una forma specializzata di un particolare pattern.



## SLIDE 03-02

<figure><img src="../.gitbook/assets/Screenshot 2023-08-15 alle 10.19.06 (1).png" alt=""><figcaption></figcaption></figure>

## SLIDE 03-03

<figure><img src="../.gitbook/assets/Screenshot 2023-08-16 alle 18.00.41.png" alt=""><figcaption></figcaption></figure>

Vediamo adesso una breve anticipazione di una raccolta di pattern nell'ambito dell'architettura a microservizi che ci aiuteranno a progettare un'applicazione utilizzando questa architettura.&#x20;

Il linguaggio di pattern è composto da diversi gruppi e suddivisi in tre livelli:&#x20;

* Pattern _infrastrutturali_ che risolvono problemi legati principalmente a questioni di infrastruttura al di fuori dello sviluppo
* Application infrastructure, per problemi di infrastruttura che però influenzano anche lo sviluppo
* Pattern di applicazione, risolvono problemi affrontati dagli sviluppatori

Questi pattern sono raggruppati in base al tipo di problema che risolvonoSL

## SLIDE 03-04

<figure><img src="../.gitbook/assets/Screenshot 2023-08-15 alle 10.25.33.png" alt=""><figcaption></figcaption></figure>

Un'applicazione costruita utilizzando l'architettura a microservizi è un sistema distribuito e di conseguenza la comunicazione tra processi (IPC) è una parte importante dell'architettura a microservizi. Bisogna prendere una serie di decisioni architettoniche e di design su come i servizi comunicano tra di loro e con il mondo esterno. Sono organizzati in cinque gruppi:

* _Stile di comunicazione_: quale meccanismo IPC dovresti utilizzare?
* _Discovery_: come un client di un servizio determina l'indirizzo IP di un'istanza di servizio in modo che, ad esempio, possa effettuare una richiesta HTTP?
* _Affidabilità_: come puoi garantire che la comunicazione tra i servizi sia affidabile anche se i servizi possono non essere disponibili?
* _Messaggistica transazionale_: come dovresti integrare l'invio di messaggi e la pubblicazione di eventi con le transazioni di database che aggiornano i dati aziendali?
* _API esterne_: come comunicano i clienti della tua applicazione con i servizi?

## SLIDE 03-05

<figure><img src="../.gitbook/assets/Screenshot 2023-08-15 alle 10.28.58.png" alt=""><figcaption></figcaption></figure>

Come già accennato in precedenza, al fine di garantire un accoppiamento ridotto, ogni servizio ha il proprio database. Purtroppo, avere un database per servizio introduce alcune problematiche significative e l'approccio tradizionale che utilizza transazioni distribuite (2PC) non è un'opzione valida per un'applicazione moderna. Al contrario, un'applicazione deve mantenere la coerenza dei dati utilizzando il pattern SAGA (che vedremo più avanti).&#x20;

## SLIDE 03-06

<figure><img src="../.gitbook/assets/Screenshot 2023-08-15 alle 10.30.54.png" alt=""><figcaption></figcaption></figure>

Un'altra problematica nell'uso di un database per servizio è che alcune query devono unire dati di proprietà di più servizi che sono accessibili solo tramite la sua API, quindi non è possibile utilizzare query distribuite sul suo database. La figura mostra alcuni pattern che è possibile utilizzare per implementare le query. A volte è possibile utilizzare il pattern di composizione dell'API, che richiama le API di uno o più servizi e aggrega i risultati. Altre volte, è necessario utilizzare il pattern CQRS (Command Query Responsibility Segregation), che mantiene una o più repliche dei dati facilmente interrogabili.&#x20;

## SLIDE 03-07

<figure><img src="../.gitbook/assets/Screenshot 2023-08-15 alle 10.32.50 (1).png" alt=""><figcaption></figcaption></figure>

Il rilascio di un'applicazione monolitica non è sempre facile, ma è pià diretto nel senso che c'è un'unica applicazione da rilasciare. In confronto, il rilascio di un'applicazione basata su microservizi è molto più complesso. Potrebbero esserci decine o centinaia di servizi scritti in diverse lingue e framework e quindi ci sono molte più componenti che devono essere gestite. Lo schema in figura mostra i pattern di rilascio. Il modo tradizionale e spesso manuale di distribuire applicazioni in un formato di packaging specifico per il linguaggio, ad esempio file WAR, non scala per supportare un'architettura a microservizi. Hai bisogno di un'infrastruttura di distribuzione altamente automatizzata. Idealmente, dovresti utilizzare una piattaforma di distribuzione che fornisca allo sviluppatore un'interfaccia utente semplice (a riga di comando o GUI) per distribuire e gestire i propri servizi. La piattaforma di distribuzione sarà tipicamente basata su macchine virtuali (VM), container o tecnologia serverless.

## SLIDE 03-08

Una parte fondamentale nell'operare un'applicazione è comprendere il suo comportamento in esecuzione e risolvere problemi come richieste fallite e alta latenza. Sebbene capire e risolvere i problemi di un'applicazione monolitica non sia sempre semplice, aiuta il fatto che le richieste vengano gestite in modo semplice e diretto. Ogni richiesta in ingresso viene bilanciata su un'istanza specifica dell'applicazione, che effettua alcune chiamate al database e restituisce una risposta. Ad esempio, se è necessario capire come è stata gestita una particolare richiesta, si guarda il file di registro dell'istanza dell'applicazione che ha gestito la richiesta.

In contrasto, capire e diagnosticare i problemi in un'architettura a microservizi è molto più complicato. Una richiesta può passare attraverso più servizi prima che una risposta venga restituita al client. Di conseguenza, non c'è un unico file di registro da esaminare. Allo stesso modo, i problemi di latenza sono più difficili da diagnosticare perché ci sono più possibili colpevoli.

Puoi utilizzare i seguenti pattern per progettare servizi osservabili:

* API di controllo dello stato di salute: esponi un punto finale che restituisce lo stato di salute del servizio.
* Aggregazione dei log: registra l'attività del servizio e scrivi i log in un server di registrazione centralizzato, che fornisce ricerca e avvisi.
* Tracciamento distribuito: assegna a ciascuna richiesta esterna un ID univoco e traccia le richieste mentre fluiscono tra i servizi.
* Monitoraggio delle eccezioni: segnala le eccezioni a un servizio di monitoraggio delle eccezioni, che deduplica le eccezioni, avvisa gli sviluppatori e tiene traccia della risoluzione di ciascuna eccezione.
* Metriche dell'applicazione: mantieni metriche come contatori e indicatori e esponile a un server di metriche.
* Audit logging: registra le azioni dell'utente.

## SLIDE 03-09

L'architettura a microservizi rende i singoli servizi più facili da testare perché sono molto più piccoli rispetto all'applicazione monolitica ma allo stesso tempo è importante testare che i diversi servizi funzionino insieme evitando di utilizzare test end-to-end complessi, lenti e fragili che testano più servizi insieme. Ecco alcuni pattern per semplificare i test testando i servizi in isolamento:

* Test di contratto guidato dal consumatore: verifica che un servizio soddisfi le aspettative dei suoi clienti.
* Test di contratto lato consumatore: verifica che il client di un servizio possa comunicare con il servizio.
* Test di componente di servizio: testa un servizio in isolamento.

## SLIDE 03-10

In un'architettura a microservizi gli utenti vengono tipicamente autenticati dalla API gateway e quindi deve passare informazioni sull'utente, come identità e ruoli, ai servizi che invoca. Una soluzione comune è applicare il pattern **Token di Accesso**. La API gateway passa un token di accesso, come JWT (JSON Web Token), ai servizi, che possono convalidare il token e ottenere informazioni sull'utente.\
