# 04-02-Coordinating Sagas

## SLIDE 04-02-01

L'implementazione di una saga consiste nell'implementazione della logica che coordina i passaggi della saga. Quando una saga viene avviata da un comando di sistema, la logica di coordinamento deve selezionare e comunicare al primo attore della saga di eseguire una transazione locale. Una volta completata questa transazione, il coordinamento del sequenziamento della saga seleziona ed invoca il partecipante successivo della saga. Questo processo continua finché la saga non ha eseguito tutti i passaggi. Se una qualsiasi transazione locale fallisce, la saga deve eseguire le transazioni di compensazione in ordine inverso. Ci sono diverse modalità diverse per strutturare la logica di coordinamento di una saga:&#x20;

* **Coreografia (Choreography)** - Distribuire la decisione e il sequenziamento tra i partecipanti della saga che comunicano principalmente scambiando eventi.
* **Orchestrare (Orchestration)** - Centralizzare la logica di coordinamento di una saga in una classe Orchestratore di saga. L'orchestratore di saga invia messaggi di comando ai partecipanti della saga dicendo loro quali operazioni eseguire.

## SLIDE 04-02-02

Un modo per implementare una saga è utilizzare quella che possiamo chiamare "**una coreografia**", qui non c'è un coordinatore centrale che dice ai partecipanti della saga cosa fare infatti i partecipanti della saga si iscrivono agli eventi degli altri partecipanti e rispondono di conseguenza.

<figure><img src="../.gitbook/assets/Screenshot 2023-08-22 alle 12.40.22.png" alt=""><figcaption></figcaption></figure>

Il percorso positivo attraverso questa saga è il seguente:

1. Il servizio Order crea un ordine nello stato _APPROVAL\_PENDING_ e pubblica un evento _OrderCreated_.
2. Il servizio Consumer consuma l'evento OrderCreated, verifica che il consumer possa effettuare l'ordine e pubblica un evento ConsumerVerified.
3. Il servizio Kitchen consuma l'evento OrderCreated, valida l'ordine, crea un Ticket nello stato CREATE\_PENDING e pubblica un evento TicketCreated.
4. Il servizio Accounting consuma l'evento OrderCreated e crea un'autorizzazione di carta di credito nello stato PENDING.
5. Il servizio Accounting consuma gli eventi TicketCreated e ConsumerVerified, addebita la carta di credito del consumatore e pubblica un evento CreditCardAuthorized.
6. Il servizio Kitchen consuma l'evento CreditCardAuthorized e cambia lo stato del Ticket in AWAITING\_ACCEPTANCE.
7. Il servizio Order riceve gli eventi CreditCardAuthorized, cambia lo stato dell'Ordine in APPROVED e pubblica un evento OrderApproved.

La _Create Order Saga_ deve anche gestire lo scenario in cui un partecipante della saga rifiuta l'Ordine e pubblica qualche tipo di _evento di errore_. Ad esempio, l'autorizzazione della carta di credito del consumatore potrebbe fallire. La saga deve eseguire le transazioni di compensazione per annullare ciò che è stato già fatto.&#x20;

## SLIDE 04-02-03

<figure><img src="../.gitbook/assets/Screenshot 2023-08-22 alle 12.44.09.png" alt=""><figcaption></figcaption></figure>

In figura vediamo il flusso di eventi quando il servizio Accounting non può autorizzare la carta di credito del consumatore.

1. Il servizio Order crea un ordine nello stato APPROVAL\_PENDING e pubblica un evento OrderCreated.
2. Il servizio Consumer consuma l'evento OrderCreated, verifica che il consumatore possa effettuare l'ordine e pubblica un evento ConsumerVerified.
3. Il servizio Kitchen consuma l'evento OrderCreated, valida l'ordine, crea un Ticket nello stato CREATE\_PENDING e pubblica un evento TicketCreated.
4. Il servizio Accounting consuma l'evento OrderCreated e crea un'autorizzazione di carta di credito nello stato PENDING.
5. Il servizio Accounting consuma gli eventi TicketCreated e ConsumerVerified, addebita la carta di credito del consumatore e pubblica un evento Credit Card Authorization Failed.
6. Il servizio Kitchen consuma l'evento Credit Card Authorization Failed e cambia lo stato del Ticket in REJECTED.
7. Il servizio Order consuma l'evento Credit Card Authorization Failed e cambia lo stato dell'Ordine in REJECTED.

