# 03-01-Overview

## SLIDE 03-01-00&#x20;

Le chiamate tra processi diversi su una rete (**inter-process**) sono molto diverse dalle chiamate all'interno di un singolo processo (**in-process**). È facile, ad esempio, pensare a un oggetto che effettua una chiamata di metodo su un altro oggetto (una classe) e quindi potrebbe venire facile mappare questa interazione anche per due microservizi che comunicano tramite una rete. Mettendo da parte il fatto che i microservizi non sono solo oggetti, questo tipo di paragone non si presta molto bene.

Analizziamo alcune di queste differenze.

#### Prestazioni

Le **prestazioni** di una chiamata _in-process_ sono fondamentalmente diverse da quelle di una chiamata _inter-process_. Quando effettuo una chiamata in-process, il compilatore e l'ambiente di runtime sottostante possono effettuare una serie di ottimizzazioni per ridurre l'impatto della chiamata che però non sono possibili con le chiamate inter-process. I pacchetti dati devono essere inviati e l'overhead di una chiamata inter-processo è significativo rispetto all'overhead di una chiamata in-process. Il primo è molto misurabile: il tempo di andata e ritorno di un singolo pacchetto in un data center si misura in millisecondi, mentre l'overhead di una chiamata di metodo è qualcosa di cui non devi preoccuparti.

Un'API che ha senso in-process potrebbe non averlo in situazioni inter-process ad esempio potrei effettuare mille chiamate attraverso un dominio API in-process senza preoccupazioni. Vorrei fare mille chiamate di rete tra due microservizi senza preoccupazioni? Forse no. Quando passo un parametro in un metodo, la struttura dati che passo di solito non si sposta: è più probabile che passi intorno a un puntatore a una posizione di memoria.&#x20;

Passare un oggetto o una struttura dati a un altro metodo non richiede necessariamente l'allocazione di più memoria per copiare i dati. Quando si effettuano chiamate tra microservizi su una rete, d'altra parte, i dati devono effettivamente essere serializzati in una forma che possa essere trasmessa su una rete. I dati devono quindi essere inviati e deserializzati all'altro capo. Pertanto, potremmo dover prestare più attenzione alla dimensione dei carichi utili inviati tra i processi. Queste considerazioni potrebbero portarci a ridurre la quantità di dati inviati o ricevuti, scegliere meccanismi di serializzazione più efficienti o persino trasferire i dati su un filesystem e passare intorno un riferimento a quella posizione del file.

#### Interfacce che cambiano

Apportare cambiamenti a un'interfaccia all'interno di un processo è relativamente semplice semplice soprattutto se codice che chiama l'interfaccia sono è pacchettizzato nello stesso processo. Infatti, se cambio una firma di metodo usando un IDE con capacità di rifattorizzazione, spesso l'IDE stesso rifattorizzerà automaticamente le chiamate a questo metodo. Implementare un tale cambiamento può essere fatto in modo atomico: entrambi i lati dell'interfaccia sono confezionati insieme in un singolo processo.

Tuttavia, con la comunicazione tra microservizi, il microservizio che espone un'interfaccia e i microservizi consumer che utilizzano quell'interfaccia sono microservizi separati che possono essere distribuiti autonomamente. Quando apportiamo un cambiamento incompatibile all'interfaccia di un microservizio, dobbiamo fare un'implementazione sincronizzata con i consumatori, assicurandoci che vengano aggiornati per utilizzare la nuova interfaccia, oppure trovare un modo per pianificare l'implementazione del nuovo contratto del microservizio.&#x20;

#### Gestione degli errori

All'interno di un processo, se chiamo un metodo, la natura degli errori tende a essere piuttosto diretta. Semplificando un minimo, gli errori sono o previsti e facili da gestire, oppure sono catastrofici al punto che propaghiamo l'errore lungo la pila delle chiamate. Gli errori, in generale, sono _deterministici_.

In un sistema distribuito, la natura degli errori può essere diversa perché siamo vulnerabili a una serie di errori che sono al di fuori del nostro controllo. Ad esempio Le reti vanno in timeout, I microservizi _downstream_ potrebbero essere temporaneamente non disponibili. Le reti si disconnettono, i container vengono terminati a causa dell'uso eccessivo di memoria e in situazioni estreme, parti del tuo data center possono prendere fuoco.

Vediamo in maniera molto semplificata cinque tipi di modalità di fallimento che è possibile osservare in una comunicazione tra processi:&#x20;