I partecipanti delle saghe basate sulla coreografia interagiscono tramite il meccanismo di **publish/subscribe** e ci sono alcune considerazioni da fare al riguardo

## SLIDE 04-02-04

Ci sono un paio di questioni legate alla comunicazione tra servizi che dobbiamo considerare quando implementiamo saghe basate sulla coreografia. Il primo problema è assicurarsi che un partecipante della saga aggiorni il suo database e pubblichi un evento come parte di una transazione di database. Ogni passo di una saga basata sulla coreografia aggiorna il database e pubblica un evento.&#x20;

Ad esempio, nella Create Order Saga, il servizio Kitchen riceve un evento Consumer Verified, crea un Ticket e pubblica un evento Ticket Created. È essenziale che l'aggiornamento del database e la pubblicazione dell'evento avvengano atomicamente. Di conseguenza, per comunicare in modo affidabile, i partecipanti delle saghe devono utilizzare la messaggistica transazionale

La seconda questione che devi considerare è assicurarti che un partecipante della saga sia in grado di associare ogni evento che riceve ai propri dati. Ad esempio, quando il servizio Order riceve un evento Credit Card Authorized, deve essere in grado di cercare l'Ordine corrispondente. La soluzione è che un partecipante della saga pubblichi eventi contenenti un ID di correlazione, che è un dato che consente agli altri partecipanti di effettuare la mappatura.

Ad esempio, i partecipanti della Create Order Saga possono utilizzare l'ID dell'ordine (orderId) come ID di correlazione che viene passato da un partecipante al successivo. Il servizio Accounting pubblica un evento Credit Card Authorized contenente l'orderId dall'evento Ticket Created. Quando il servizio Order riceve un evento Credit Card Authorized, utilizza l'orderId per recuperare l'Ordine corrispondente. Allo stesso modo, il servizio Kitchen utilizza l'orderId da quell'evento per recuperare il Ticket corrispondente.

#### Vantaggi / Svantaggi saghe basate su coreografie

Le saghe basate sulla coreografia presentano diversi vantaggi:

* **Semplicità**: i servizi pubblicano eventi quando creano, aggiornano o eliminano oggetti di business.
* **Accoppiamento lasco**: i partecipanti si sottoscrivono agli eventi e non hanno conoscenza diretta l'uno dell'altro.

Tuttavia, ci sono anche alcuni svantaggi:

* Maggior difficoltà di **comprensione**: a differenza dell'orchestrazione, non c'è un unico punto nel codice che definisce la saga perché la coreografia distribuisce l'implementazione della saga tra i servizi. Di conseguenza, a volte può essere difficile per uno sviluppatore capire come funziona una determinata saga.
* Dipendenze **cicliche** tra i servizi: i partecipanti della saga si sottoscrivono agli eventi l'uno dell'altro, creando spesso dipendenze cicliche. Ad esempio nel nostro caso abbiamo delle dipendenze cicliche, come _Order Service_ -> _Accounting Service_ -> _Order Service_. Di base non è necessariamente un problema ma comunque bisogna farci attenzione
* Rischio potenziale di **accoppiamento stretto**: ogni partecipante della saga deve sottoscriversi a tutti gli eventi che li riguardano. Ad esempio, il servizio Accounting deve sottoscriversi a tutti gli eventi che causano l'addebito o il rimborso della carta di credito del consumatore. Di conseguenza, c'è il rischio che debba essere aggiornato sincronicamente con il ciclo di vita dell'ordine implementato dal servizio Order.

La coreografia può funzionare bene per le saghe semplici, ma a causa di questi svantaggi spesso è meglio utilizzare l'orchestrazione per le saghe più complesse.&#x20;

Vediamo come funziona l'orchestrazione.

## SLIDE 04-02-05

<figure><img src="../.gitbook/assets/Screenshot 2023-08-22 alle 14.28.05.png" alt=""><figcaption></figcaption></figure>

**L'orchestrazione** è un altro modo per implementare le saghe definendo una classe Orchestratrice il cui unico compito è dire ai partecipanti della saga cosa fare. L'orchestratore della saga comunica con i partecipanti utilizzando un'interazione basata su comandi e risposte asincrone.&#x20;

Per eseguire un passo della saga, l'orchestratore invia un messaggio di comando a un partecipante indicandogli quale operazione eseguire. Dopo che il partecipante della saga ha eseguito l'operazione, invia un messaggio di feedback.

L'orchestratore elabora quindi il messaggio e determina quale passo della saga eseguire successivamente. Per mostrare come funzionano le saghe basate sull'orchestrazione.&#x20;

La figura nella slide mostra il design della versione basata sull'orchestrazione della _Create Order Saga,_ orchestrata dalla classe CreateOrderSaga, che invoca i partecipanti della saga utilizzando richieste/risposte asincrone. Questa classe tiene traccia del processo e invia messaggi di comando ai partecipanti della saga, come ad esempio _Kitchen Service_ e _Consumer Service_. La classe _CreateOrderSaga_ legge i messaggi di risposta dal suo canale di risposta e quindi determina il passo successivo, se presente, nella saga.

Order Service prima crea un Ordine e un orchestratore Create Order Saga. Dopo di ciò, il flusso per il percorso positivo è il seguente:&#x20;

1. L'orchestratore della saga invia un comando Verify Consumer a Consumer Service
2. Consumer Service risponde con un messaggio Consumer Verified
3. L'orchestratore della saga invia un comando Create Ticket a Kitchen Service
4. Kitchen Service risponde con un messaggio Ticket Created
5. L'orchestratore della saga invia un messaggio Authorize Card a Accounting Service
6. Accounting Service risponde con un messaggio Card Authorized
7. L'orchestratore della saga invia un comando Approve Ticket a Kitchen Service
8. L'orchestratore della saga invia un comando Approve Order a Order Service

Nota che nell'ultimo passo, l'orchestratore della saga invia un messaggio di comando a Order Service, anche se è un componente di Order Service. In teoria, la Create Order Saga potrebbe approvare l'Ordine aggiornandolo direttamente. Ma per essere coerente, la saga tratta Order Service come un altro partecipante.

I diagrammi come questo rappresentano uno scenario per saga, ma è molto probabile che una saga abbia numerosi scenari. Ad esempio, la _Create Order Saga_ ha quattro scenari. Oltre al percorso positivo, la saga può fallire a causa di un errore in Consumer Service, Kitchen Service o Accounting Service. È utile, quindi, modellare una saga come una **macchina a stati**, poiché descrive tutti gli scenari possibili.

## SLIDE 04-02-06

Un buon modo per modellare un orchestratore di saga è come una **macchina a stati,** composta da un insieme di stati e un insieme di transizioni tra gli stati che vengono attivate dagli eventi.

Ogni transizione può avere un'azione, che per una saga è l'invocazione di un partecipante della saga. Le transizioni tra gli stati vengono attivate dalla completamento di una transazione locale eseguita da un partecipante della saga. Lo stato attuale e l'esito specifico della transazione locale determinano la transizione dello stato e quale azione, se del caso, eseguire.&#x20;

Qui in figura vediamo il modello di macchina a stati per la _Create Order Saga_. Questa macchina a stati è composta da _numerosi stati_, tra cui i seguenti:

* Verifica del **consumer** - Lo stato iniziale. Quando si trova in questo stato, la saga è in attesa che il Consumer Service verifichi che il consumatore possa effettuare l'ordine.
* **Creazione del Ticket** - La saga è in attesa di una risposta al comando Create Ticket.
* **Autorizzazione** della Carta - In attesa che l'Accounting Service autorizzi la carta di credito del consumatore.
* **Ordine Approvato** - Uno stato finale che indica che la saga è stata completata con successo.
* **Ordine Respinto** - Uno stato finale che indica che l'Ordine è stato respinto da uno dei partecipanti.