* **Crash failure** Tutto andava bene finché il server non è crashato. Riavvio!
* **Omission failure:** Abbiamo inviato qualcosa, ma non abbiamo ricevuto una risposta. Include anche situazioni in cui ti aspetti che un microservizio stia inviando messaggi (magari eventi) e si ferma improvvisamente.
* **Timing failure:** Qualcosa è successo troppo tardi (non l'hai ricevuto in tempo), oppure è successo troppo presto!
*   **Response failure**: Abbiamo ricevuto una risposta, ma sembra sbagliata. Ad esempio, hai richiesto un riepilogo dell'ordine, ma alcune informazioni necessarie mancano nella risposta.


* **Arbitrary failure:** Altrimenti noti come guasti bizantini, si verificano quando qualcosa è andato storto, ma i partecipanti non riescono a concordare se il guasto è avvenuto (o perché). Come suona, è un brutto momento per tutti.

Molti di questi errori sono spesso di natura _transitoria_: sono problemi di breve durata che potrebbero risolversi da soli. Considera la situazione in cui inviamo una richiesta a un microservizio ma non riceviamo risposta (un tipo di guasto _per omissione_). Questo potrebbe significare che il microservizio non ha mai ricevuto la richiesta, quindi dobbiamo inviarla nuovamente. Altri problemi non possono essere risolti facilmente e potrebbero richiedere l'intervento di un operatore umano. Di conseguenza, può diventare importante avere un insieme più ampio di semantiche per restituire gli errori in modo che i client possano intraprendere azioni appropriate.

HTTP è un esempio di protocollo che comprende l'importanza di questo concetto. Ogni risposta HTTP ha un codice, con i codici delle serie _400_ e _500_ riservati agli errori. I codici degli errori della serie 400 si riferiscono a errori di richiesta, essenzialmente un servizio downstream sta dicendo al client che c'è qualcosa di sbagliato nella richiesta originale. Pertanto, è probabile che dovresti rinunciare - ha senso ritentare una richiesta di "404 Not Found", ad esempio? I codici di risposta della serie 500 si riferiscono a problemi downstream, di cui un sottoinsieme indica al client che il problema potrebbe essere temporaneo. Un "503 Service Unavailable" indica che il server  non è in grado di gestire la richiesta, ma potrebbe essere uno stato temporaneo, nel qual caso un client upstream potrebbe decidere di riprovare la richiesta. D'altra parte, se un client riceve una risposta "501 Not Implemented", è improbabile che un nuovo tentativo sia di grande aiuto.

Sia che tu scelga un protocollo basato su HTTP per la comunicazione tra microservizi o meno, se disponi di un insieme ricco di semantiche riguardanti la natura dell'errore, renderai più facile per i client eseguire azioni di compensazione, il che a sua volta dovrebbe aiutarti a costruire sistemi più robusti.

## SLIDE 03-01-01

<figure><img src="../.gitbook/assets/Screenshot 2023-08-18 alle 15.43.31.png" alt=""><figcaption></figcaption></figure>

Torniamo al nostro studio di caso. Lisa e il suo team di ingegneri avevano qualche esperienza con i meccanismi di comunicazione inter-processo, la loro applicazione dispone di diverse API REST utilizzate dalle applicazioni mobili e dal JavaScript lato browser. Utilizza anche vari servizi cloud, come Twilio e Stripe.&#x20;

Tuttavia, all'interno di un'applicazione monolitica come quella di UrbanEats, i moduli si invocano reciprocamente tramite chiamate di metodo o funzione a livello di linguaggio. In generale non sono state previste reali comunicazioni IPC

_Nota_: L'IPC (Inter-Process Communication) è il processo attraverso il quale diversi processi o componenti di un sistema comunicano tra loro, scambiando dati e informazioni. Questa comunicazione può avvenire in diversi modi, come richieste e risposte sincrone (come nelle chiamate API REST), oppure in modo asincrono attraverso l'invio e la ricezione di messaggi (come nei sistemi di messaggistica).

Al contrario, come abbiamo visto fino ad ora, l'architettura a microservizi struttura un'applicazione come un insieme di servizi indipendenti che devono collaborare per gestire una richiesta e visto che le istanze di servizio sono tipicamente processi in esecuzione su più macchine, devono interagire utilizzando l'IPC.&#x20;

Questo gioca un ruolo molto più importante in un'architettura a microservizi rispetto a un'applicazione monolitica. Di conseguenza, mentre migrano la loro applicazione verso una architettura a microservizi, Lisa e il resto degli sviluppatori dovranno dedicare molto più tempo a pensare all'IPC. Abbiamo diversi meccanismi IPC tra cui scegliere, oggi la tecnologia più affermata è quella REST utilizzando il formato JSON che però presenta alcuni compromessi da tenere in considerazione.

È utile iniziare pensando allo **stile di interazione** tra un servizio e i suoi client prima di selezionare un meccanismo IPC, ci aiuterà a concentrarci sui requisiti e ad evitare di rimanere bloccato nei dettagli di una particolare tecnologia IPC.Ci sono vari stili di interazione come possiamo vedere in tabella e possiamo categorizzarli in due dimensioni:&#x20;

La _prima dimensione_ riguarda l'interazione uno-a-uno o uno-a-molti:

* Uno a uno: ogni richiesta del client viene elaborata esattamente da un servizio.&#x20;
* Uno a molti: ogni richiesta viene elaborata da più servizi.

La seconda dimensione riguarda se l'interazione sia sincrona o asincrona:

* **Sincrona**: Il client si aspetta una risposta entro un range temporale dal servizio e potrebbe persino bloccarsi durante l'attesa.&#x20;
* **Asincrona**: Il client non si blocca e la risposta, se presente, non viene necessariamente inviata immediatamente

Ecco i diversi tipi di interazioni uno a uno:&#x20;

* **Richiesta/risposta sincrona:** un client del servizio invia una richiesta a un servizio e attende una risposta. Il client si aspetta che la risposta arrivi in modo tempestivo. Potrebbe anche bloccarsi durante l'attesa. Questo è uno stile di interazione che generalmente porta a una forte accoppiamento dei servizi.&#x20;
* **Richiesta/risposta asincrona:** un client del servizio invia una richiesta a un servizio, che risponde in modo asincrono. Il client non si blocca durante l'attesa, perché il servizio potrebbe non inviare la risposta per molto tempo.&#x20;
* **Messaggi monodirezionali:** un client del servizio invia una richiesta a un servizio, ma non viene attesa o inviata alcuna risposta.

È importante ricordare che lo stile di interazione _sincrona richiesta/risposta_ è principalmente indipendente dalle tecnologie IPC. Ad esempio, un servizio può interagire con un altro servizio utilizzando lo stile di interazione richiesta/risposta con REST o la messaggistica. Anche se due servizi comunicano utilizzando un message broker, il servizio client potrebbe essere bloccato in attesa di una risposta.

Ecco i diversi tipi di interazioni uno a molti:&#x20;

* **Publish/subscribe**: un client pubblica un messaggio di notifica, che viene consumato da zero o più servizi interessati.&#x20;
* **Publish/async responses**: un client pubblica un messaggio di richiesta e poi attende per un certo periodo di tempo le risposte dai servizi interessati.&#x20;

Di solito, ogni servizio utilizzerà una combinazione di questi stili di interazione. Molti dei servizi nell'applicazione UrbanEats hanno sia API sincrone che asincrone per le operazioni, e molti pubblicano anche eventi.

## SLIDE 03-01-02

<figure><img src="../.gitbook/assets/Screenshot 2023-08-21 alle 12.00.24.png" alt=""><figcaption></figcaption></figure>

Ricordiamo che un'architettura a microservizi nel suo complesso può avere una miscela di stili di collaborazione, ed è tipicamente la norma. Alcune interazioni hanno senso come richiesta-risposta, mentre altre hanno senso come event-driven. Infatti, è comune che un singolo microservizio implementi più di una forma di collaborazione. Considera ad esempio un microservizio per gli Ordini che espone un'API di richiesta-risposta per consentire di effettuare o modificare ordini, e invia eventi quando queste modifiche vengono effettuate.

Vale la pena quindi vedere qualche dettaglio in più

#### Pattern: Sincrono con Blocco&#x20;

Come già accennato poco fa, con una chiamata sincrona con blocco un microservizio invia una chiamata di qualche tipo a un processo a valle (probabilmente un altro microservizio) e si blocca finché la chiamata non è stata completata, e potenzialmente finché non è stata ricevuta una risposta perché il risultato della chiamata è necessario per un'ulteriore operazione, o semplicemente perché vuole assicurarsi che la chiamata abbia funzionato e per effettuare un tipo di riprova in caso contrario. In l'Order Processor invia una chiamata al microservizio Loyalty per informarlo che alcuni punti devono essere aggiunti all'account di un cliente.&#x20;

#### Vantaggi&#x20;

C'è qualcosa di semplice e familiare in una chiamata bloccante e sincrona. Molti di noi hanno imparato a programmare in uno stile fondamentalmente sincrono: leggendo un pezzo di codice come uno script, con ciascuna riga che viene eseguita in sequenza e la riga successiva che aspetta il suo turno per fare qualcosa. La maggior parte delle situazioni in cui avresti usato chiamate tra processi probabilmente sono state fatte in uno stile sincrono e bloccante: ad esempio eseguire una query SQL su un database o effettuare una richiesta HTTP a un'API a valle.&#x20;

#### Svantaggi

La principale sfida delle chiamate sincrone è il vincolo temporale intrinseco che si verifica,come abbiamo già visto in precedenza. Quando l'Order Processor effettua una chiamata a Loyalty nell'esempio precedente, il microservizio Loyalty deve essere raggiungibile affinché la chiamata funzioni. Se il microservizio Loyalty non è disponibile, la chiamata fallirà e l'Order Processor dovrà stabilire quale tipo di azione compensativa eseguire, che potrebbe includere un nuovo tentativo immediato, l'archiviazione temporanea della chiamata per un nuovo tentativo in seguito o addirittura l'abbandono completo.

Questo accoppiamento è **bidirezionale**. Con questo stile di integrazione, la risposta viene tipicamente inviata sulla stessa connessione di rete in ingresso al microservizio a monte. Quindi, se il microservizio Loyalty vuole inviare una risposta all'Order Processor, ma l'istanza a monte è successivamente andata in crash, la risposta andrà persa. L'accoppiamento temporale qui non riguarda solo due microservizi; riguarda due istanze specifiche di questi microservizi.

Poiché il mittente della chiamata è bloccante e in attesa che il microservizio a valle risponda, segue anche che se il microservizio a valle risponde lentamente o se c'è un problema con la latenza della rete, il mittente della chiamata sarà bloccato per un periodo prolungato in attesa di una risposta. Se il microservizio Loyalty è sotto carico significativo e risponde lentamente alle richieste, ciò a sua volta causerà un ritardo nella risposta dell'Order Processor.

Di conseguenza, l'uso di chiamate sincrone può rendere un sistema vulnerabile a problemi a cascata causati da interruzioni a valle più prontamente rispetto all'uso di chiamate asincrone.

Ora, per architetture di microservizi relativamente semplici e lineari, non abbiamo un grande problema con l'uso di chiamate sincrone e bloccanti, va considerato che la loro familiarità per molti dev è un vantaggio quando si tratta di implementare i sistemi distribuiti. Possiamo dire che questi tipi di chiamate iniziano a diventare problematici quando si inizia ad avere più chiamate concatenate.&#x20;

Per migliorare la situazione si potrebbero riesaminare le interazioni tra i microservizi in primo luogo eseguendo in background parte della catena di chiamate eseguendo parte di questo lavoro in parallelo. Riducendo la lunghezza della catena di chiamate, vedremo migliorare la latenza complessiva dell'operazione, e toglieremo uno dei nostri microservizi dal percorso critico riducendo una dipendenza in meno di cui preoccuparsi per quella che è un'operazione critica.

Ovviamente, potremmo anche sostituire l'uso di chiamate bloccanti con qualche tipo di interazione non bloccante senza cambiare il flusso di lavoro qui, un approccio che vedremo ora

## SLIDE 03-01-03

<figure><img src="../.gitbook/assets/Screenshot 2023-08-21 alle 12.09.56.png" alt=""><figcaption></figcaption></figure>

Con la comunicazione **asincrona**, l'atto di inviare una chiamata attraverso la rete non blocca il microservizio che emette la chiamata. Questo è in grado di continuare qualsiasi altra elaborazione senza dover aspettare una risposta. La comunicazione asincrona non bloccante assume molte forme ne vedremo alcune modalità più comuni:

1. **Common data**: il microservizio a monte modifica alcuni dati condivisi, che uno o più microservizi possono usare successivamente.
2. **Request-response**: Un microservizio invia una richiesta a un altro microservizio chiedendogli di fare qualcosa e quando l'operazione richiesta viene completata, con successo o fallimento, il microservizio a monte riceve la risposta.&#x20;
3. **Interazione Event-driven** Un microservizio diffonde un evento, che può essere considerato come una dichiarazione su qualcosa che è accaduto. Altri microservizi possono ascoltare gli eventi di loro interesse e reagire di conseguenza.

#### Vantaggi&#x20;

Con la comunicazione asincrona non bloccante il microservizio che effettua la chiamata iniziale e il microservizio (o i microservizi) che riceve la chiamata sono temporaneamente disaccoppiati. I microservizi che ricevono la chiamata non devono essere raggiungibili allo stesso momento in cui viene effettuata la chiamata. Ciò significa che evitiamo le preoccupazioni legate al disaccoppiamento temporale che abbiamo visto in precedenza.&#x20;

Questo stile di comunicazione è anche vantaggioso se la funzionalità invocata da una chiamata richiederà molto tempo per essere elaborata.

#### Svantaggi&#x20;

I principali svantaggi della comunicazione asincrona non bloccante, rispetto alla comunicazione sincrona bloccante, sta nel livello di complessità e di possibili scelte tecnologiche perché c'è una lista molto ampie di tecnologie che potremmo considerare.&#x20;

Alla fine, quando si considera se la comunicazione asincrona è adatta al tuo progetto, devi anche considerare quale tipo di comunicazione asincrona vogliamo scegliere, poiché ogni tipo ha i suoi compromessi. Processi a lunga durata sono un candidato ovvio e anche le situazioni in cui abbiamo lunghe catene di chiamate che non possiamo facilmente ristrutturare potrebbero essere un buon candidato.&#x20;

## SLIDE 03-01-04

<figure><img src="../.gitbook/assets/Screenshot 2023-08-21 alle 15.14.08.png" alt=""><figcaption></figcaption></figure>

Uno stile di comunicazione che abbraccia molteplici implementazioni è la **comunicazione attraverso dati condivisi**. Questo modello è utilizzato quando un microservizio inserisce dati in una posizione definita e un altro microservizio (o potenzialmente più microservizi) fa uso di tali dati. Può essere semplice come un microservizio che inserisce un file in una posizione, e in un momento successivo un altro microservizio preleva quel file e fa qualcosa con questa risorsa. Questo stile di integrazione è fondamentalmente di natura _asincrona_.

Un esempio di questo stile è mostrato nella figura nella slide in cui il "New Product Importer" crea un file che verrà poi letto dai microservizi downstream "Inventory" e "Catalog". In qualche modo, questo modello è il modello generale di comunicazione tra processi più comune che si possono incontrare

Per implementare questo modello, è necessario un tipo di archivio persistente per i dati. In molti casi, un filesystem può essere sufficiente. Ad esempio si può realizzare un sistema che semplicemente scansiona periodicamente un filesystem, rileva la presenza di un nuovo file e agisce di conseguenza o si potrebbe usare un archivio più robusto di memoria distribuita. Vale la pena notare che ogni microservizio che deve agire su questi dati avrà bisogno del proprio meccanismo per identificare che nuovi dati sono disponibili: il polling è una soluzione frequente per questo problema.

Due esempi comuni di questo modello sono il "data lake" e il "data warehouse". In entrambi i casi, queste soluzioni sono tipicamente progettate per elaborare grandi volumi di dati, ma si trovano agli opposti dello spettro in termini di accoppiamento. Con un "data lake", le fonti caricano dati grezzi nel formato che ritengono opportuno, e i consumer di questi dati grezzi devono sapere come elaborare le informazioni. Con un "data warehouse", il data warehouse stesso è un archivio di dati strutturato. I microservizi che inviano dati al data warehouse devono conoscere la struttura del data warehouse; se la struttura cambia in modo incompatibile all'indietro, questi produttori dovranno essere aggiornati.

Tanto per il "data warehouse" quanto per il "data lake", l'assunzione è che il flusso di informazioni sia unidirezionale. Un microservizio pubblica dati nello store di dati comuni e i consumatori downstream leggono quei dati e eseguono azioni appropriate. Questo flusso unidirezionale può semplificare il ragionamento sul flusso di informazioni. Un'implementazione più problematica sarebbe l'uso di un database condiviso in cui più microservizi leggono e scrivono nello stesso store dati, un esempio di cui abbiamo discusso nel Capitolo 2 quando abbiamo esplorato l'accoppiamento comune—la Figura 4-7 mostra sia il "Order Processor" che il "Warehouse" che aggiornano lo stesso record.

#### Vantaggi&#x20;

Questo modello può essere implementato in modo molto semplice, utilizzando tecnologie abbastanza comuni. Se puoi leggere o scrivere su un file o leggere e scrivere su un database, puoi utilizzare questo modello. L'uso di tecnologie ampiamente conosciute consente anche l'interoperabilità tra diversi tipi di sistemi, comprese applicazioni mainframe più vecchie o prodotti software personalizzabili. Anche i volumi di dati sono un problema relativo: se stai inviando molti dati in un'unica volta, questo modello può funzionare bene.&#x20;

#### Svantaggi&#x20;

I microservizi consumers a valle saranno tipicamente consapevoli che ci sono nuovi dati da elaborare tramite qualche meccanismo di polling, oppure forse attraverso un job temporizzato periodicamente attivato. Ovviamente puoi combinare questo modello con qualche altro tipo di chiamata che informa un microservizio a valle che sono disponibili nuovi dati. Ad esempio, potrei scrivere un file su un filesystem condiviso e quindi inviare una chiamata al microservizio interessato informandolo che ci sono nuovi dati che potrebbe voler elaborare.&#x20;

Questo può colmare il divario tra la pubblicazione dei dati e l'elaborazione dei dati. In generale, però, se stai utilizzando questo modello per volumi molto grandi di dati, è meno probabile che la bassa latenza sia in cima alla tua lista di requisiti.&#x20;

Se sei interessato a inviare volumi più grandi di dati e farli elaborare in tempo reale, allora l'uso di qualche tecnologia di streaming come Kafka sarebbe più adatto. Un altro grande svantaggio è che lo store dati comune diventa una possibile fonte di accoppiamento. Se quella memoria dati cambia struttura in qualche modo, può interrompere la comunicazione tra i microservizi. La robustezza della comunicazione dipenderà anche dalla robustezza dell'archivio dati sottostante. Questo non è uno svantaggio in senso stretto ma è qualcosa di cui essere consapevoli. Se stai depositando un file su un filesystem, potresti voler assicurarti che il filesystem stesso non vada in errore in modi interessanti.

Dove questo modello brilla davvero è nel consentire l'interoperabilità tra processi che potrebbero avere restrizioni su quale tecnologia possono utilizzare. Far comunicare un sistema esistente con l'interfaccia GRPC del tuo microservizio o farlo iscrivere al suo topic Kafka potrebbe essere più conveniente dal punto di vista del microservizio, ma non dal punto di vista del consumer. I sistemi più vecchi potrebbero avere limitazioni su quale tecnologia possono supportare e potrebbero avere costi elevati di modifica. D'altro canto, anche i sistemi mainframe più vecchi dovrebbero essere in grado di leggere dati da un file. Naturalmente, tutto ciò dipende dall'uso di tecnologie di archiviazione dati ampiamente supportate.

Un altro grande vantaggio di questo modello è nella condivisione di grandi volumi di dati. Se devi inviare un file di diverse gigabyte a un filesystem o caricare alcuni milioni di righe in un database, allora questo modello è la scelta giusta.

## SLIDE 03-01-05

<figure><img src="../.gitbook/assets/Screenshot 2023-08-21 alle 15.38.30.png" alt=""><figcaption></figcaption></figure>

Con il modello request-response, un microservizio invia una richiesta a un servizio chiedendogli di fare qualcosa e si aspetta di ricevere una risposta con il risultato della richiesta. Questa interazione può avvenire tramite una chiamata **sincrona bloccante** o potrebbe essere implementata in modo asincrono e non bloccante.&#x20;

Un esempio semplice di questa interazione è mostrato in figura, in cui il microservizio invia una richiesta al servizio Inventory per conoscere i livelli attuali di stock per alcuni CD. Recuperare dati da altri microservizi in questo modo è un caso d'uso comune per una chiamata di tipo request-response.&#x20;

Quando si lavora con interazioni di tipo _request-response_, spesso ci si trova di fronte alla situazione in cui è necessario effettuare più chiamate prima di poter proseguire con qualche elaborazione.

Consideriamo una situazione in cui MusicCorp deve verificare il prezzo di un determinato articolo presso tre diversi fornitori utilizzando alcune API. Vogliamo ottenere i prezzi da tutti e tre i fornitori prima di decidere da quale di loro vogliamo ordinare nuovo materiale.&#x20;

Potremmo decidere di effettuare le tre chiamate in sequenza, aspettando che ciascuna finisca prima di proseguire con la successiva. In una tale situazione, staremmo aspettando per la somma delle latenze di ciascuna delle chiamate. Se la chiamata API a ciascun fornitore richiedesse un secondo per restituire i dati, dovremmo attendere tre secondi prima di poter decidere da quale fornitore ordinare.

Una soluzione migliore sarebbe eseguire queste tre richieste in **parallelo**; in tal caso, la latenza complessiva dell'operazione sarebbe basata sulla chiamata API più lenta, anziché sulla somma delle latenze di ciascuna chiamata API.

Le estensioni reattive e meccanismi come **async/await** possono essere molto utili per eseguire chiamate in parallelo, e ciò può comportare miglioramenti significativi nella latenza di alcune operazioni.

## SLIDE 03-01-06

<figure><img src="../.gitbook/assets/Screenshot 2023-08-21 alle 15.44.45.png" alt=""><figcaption></figcaption></figure>

La comunicazione **basata sugli event**i appare piuttosto diversa rispetto alle chiamate di tipo request-response. Invece che un microservizio chieda a un altro microservizio di fare qualcosa, un microservizio emette eventi che possono essere ricevuti o meno da altri microservizi.&#x20;

Si tratta di un'interazione intrinsecamente asincrona, poiché i listener degli eventi verranno eseguiti nel loro proprio thread di esecuzione. Un evento è una dichiarazione su qualcosa che è avvenuto. Il microservizio che emette l'evento non ha conoscenza dell'intento degli altri microservizi di utilizzare l'evento, e potrebbe addirittura non essere consapevole dell'esistenza di altri microservizi. Questo emette l'evento quando necessario, e questo finisce le sue responsabilità.

In figura vediamo un esempio del Magazzino che emette eventi relativi al processo di imballaggio di un ordine. Questi eventi sono ricevuti da altri due microservizi: Notifiche e Inventario, che reagiscono di conseguenza. Il microservizio Notifiche invia un'email per aggiornare il cliente sulle modifiche dello stato dell'ordine, mentre il microservizio Inventario può aggiornare i livelli di magazzino man mano che gli articoli vengono imballati nell'ordine del cliente.

Il Magazzino sta semplicemente trasmettendo eventi, presumendo che le parti interessate reagiranno di conseguenza. Non è consapevole dei destinatari degli eventi, rendendo le interazioni basate sugli eventi generalmente più debolmente accoppiate. Quando si confronta questo scenario con una chiamata di tipo request-response, potrebbe volerci un po' di tempo per capire l'inversione di responsabilità. Con la richiesta-risposta, ci aspetteremmo invece che il Magazzino dica al microservizio Notifiche di inviare e-mail quando appropriato. In un modello simile, il Magazzino dovrebbe sapere quali eventi richiedono una notifica al cliente. Con un'interazione basata sugli eventi, stiamo invece spostando quella responsabilità nel microservizio Notifiche.

L'intento dietro un evento potrebbe essere considerato l'opposto di una richiesta. La fonte dell'evento lascia ai destinatari la decisione su cosa fare. Con la richiesta-risposta, il microservizio che invia la richiesta sa cosa dovrebbe essere fatto e sta dicendo all'altro microservizio ciò che pensa debba accadere successivamente. Questo naturalmente implica che nella richiesta-risposta, il richiedente deve avere conoscenza di ciò che il destinatario a valle può fare e questo implica un maggiore grado di accoppiamento di dominio

Con la interazione basata sugli eventi, l'emettitore dell'evento non ha bisogno di sapere cosa possono fare i microservizi a valle e come dicevo poco fa, in effetti, potrebbe non essere nemmeno a conoscenza della loro esistenza; di conseguenza, l'accoppiamento viene notevolmente ridotto.&#x20;

La distribuzione delle responsabilità che vediamo con le interazioni basate sugli eventi può riflettere la distribuzione delle responsabilità che vediamo nelle organizzazioni che cercano di creare team più autonomi. Piuttosto che detenere tutte le responsabilità centralmente, vogliamo delegarle verso i team stessi per consentire loro di operare in modo più autonomo, un concetto che è ricorrente quando si parla di microservizi.&#x20;

In questo esempio stiamo spostando la responsabilità dal Magazzino a Notifiche e Pagamenti, ciò può aiutarci a ridurre la complessità dei microservizi come il Magazzino.

Nota a margine: in alcune occasioni si potrebbero confondere i termini "**messaggi**" ed "**eventi**" ma un evento è un _fatto_: u_na dichiarazione che qualcosa è accaduto, insieme a alcune informazioni su cosa sia esattamente successo_. Un _messaggio_ è qualcosa che inviamo attraverso un meccanismo di comunicazione asincrona, come un message broker. Con la collaborazione basata sugli eventi, vogliamo trasmettere quell'evento, e un modo tipico per implementare tale meccanismo di trasmissione sarebbe inserire l'evento in un messaggio. Il messaggio è il mezzo, l'evento è il payload.

## SLIDE 03-01-07

Ci sono due aspetti principali da considerare per l'implementazione: un modo per far sì che i nostri microservizi emettano eventi e un modo per far sì che i consumatori vengano a conoscenza di questi eventi.&#x20;

Tradizionalmente, i message broker più comuni cercano di gestire entrambi questi problemi. I produttori utilizzano un'API per pubblicare un evento sul broker. Il broker gestisce le sottoscrizioni, consentendo ai consumatori di essere informati quando un evento arriva. Questi broker possono persino gestire lo stato dei consumers, ad esempio aiutando a tener traccia dei messaggi che hanno già visto. Questi sistemi sono progettati per essere scalabili e resilienti, ma ciò non avviene gratuitamente.

Inoltre può aggiungere complessità al processo di sviluppo, poiché è un altro sistema che potresti dover eseguire per sviluppare e testare i tuoi servizi. Potrebbe essere necessario disporre di risorse aggiuntive e competenze per mantenere questa infrastruttura. Va detto però che una volta configurata, può essere un modo incredibilmente efficace per implementare architetture _event-driven_ debolmente accoppiate ed è una prassi molto comune

Tuttavia, dobbiamo fare attenzione ai _middleware_, di cui il message broker è solo una piccola parte. Le code di per sé sono cose sensate e utili ma i fornitori tendono a voler includere molte funzionalità nei loro middleware, il che può portare all'inserimento di sempre più intelligenza nel middleware stesso.

Dovete assicurarvi di sapere cosa stai portando dentro il sistema, dovete trovare il giusto compromesso per mantenere il middleware semplice e la logica di business sugli endpoint.&#x20;

Un altro approccio è cercare di utilizzare HTTP come modo per propagare gli eventi come vedremo più avanti

#### Ma cosa contiene un Evento?

Vediamo un evento essere trasmesso dal microservizio Cliente, che informa le parti interessate che un nuovo cliente si è registrato nel sistema. Due dei microservizi, Loyalty e Notifications, sono interessati a questo evento.&#x20;

Il microservizio Loyalty reagisce alla ricezione dell'evento creando un account per il nuovo cliente in modo che possa iniziare a accumulare punti, mentre il microservizio Notifications invia una e-mail al cliente appena registrato per dare il benvenuto alle meraviglie di MusicCorp. Con una richiesta, stiamo chiedendo a un microservizio di fare qualcosa e fornendo le informazioni necessarie perché l'operazione richiesta venga eseguita. Con un evento quindi stiamo diffondendo un fatto che altre parti potrebbero essere interessate a conoscere, ma poiché il microservizio che emette un evento non può e non dovrebbe sapere chi riceve l'evento, come possiamo sapere quali informazioni potrebbero essere necessarie ad altre parti dall'evento? Cosa dovrebbe contenere esattamente l'evento?

## SLIDE 03-01-08

<figure><img src="../.gitbook/assets/Screenshot 2023-08-21 alle 16.31.59.png" alt=""><figcaption></figcaption></figure>

Un'opzione è che l'evento contenga solo un identificatore per il nuovo cliente registrato. Il microservizio Loyalty ha bisogno solo di questo identificatore per creare l'account fedeltà corrispondente, quindi ha tutte le informazioni di cui ha bisogno. Però, mentre il microservizio Notifications sa che deve inviare un'e-mail di benvenuto quando riceve questo tipo di evento, avrà bisogno di informazioni aggiuntive per svolgere il suo compito, almeno un indirizzo e-mail e probabilmente il nome del cliente per dare un tocco personale all'e-mail. Poiché queste informazioni non sono nell'evento ricevuto dal microservizio Notifica, non ha altra scelta se non recuperare queste informazioni dal microservizio Cliente.

Ci sono alcuni svantaggi in questo approccio.&#x20;

Innanzitutto, il microservizio Notifications deve ora conoscere il microservizio Cliente, aggiungendo un accoppiamento di dominio aggiuntivo anche se va ricordato che questo si trova nella parte meno stretta dello spettro di accoppiamento.

Se l'evento ricevuto dal microservizio Notifications contenesse tutte le informazioni di cui aveva bisogno, allora non sarebbe richiesta questa chiamata di callback. La chiamata di callback dal microservizio ricevente può anche portare all'altro svantaggio principale, ovvero che in una situazione con un gran numero di microservizi riceventi, il microservizio che emette l'evento potrebbe ricevere una serie di richieste come risultato. Immaginiamo se cinque diversi microservizi ricevessero tutti lo stesso evento di creazione del cliente e avessero bisogno di richiedere informazioni aggiuntive: dovrebbero tutti inviare immediatamente una richiesta al microservizio Cliente per ottenere ciò di cui avevano bisogno. Man mano che il numero di microservizi interessati a un determinato evento aumenta, l'impatto di queste chiamate potrebbe diventare significativo.

## SLIDE 03-01-09

<figure><img src="../.gitbook/assets/Screenshot 2023-08-21 alle 16.37.00.png" alt=""><figcaption></figcaption></figure>

L'alternativa è inserire tutto ciò in un evento che avremmo dovuto condividere tramite un'API. L'idea dietro è che se avessimo comunque permesso al microservizio Notifiche di richiedere l'indirizzo e-mail e il nome di un determinato cliente, perché non mettere direttamente quelle informazioni nell'evento fin dall'inizio?&#x20;

In questo schema, vediamo proprio questo approccio: il microservizio Notifiche ora è più autonomo ed è in grado di svolgere il suo lavoro senza dover comunicare con il microservizio Cliente. In effetti, potrebbe non dover mai sapere che il microservizio Cliente esiste.

Oltre al fatto che gli eventi con più informazioni possono consentire un accoppiamento più lasco, gli eventi con più informazioni possono anche fungere da registro storico di ciò che è accaduto a una determinata entità e aiutarci nell'implementazione di un sistema di audit o persino fornire la capacità di ricostruire un'entità in determinati momenti, il che significa che questi eventi potrebbero essere utilizzati come parte del concetto di _event sourcing_.&#x20;

Anche se questo approccio è sicuramente molto valido non è privo di svantaggi. In primo luogo, se i dati associati a un evento sono numerosi, potremmo avere preoccupazioni per la dimensione dell'evento. I moderni message broker hanno comunque limiti abbastanza ampi per la dimensione del singolo messaggio; Ad esempio in Kafka  la dimensione è di 1 MB fino all'ultima versione di RabbitMQ che ha un limite massimo teorico di 512 MB per un singolo messaggio.&#x20;

Nella Figura che vediamo, il servizio Loyalty non ha bisogno di conoscere l'indirizzo e-mail o il nome del cliente eppure lo riceve comunque tramite l'evento. Ciò potrebbe suscitare preoccupazioni se stiamo cercando di limitare l'ambito dei microservizi che possono accedere a quali tipi di dati, ad esempio potremmo voler limitare quali microservizi possono accedere alle informazioni personali (o PII), i dettagli delle carte di pagamento o dati sensibili simili.&#x20;

Un modo per risolvere questo problema potrebbe essere quello di inviare due diversi tipi di eventi: uno che contiene PII e può essere visualizzato da alcuni microservizi, e un altro che esclude il PII e può essere diffuso più ampiamente. Questo aggiunge complessità in termini di gestione della visibilità di eventi diversi e di garantire che entrambi gli eventi vengano effettivamente generati. Cosa succede quando un microservizio invia il primo tipo di evento ma muore prima che il secondo evento possa essere inviato?

Un'altra considerazione è che una volta che mettiamo dati in un evento, diventa parte del nostro contratto con il mondo esterno. Dobbiamo essere consapevoli che se rimuoviamo un campo da un evento, potremmo rompere le parti esterne. Nascondere le informazioni è ancora un concetto importante nella collaborazione basata sugli eventi: più dati mettiamo in un evento, più supposizioni avranno le parti esterne sull'evento.

Allontanarsi da un modello in cui dici ad altre cose cosa fare e invece permettere ai microservizi downstream di risolvere da soli questo problema ha un grande appeal in situazioni in cui ci stiamo concentrando maggiormente sul basso accoppiamento rispetto ad altri fattori, la collaborazione basata sugli eventi avrà un appeal evidente.&#x20;

La nota di cautela è che spesso emergono nuove fonti di complessità con questo stile di collaborazione ma ricordiamoci che la nostra architettura a microservizi può (e probabilmente conterrà) una combinazione di diversi stili di interazione.

## SLIDE 03-01-10

Le API o interfacce sono fondamentali nello sviluppo del software, tipicamente qualsiasi applicazione enterprise è composta da moduli, ognuno dei quali ha un'interfaccia che definisce l'insieme di operazioni che i client del modulo possono invocare.&#x20;

Un'interfaccia ben progettata espone funzionalità che vengono usate nascondendo al tempo stesso la sua implementazione, consentendo così di apportare modifiche senza influenzare i client.

In un'applicazione monolitica, un'interfaccia è generalmente specificata utilizzando una costruzione del linguaggio di programmazione, come ad esempio un'interfaccia Java che specifica un insieme di metodi che un client può invocare, nascondendo la classe di implementazione dalle altre classi. Inoltre, poiché Java è un linguaggio staticamente tipizzato, se l'interfaccia cambia in modo incompatibile con il client, l'applicazione non compilerà.

Le API e le interfacce sono altrettanto importanti in un'architettura a microservizi e le possiamo considerare come un contratto tra il servizio e i suoi client. L'API di un servizio è composta da operazioni che i client possono invocare, ed eventi, che vengono pubblicati dal servizio. Tipicamente una operazione ha un nome, parametri e un tipo di ritorno. Un evento ha un tipo e un insieme di campi ed è pubblicato su un canale di messaggi.

La sfida sta nel fatto che un'API di servizio non è definita utilizzando una semplice costruzione di linguaggio di programmazione perché per definizione, un servizio e i suoi client non vengono compilati insieme.&#x20;

Indipendentemente dal meccanismo IPC che si sceglie è importante definire precisamente l'API di un servizio utilizzando un qualche tipo di linguaggio di definizione dell'interfaccia (IDL). Inoltre è consigliabile usare un approccio API-first per la definizione dei servizi. Prima scriviamo la definizione dell'interfaccia e poi ci si confronta con gli sviluppatori dei client e successivamente si implementa il servizio. Questo design preliminare aumenta le probabilità di costruire un servizio che soddisfi le esigenze dei suoi client.

La natura della definizione dell'API dipende dal meccanismo IPC che stai utilizzando. Ad esempio, se stai utilizzando la messaggistica, l'API è composta dai canali dei messaggi, dai tipi di messaggi e dai formati dei messaggi. Se stai utilizzando HTTP, l'API è composta dagli URL, dai verbi HTTP e dai formati di richiesta e risposta.&#x20;

L'API di un servizio difficilmente rimane immutata nel tempo. Probabilmente evolverà nel tempo man mano che vengono aggiunte nuove funzionalità. In un'applicazione monolitica, è relativamente semplice modificare un'API e aggiornare tutti i servizi chiamanti soprattutto se si utilizza un linguaggio a tipizzazione statica, il compilatore aiuta fornendo un elenco di errori di compilazione ma potrebbe richiedere molto tempo modificare un'API ampiamente utilizzata.

In un'applicazione basata su microservizi, cambiare il contratto di una API di un servizio è molto più difficile perché i client di un servizio sono altri servizi, spesso sviluppati da altri team magari al di fuori dell'organizzazione e di solito non è possibile forzare tutti i client ad aggiornarsi contemporaneamente alla nuova interfaccia sopratutto se le applicazione non sono mai inattive per la manutenzione

È importante avere una strategia per affrontare queste sfide. Come gestire un cambiamento a un'API dipende dalla natura del cambiamento.

## SLIDE 03-01-11

Ci viene in aiuto la specifica di Semantic Versioning ([http://semver.org](http://semver.org/)) che è una guida utile per la versione delle API. Si tratta di un insieme di regole che specificano come i numeri di versione vengono utilizzati e incrementati. Questo versionamento era originariamente destinato alla versione dei pacchetti software, ma può essere utilizzata per versionare le API in un sistema distribuito.

La specifica richiede che un numero di versione sia composto da tre parti: **MAJOR.MINOR.PATCH**. che vanno incrementati come segue:

* _MAJOR_: quando apporti una modifica incompatibile all'API
* _MINOR_: quando apporti miglioramenti compatibili all'API
* _PATCH_: quando apporti una correzione di bug retrocompatibile

Se stiamo implementando un'API REST, possiamo utilizzare il numero di versione principale come primo elemento del percorso URL. In alternativa, se stai implementando un servizio che utilizza la messaggistica, puoi includere il numero di versione nei messaggi che pubblica. In ogni caso l'obiettivo è versionare correttamente le API e farle evolvere in modo controllato.

Idealmente, dovresti cercare di apportare solo modifiche compatibili all'indietro. Le modifiche compatibili all'indietro sono modifiche additive a un'API:

* Aggiunta di attributi opzionali alla richiesta
* Aggiunta di attributi a una risposta
* Aggiunta di nuove operazioni

Se apporti solo questo tipo di modifiche, i client più vecchi funzioneranno con i nuovi servizi, a condizione che rispettino il principio di Robustezza, cioè "Sii conservativo in ciò che fai, sii liberale in ciò che accetti dagli altri".

I servizi dovrebbero fornire valori predefiniti per gli attributi di richiesta mancanti. Allo stesso modo, i clienti dovrebbero ignorare eventuali attributi di risposta extra. Affinché ciò avvenga senza problemi, i clienti e i servizi devono utilizzare un formato di richiesta e risposta che supporti il principio di Robustezza.

Alcune volte è purtroppo necessario apportare modifiche principali e incompatibili con le vecchie API. Poiché tendenzialmente non possiamo obbligare i client a effettuare un aggiornamento immediato, un servizio deve supportare contemporaneamente le versioni vecchie e nuove di un'API per un certo periodo di tempo.&#x20;

Se stiamo utilizzando un meccanismo di IPC basato su HTTP, come REST, un approccio è incorporare il numero di versione principale nell'URL. Ad esempio, i percorsi della versione 1 sono preceduti da '/v1/...', e i percorsi della versione 2 da '/v2/...'.

Un'altra opzione di negoziazione è utilizzare il meccanismo di negoziazione dei contenuti HTTP il includere il numero di versione nel tipo MIME.&#x20;

## SLIDE 03-01-12

L'essenza della comunicazione tra processi (IPC) è lo **scambio di messaggi** che di solito contengono dati, e quindi una decisione di design importante riguarda proprio il formato di quei dati. La scelta del formato del messaggio può influenzare l'efficienza e usabilità di questo scambio. \


Se stiamo utilizzando un sistema di messaggistica o protocolli come HTTP, possiamo scegliere il formato del messaggi mentre alcuni come ad esempio gRPC, che vedremo, potrebbero imporre il formato del messaggio (e quindi potremmo avere meno libertà). In entrambi i casi, è essenziale utilizzare un formato di messaggio che sia il più cross-language possibile, anche se stai scrivendo i tuoi microservizi in un singolo linguaggio oggi, è probabile che utilizzerai altri linguaggi in futuro. Vuol dire che ad esempio non dovremmo utilizzare la serializzazione di Java.

Esistono due principali categorie di formati di messaggi: testuali e binari

**FORMATI DI MESSAGGIO BASATI SU TESTO** come JSON e XML. Un vantaggio di questi formati è che non solo sono leggibili dall'essere umano, ma sono anche abbastanza auto-descrittivi. Un messaggio JSON è una collezione di proprietà denominate e allo stesso modo un messaggio XML è essenzialmente una collezione di elementi e valori denominati. Questo formato consente ad un consumer di un messaggio di selezionare i valori di interesse e ignorare il resto. Di conseguenza, molte modifiche allo schema del messaggio possono essere facilmente compatibili all'indietro.

La struttura dei documenti XML è specificata da uno schema XML ([www.w3.org/](http://www.w3.org/) XML/Schema) e nel tempo la comunità degli sviluppatori si è resa conto che anche JSON ha bisogno di un meccanismo simile. Una delle opzioni popolari è utilizzare lo standard JSON Schema ([http://json-schema.org](http://json-schema.org/)) che definisce i nomi e i tipi delle proprietà di un messaggio e se sono opzionali o obbligatori. Oltre ad essere una documentazione utile, uno schema JSON può essere utilizzato da un'applicazione per convalidare i messaggi in ingresso.

Uno svantaggio dell'utilizzo di un formato di messaggio basato su testo è che i messaggi tendono ad essere verbosi, specialmente XML. Ogni messaggio ha il costo di contenere i nomi degli attributi oltre ai loro valori. Un altro svantaggio è il costo del parsing del testo, specialmente quando i messaggi sono grandi. Di conseguenza, se l'efficienza e le prestazioni sono importanti, potresti voler considerare l'uso di un formato binario.

**FORMATI DI MESSAGGIO BINARI** Ci sono diversi formati binari tra cui scegliere. I formati popolari includono **Protocol Buffers** ([https://developers.google.com/protocol-buffers/docs/overview](https://developers.google.com/protocol-buffers/docs/overview)) e **Avro** ([https://avro.apache.org](https://avro.apache.org/)). Entrambi i formati forniscono un IDL tipizzato per definire la struttura dei  messaggi. Un compilatore genera quindi il codice che serializza e deserializza i messaggi.&#x20;

Una differenza tra questi due formati binari è che Protocol Buffers utilizza campi con tag, mentre un consumer Avro deve conoscere lo schema per interpretare i messaggi. Di conseguenza, la gestione dell'evoluzione dell'API è più facile con Protocol Buffers che con Avro.