La macchina a stati definisce anche numerose transizioni di stato. Ad esempio, la macchina a stati transita dallo stato _Creazione del Ticket_ allo stato _Autorizzazione della Carta o allo stato Ordine Respinto_.&#x20;

Transita allo stato Autorizzazione della Carta quando riceve una risposta positiva al comando Create Ticket. In alternativa, se il Kitchen Service non è riuscito a creare il Ticket, la macchina a stati passa allo stato Ordine Respinto.&#x20;

L'azione iniziale della macchina a stati è inviare il comando VerifyConsumer al Consumer Service. La risposta dal Consumer Service attiva la prossima transizione di stato. Se il consumatore è stato verificato con successo, la saga crea il Ticket e transita allo stato Creazione del Ticket. Ma se la verifica del consumatore è fallita, la saga respinge l'Ordine e passa allo stato Ordine Respinto. La macchina a stati subisce numerose altre transizioni di stato, guidate dalle risposte dei partecipanti della saga, finché non raggiunge uno stato finale di Ordine Approvato o Ordine Respinto.

Ogni passo di una saga basata sull'orchestrazione consiste in un servizio che aggiorna un database e pubblica un messaggio. Ad esempio, il servizio Order Service persiste un ordine e un orchestratore di saga Create Order Saga e invia un messaggio al primo partecipante della saga. Un partecipante della saga, come il Kitchen Service, gestisce un messaggio di comando aggiornando il suo database e inviando un messaggio di risposta. Order Service elabora il messaggio di risposta del partecipante aggiornando lo stato dell'orchestratore di saga e inviando un messaggio di comando al prossimo partecipante della saga. Come descritto nel capitolo 3, un servizio deve utilizzare la messaggistica transazionale per aggiornare atomicamente il database e pubblicare messaggi. Più avanti nella sezione 4.4, descriverò in modo più dettagliato l'implementazione dell'orchestratore di Create Order Saga, inclusa la modalità in cui utilizza la messaggistica transazionale.

Esaminiamo i vantaggi e gli svantaggi dell'uso delle orchestrazioni delle saghe.

#### Vantaggi e svantaggi

Le saghe basate sull'orchestrazione hanno diversi vantaggi:

* Dipendenze più **semplici**: Un beneficio dell'orchestrazione è che non introduce dipendenze _cicliche_ perché l'orchestratore di saga invoca i partecipanti della saga, ma i partecipanti non invocano l'orchestratore. Di conseguenza, l'orchestratore dipende dai partecipanti ma non viceversa, quindi non ci sono dipendenze cicliche.
* **Minore dipendenza**: Ogni servizio implementa un'API invocata dall'orchestratore, quindi non ha bisogno di conoscere gli eventi pubblicati dai partecipanti della saga.
* Migliora la **separazione delle responsabilità** e s_emplifica la logica aziendale_: La logica di coordinamento della saga è localizzata nell'orchestratore di saga. Gli oggetti di dominio sono più semplici e non hanno conoscenza delle saghe a cui partecipano. Ad esempio, quando si utilizza l'orchestrazione, la classe Order non ha conoscenza di nessuna delle saghe, quindi ha un modello di macchina a stati più semplice. Durante l'esecuzione della Create Order Saga, passa direttamente dallo stato APPROVAL\_PENDING allo stato APPROVED. La classe Order non ha stati intermedi corrispondenti ai passi della saga. Di conseguenza, l'aspetto aziendale è molto più semplice.

L'orchestrazione ha uno **svantaggio**: si _centralizza_ troppa logica di business nell'orchestratore. Ciò porta a un design in cui l'orchestratore che la logica globale dice ai servizi semplici quali operazioni eseguire. Fortunatamente, è possibile evitare questo problema progettando orchestratori responsabili esclusivamente della sequenza e che non contengono altre logiche di business.

Implementare la logica di coordinamento per le saghe è solo uno dei problemi di progettazione che dobbiamo risolvere insieme alla mancanza di isolamento che andremo ora ad esaminare



