# 03-02-Sync-Remote-Procedure

## SLIDE 03-02-01

<figure><img src="../.gitbook/assets/Screenshot 2023-08-18 alle 18.07.01.png" alt=""><figcaption></figcaption></figure>

Quando si utilizza un meccanismo di IPC basato su _invocazione remota_ (Remote Procedure Invocation, RPI), un client invia una richiesta a un servizio e il servizio elabora la richiesta e invia indietro una risposta. Alcuni client possono bloccarsi in attesa di una risposta, mentre altri potrebbero avere un'architettura reattiva e non bloccante.&#x20;

Ma a differenza dell'utilizzo di messaggi, il client assume che la risposta arriverà tempestivamente. La logica di business nel client invoca un'interfaccia proxy, implementata da una classe Adapter. Il proxy RPI effettua una richiesta al servizio. La richiesta è gestita da una classe adapter che invoca la logica aziendale del servizio tramite un'interfaccia. Invia quindi una risposta al proxy RPI, che restituisce il risultato alla logica aziendale del client.

L'interfaccia proxy solitamente racchiude il protocollo di comunicazione sottostante e ce ne diversi tra cui scegliere noi vedremo REST e gRPC.

## SLIDE 03-02-02

Lo stile RESTful è tra le basi nello sviluppo moderno, REST è un meccanismo di IPC che (quasi sempre) utilizza HTTP. Possiamo dare una definizione formale di questa tecnologia:&#x20;

"REST fornisce un insieme di vincoli architetturali che, quando applicati nel loro complesso, enfatizzano la scalabilità delle interazioni tra componenti, la generalità delle interfacce, il dispiegamento indipendente dei componenti e i componenti intermedi per ridurre la latenza delle interazioni, garantire la sicurezza ed incapsulare i sistemi legacy."&#x20;

Un concetto chiave in REST è la risorsa che tipicamente rappresentano un singolo oggetto di business, come un Cliente o un Prodotto, o una collezione di oggetti. REST utilizza i verbi HTTP per manipolare le risorse, che vengono referenziate utilizzando un URL.&#x20;

Ad esempio, una richiesta GET restituisce la rappresentazione di una risorsa, spesso sotto forma di documento XML, binario o più comunemente come oggetto JSON. Una richiesta POST crea una nuova risorsa e una richiesta PUT aggiorna una risorsa. Ad esempio, il servizio Ordine ha un endpoint POST /orders per creare un Ordine e un endpoint GET /orders/{orderId} per recuperare un Ordine.

Quando si parla di sviluppo REST dobbiamo tenere a mente il concetto di Maturità REST

IL MODELLO DI MATURITÀ REST si misura nei seguenti livelli:&#x20;

* **Livello 0** - I client di un servizio di livello 0 invocano il servizio effettuando richieste HTTP POST al suo unico endpoint URL. Ogni richiesta specifica l'azione da eseguire, il target dell'azione (ad esempio, l'oggetto di business) e eventuali parametri.&#x20;
* **Livello 1** - Un servizio di livello 1 supporta l'idea di risorse. Per eseguire un'azione su una risorsa, un client effettua una richiesta POST che specifica l'azione da eseguire e eventuali parametri.&#x20;
* **Livello 2** - Un servizio di livello 2 utilizza i verbi HTTP per eseguire azioni: GET per recuperare, POST per creare e PUT per aggiornare. I parametri della query della richiesta e il corpo, se presenti, specificano i parametri delle azioni. Questo consente ai servizi di utilizzare l'infrastruttura web come la memorizzazione nella cache per le richieste GET.&#x20;
* **Livello 3** - La progettazione di un servizio di livello 3 si basa sul principio chiamato _HATEOAS_ (Hypertext As The Engine Of Application State) e l'idea di base è che la rappresentazione di una risorsa restituita da una richiesta GET contiene collegamenti per eseguire azioni su quella risorsa. Ad esempio, un client può annullare un ordine utilizzando un collegamento nella rappresentazione restituita dalla richiesta GET che ha recuperato l'ordine. I vantaggi di HATEOAS includono il fatto di non dover più cablare gli URL nel codice del client

## SLIDE 03-02-03

Come accennato in precedenza è necessario definire le nostre API utilizzando una "interface defi- nition language" (IDL) cioè un linguaggio di definizione dell'interfaccia. A differenza di protocolli come CORBA e SOAP, REST originariamente non aveva un IDL. In seguito la comunità degli sviluppatori ha riscoperto il valore aggiunto di avere a disposizione un IDL per le API RESTful. L'IDL REST più popolare è la Specifica Open API ([www.openapis.org](http://www.openapis.org/)), che è evoluta dal progetto open source Swagger. Per chi non lo conoscesse, il progetto Swagger è un insieme di strumenti per lo sviluppo e la documentazione delle API REST. Include strumenti che generano stub client e scheletri server da una definizione di interfaccia.

Le risorse REST sono di solito orientate intorno agli oggetti di business aziendali, come Consumer e Order. Di conseguenza, un problema comune durante la progettazione di un'API REST è come consentire al client di recuperare più oggetti correlati in una singola richiesta. Ad esempio, immagina che un client REST voglia recuperare un Ordine e il cliente di quell'Ordine. Di base una API REST richiederebbe al client di effettuare almeno due richieste, una per l'Ordine e un'altra per il suo Consumer. Uno scenario più complesso richiederebbe ancora più round-trip e soffrirebbe di una latenza eccessiva.

Una soluzione a questo problema è che un'API permetta al client di recuperare le risorse correlate quando ottiene una risorsa. Ad esempio, un client potrebbe recuperare un Ordine e il suo Consumer utilizzando GET /orders/order-id-1345?expand=consumer. Il parametro di query specifica le risorse correlate da restituire insieme all'Ordine. Questo approccio funziona bene in molti scenari, ma spesso è insufficiente per scenari più complessi. Inoltre, potrebbe essere potenzialmente dispendioso da implementare. Queste problematiche hanno portato all'aumentare della popolarità di tecnologie API alternative come GraphQL che è un database a grafo

Un altro problema comune nella progettazione di API REST è come mappare le operazioni che si desidera eseguire su un oggetto di business a un verbo HTTP. Un'API REST dovrebbe utilizzare PUT per le modifiche, ma potrebbero esserci molteplici modi per aggiornare un ordine, inclusa la cancellazione, la revisione dell'ordine e così via. Inoltre, una modifica potrebbe non essere idempotente, che è un requisito per l'uso di PUT.&#x20;

Una soluzione è definire una sotto-risorsa per l'aggiornamento di un aspetto specifico di una risorsa. Ad esempio, il servizio Order ha un endpoint POST /orders/{orderId}/cancel per la cancellazione degli ordini e un endpoint POST /orders/{orderId}/revise per la revisione degli ordini. Un'altra soluzione è specificare un verbo come parametro di query URL. Purtroppo, nessuna delle due soluzioni è particolarmente RESTful.

Questo problema di mappare le operazioni ai verbi HTTP ha portato alla crescente popolarità di alternative a REST, come gRPC che vedremo a breve.&#x20;

## SLIDE 03-02-04

Ci sono numerosi vantaggi nell'utilizzare REST:

* È semplice e familiare.
* Puoi testare un'API HTTP da un browser utilizzando, ad esempio, il plugin Postman, o dalla linea di comando usando curl (supponendo che venga utilizzato JSON o qualche altro formato di testo).
* Supporta direttamente la comunicazione di tipo richiesta/risposta.
* HTTP è, ovviamente, friendly con i firewall.
* Non richiede un intermediario intermedio, semplificando l'architettura del sistema.

Ci sono anche alcuni svantaggi nell'utilizzare REST:

* Supporta solo lo stile di comunicazione richiesta/risposta.
* Ridotta disponibilità. Poiché il client e il servizio comunicano direttamente senza un intermediario per bufferizzare i messaggi, entrambi devono essere in esecuzione per la durata dello scambio.
* I client devono conoscere le posizioni (URL) dell'istanza(i) del servizio e questo è potenzialmente un problema non banale in un'applicazione moderna. I client devono utilizzare quello che è noto come un meccanismo di discovery del servizio per individuare le istanze del servizio.
* Recuperare più risorse in una singola richiesta è difficile.
* A volte è difficile mappare più operazioni di aggiornamento su verbi HTTP.

## SLIDE 03-02-05

<figure><img src="../.gitbook/assets/Screenshot 2023-08-19 alle 12.20.45.png" alt=""><figcaption></figcaption></figure>

In un sistema distribuito, ogni volta che un servizio fa una richiesta sincrona a un altro servizio, c'è un rischio costante di fallimento parziale. Poiché il client e il servizio sono processi separati, un servizio potrebbe non essere in grado di rispondere tempestivamente a una richiesta, magari il servizio potrebbe essere inattivo a causa di un guasto o per manutenzione potrebbe essere sovraccarico e rispondere in modo estremamente lento alle richieste.

Poiché il cliente è bloccato in attesa di una risposta, il pericolo è che il fallimento possa propagarsi ai client del client e così via, causando un fallimento completo dell'operazione. Consideriamo ad esempio lo scenario mostrato in figura, in cui il servizio Order non risponde. Un client mobile fa una richiesta REST a un API gateway, che è il punto di ingresso dell'applicazione per i client che usano queste API. L'API gateway inoltra la richiesta al servizio Order che però non risponde.

Un'implementazione basica dell'OrderServiceProxy si bloccherebbe indefinitamente, in attesa di una risposta. Ciò non solo comporterebbe una scarsa esperienza utente, ma in molte applicazioni consumerebbe una risorsa preziosa, come un thread. Alla fine, l'API gateway esaurirebbe le risorse e non sarebbe in grado di gestire le richieste. L'intera API sarebbe irraggiungibile.

È essenziale progettare i tuoi servizi in modo da evitare che i fallimenti parziali si propaghino in tutto l'applicazione. Ci sono due parti della soluzione:

* Devi utilizzare design RPI proxies, come OrderServiceProxy, per gestire i servizi remoti non attivi.
* Devi decidere come recuperare da un servizio remoto inattivo. Prima vedremo come scrivere proxy RPI robusti.

## SLIDE 03-02-06

Quando un servizio invoca in maniera sincrona un altro servizio, dovrebbe proteggersi utilizzando l'approccio descritto dagli sviluppatori di Netflix e consiste in una combinazione dei seguenti meccanismi:

* **Timeout di rete:** non bloccarti mai indefinitamente e utilizza sempre timeout quando aspetti una risposta. Utilizzando i timeout, si garantisce che le risorse non vengano impegnate indefinitamente.
* **Limitazione del numero di richieste** pendenti da un client verso un servizio: imponi un limite superiore al numero di richieste pendenti che un cliente può fare a un servizio specifico. Se il limite è stato raggiunto, è probabilmente inutile fare ulteriori richieste e questi tentativi dovrebbero fallire immediatamente.
* Pattern del **circuit breaker**: traccia il numero di richieste riuscite e fallite e, se il tasso di errore supera una certa soglia, interrompi il circuito in modo che ulteriori tentativi falliscano immediatamente. Un gran numero di richieste fallite suggerisce che il servizio non è disponibile e che inviare ulteriori richieste sarebbe inutile.

## SLIDE 03-02-07

<figure><img src="../.gitbook/assets/Screenshot 2023-08-19 alle 13.03.03.png" alt=""><figcaption></figcaption></figure>

Utilizzare delle librerie dedicate rappresenta però solo una parte della soluzione, bisogna decidere caso per caso come i tuoi servizi dovrebbero recuperare da un servizio remoto che non risponde alle chiamate. Un'opzione semplice che va bene in alcuni scenari è che un servizio restituisca semplicemente un errore al suo client

In altri scenari invece potrebbe avere senso restituire un valore di fallback, come ad esempio un valore predefinito o una risposta memorizzata nella cache. Come vediamo in figura l'implementazione del punto finale GET /orders/{orderId} invoca diversi servizi, tra cui Order Service, Kitchen Service e DeliveryService, e combina i risultati.

Se i dati sono importanti e il servizio non è disponibile, l'API gateway dovrebbe restituire una versione memorizzata nella cache dei suoi dati o un errore. I dati degli altri servizi sono meno critici. Un client può, ad esempio, mostrare informazioni utili all'utente anche se il layer di consegna non fosse disponibile. Se il Delivery Service non è disponibile, l'API gateway dovrebbe restituire una versione memorizzata nella cache dei suoi dati o ometterli dalla risposta.

È essenziale progettare i servizi in modo da gestire i fallimenti parziali, ma questo non è l'unico problema che dobbiamo risolvere quando si utilizza una RPI. Un altro problema è che, affinché un servizio possa invocare un altro servizio questo deve conoscere la posizione di rete di un'istanza. A prima vista sembra semplice, ma nella pratica è un problema complesso. È necessario utilizzare quella che viene chiamata meccanismo di scoperta del servizio.

## SLIDE 03-02-08

<figure><img src="../.gitbook/assets/Screenshot 2023-08-19 alle 13.21.24.png" alt=""><figcaption></figcaption></figure>

Supponiamo che stiamo scrivendo del codice che invoca un servizio che ha un'API REST. Per effettuare una richiesta, il nostro codice deve conoscere la posizione di rete (indirizzo IP e porta) di un'istanza di servizio. In un'applicazione tradizionale eseguita su hardware fisico, le posizioni di rete delle istanze di servizio sono di solito statiche. Ad esempio, il tuo codice potrebbe leggere le posizioni di rete da un file di configurazione che viene occasionalmente aggiornato. Ma in un'applicazione moderna basata su microservizi in cloud, di solito non è così semplice, come possiamo vedere anche in figura.&#x20;

Le istanze di servizio hanno posizioni di rete assegnate dinamicamente. Inoltre, l'insieme di istanze di servizio cambia dinamicamente a causa del ridimensionamento automatico, dei guasti e degli aggiornamenti. Di conseguenza, il codice del tuo cliente deve utilizzare un meccanismo di scoperta del servizio.

Quindi non possiamo configurare staticamente un client con gli indirizzi IP dei services. Invece, un'applicazione deve utilizzare un meccanismo di scoperta del servizio dinamico. La scoperta del servizio è concettualmente piuttosto semplice: il suo componente chiave è un registro dei servizi, che è un database delle posizioni di rete delle istanze di servizio di un'applicazione. Il meccanismo di scoperta del servizio aggiorna il registro dei servizi quando le istanze di servizio vengono avviate e interrotte. Quando un client invoca un servizio, il meccanismo di scoperta del servizio interroga il registro dei servizi per ottenere un elenco di istanze di servizio disponibili e inoltra la richiesta a una di esse.

Esistono due modi principali per implementare la scoperta del servizio:

* I servizi e i loro client  interagiscono direttamente con il registro dei servizi.
* L'infrastruttura di distribuzione gestisce la scoperta del servizio (vedremo più avanti nel dettaglio)

Vediamo ciascuna opzione.

## SLIDE 03-02-09

<figure><img src="../.gitbook/assets/Screenshot 2023-08-19 alle 14.53.54.png" alt=""><figcaption></figcaption></figure>

Un modo per implementare la scoperta del servizio è far interagire i servizi dell'applicazione e i loro clienti con il registro dei servizi. In figura vediamo il funzionamento di questo approccio. Un'istanza del servizio registra la sua posizione di rete in questo registro e un client, prima di invocare un servizio, interroga il registro dei servizi per ottenere un elenco di istanze di servizio. Quindi successivamente invia una richiesta a una di quelle istanze.

Questo approccio alla scoperta del servizio è una combinazione di due pattern:

* **pattern di auto-registrazione (Self registration pattern):** nel pattern di auto-registrazione, un'istanza del servizio si registra presso un registro dei servizi. Ciò significa che quando un'istanza viene avviata, questa invia una richiesta di registrazione all'API del registro dei servizi, fornendo informazioni sulla sua posizione di rete (indirizzo IP e porta) e altre informazioni necessarie. Inoltre, può anche fornire un URL di controllo dello stato (health check URL), che è un endpoint API che il registro dei servizi verifica periodicamente per verificare che l'istanza di servizio sia ancora sana e in grado di rispondere alle richieste. In questo modo, il registro dei servizi può tenere traccia delle istanze di servizio attive e disponibili.
* **pattern di scoperta lato client (Client-side discovery pattern)**: nel pattern di scoperta lato client, questo interroga direttamente il registro dei servizi per ottenere un elenco delle istanze di servizio disponibili. Questo elenco contiene le posizioni di rete di tutte le istanze di servizio che sono attive e registrate. Il client può memorizzare nella cache questo elenco per migliorare le prestazioni. Quando il client vuole invocare il servizio, utilizza un algoritmo di bilanciamento del carico (come round-robin o random) per selezionare una delle istanze di servizio disponibili dalla lista. Quindi, effettua la richiesta al servizio selezionato.

Un beneficio della scoperta del servizio a livello di applicazione è la sua capacità di gestire la situazione in cui i servizi sono sparsi su più piattaforme di distribuzione mentre una possibile limitazione è la necessità di avere una libreria di discovery per ciascun linguaggio o, eventualmente, framework che utilizzi. Spring Cloud, per esempio, aiuta solo gli sviluppatori Spring. Se stai usando un altro framework Java o linguaggi come NodeJS, dovresti cercare una libreria diversa per la scoperta del servizio.&#x20;

Un'altra limitazione è che devi gestire tu stesso la configurazione e la manutenzione del registro dei servizi che potrebbe essere complesso.&#x20;

## SLIDE 03-02-10

<figure><img src="../.gitbook/assets/Screenshot 2023-08-19 alle 15.13.16.png" alt=""><figcaption></figcaption></figure>

Come vedremo più avanti, ci sono piattaforme come Docker e Kubernetes che dispongono di un registro dei servizi e un meccanismo di scoperta dei servizi integrato. La piattaforma di distribuzione assegna a ciascun servizio un _nome DNS_, un _indirizzo IP virtuale (VIP)_ e un _nome DNS_ che si risolve nell'indirizzo VIP.&#x20;

Quando un client del servizio effettua una richiesta al nome DNS/VIP, la piattaforma di distribuzione indirizza automaticamente la richiesta a una delle istanze del servizio disponibili. Di conseguenza, la registrazione dei servizi, la scoperta dei servizi e l'instradamento delle richieste sono gestiti completamente dalla piattaforma di distribuzione. Questa figura illustra come funziona questo concetto. La piattaforma di distribuzione include un registro dei servizi che tiene traccia degli indirizzi IP dei servizi distribuiti.&#x20;

In questo esempio, un client accede al servizio Ordine utilizzando il nome DNS "order-service", che corrisponde all'indirizzo IP virtuale 10.1.3.4. La piattaforma di distribuzione bilancia automaticamente le richieste tra le tre istanze del servizio Ordine.

Questo approccio è una combinazione di due modelli:

* **Modello di registrazione di terze parti**: invece che lasciare che il servizio si registri, una terza parte chiamata "registrar" (tipicamente parte della piattaforma di distribuzione) si occupa della registrazione.
* **Modello di scoperta lato server:** Invece che i client interroghino direttamente il registro dei servizi, questi effettuano una richiesta a un nome DNS, che si risolve in un instradatore delle richieste che interroga il registro e bilancia le richieste.

Il principale vantaggio della scoperta dei servizi fornita dalla piattaforma è che tutti gli aspetti della scoperta dei servizi sono interamente gestiti dalla piattaforma di distribuzione. Né i servizi né i client contengono alcun codice di scoperta dei servizi. Di conseguenza, il meccanismo di scoperta dei servizi è facilmente disponibile per tutti i servizi e i client, indipendentemente dal linguaggio di programmazione o dal framework utilizzato.

Tuttavia, un limite è che la scoperta dei servizi fornita dalla piattaforma funziona solo per i servizi distribuiti sulla stessa piattaforma. Ad esempio, come accennato in precedenza descrivendo la scoperta a livello di applicazione, la scoperta basata su Kubernetes funziona solo per i servizi in esecuzione su Kubernetes. Nonostante questo vincolo, in genere si consiglia di utilizzare la scoperta dei servizi fornita dalla piattaforma ogni volta che sia possibile.

## SLIDE 03-02-11

Quando si utilizza la messaggistica, i servizi comunicano scambiandosi messaggi in modo asincrono. Un'applicazione basata su messaggistica utilizza tipicamente un Message Broker, che agisce come intermediario tra i servizi.&#x20;

Tuttavia, un'altra opzione è utilizzare un'architettura senza broker, in cui i servizi comunicano direttamente tra loro. Un client di un servizio effettua una richiesta inviando un messaggio al servizio. Se ci si aspetta che l'istanza del servizio risponda, essa lo farà inviando un messaggio separato al client. Poiché la comunicazione è asincrona, il cliente non si blocca in attesa di una risposta. Invece, il client è scritto assumendo che la risposta non sarà ricevuta immediatamente.

Un mittente (un'applicazione o un servizio) scrive un messaggio su un canale dedicato, e un destinatario (a sua volta un'applicazione o un servizio) legge i messaggi da questo canel. Vediamo i messaggi e poi esaminiamo i canali.

Come è composto un messaggio? Un messaggio è composto da un'intestazione e un corpo del messaggio. L'intestazione è una collezione di coppie nome-valore, metadati che descrivono i dati che vengono inviati. Oltre alle coppie nome-valore fornite dal mittente del messaggio, l'intestazione del messaggio contiene coppie nome-valore, come un ID del messaggio univoco generato dal mittente o dall'infrastruttura di messaggistica, e un indirizzo di ritorno opzionale, che specifica il canale di messaggistica su cui dovrebbe essere scritta una risposta.&#x20;

Il corpo del messaggio è il dato che viene inviato, in formato testuale o binario. Esistono diversi tipi di messaggi:

* **Documento**: un messaggio generico che contiene solo dati. Il destinatario decide come interpretarlo. La risposta a un comando ne è un esempio.&#x20;
* **Comando**: un messaggio che equivale a una richiesta RPC. Specifica l'operazione da invocare e i suoi parametri.&#x20;
* **Evento**: un messaggio che indica che è accaduto qualcosa di rilevante nel mittente. Un evento è spesso un evento di dominio, che rappresenta una modifica di stato di un oggetto di dominio come un Ordine o un Cliente.

L'approccio all'architettura a microservizi che vedremo utilizza ampiamente comandi ed eventi.&#x20;

## SLIDE 03-02-12

<figure><img src="../.gitbook/assets/Screenshot 2023-08-19 alle 15.41.25.png" alt=""><figcaption></figcaption></figure>

Come mostrato in figura i messaggi vengono scambiati tramite canali. La logica di business nel mittente invoca un'interfaccia di porta di invio, che racchiude il meccanismo di comunicazione sottostante. La porta di invio è implementata da una classe Adapter specializzata nell'invio di messaggi tramite un canale dedicato.&#x20;

Un canale è un'astrazione dell'infrastruttura di messaggistica mentre la classe Adapter per la gestione dei messaggi nel servizio ricevente viene invocata per gestire il messaggio. Questa chiama un'interfaccia di ricezione implementata dalla logica di business del consumatore. Un numero qualsiasi di mittenti può inviare messaggi a un canale. Allo stesso modo, un numero qualsiasi di riceventi può ricevere messaggi da un canale.

Esistono due tipi di canali:&#x20;

* **point-to-point**: un canale point-to-point consegna un messaggio esattamente a uno dei consumatori che sta leggendo dal canale. I servizi utilizzano i canali point-to-point per gli stili di interazione uno a uno descritti in precedenza. Ad esempio, un messaggio di comando viene spesso inviato su un canale point-to-point
* **publish-subscribe:** un canale publish-subscribe consegna ogni messaggio a tutti i consumatori collegati. I servizi utilizzano i canali publish-subscribe per gli stili di interazione uno a molti descritti in precedenza. Ad esempio, un messaggio di evento viene di solito inviato su un canale publish-subscribe.

## SLIDE 03-02-13

<figure><img src="../.gitbook/assets/Screenshot 2023-08-19 alle 15.49.44.png" alt=""><figcaption></figcaption></figure>

Uno dei vantaggi preziosi della messaggistica è che è sufficientemente flessibile da supportare tutti gli stili di interazione descritti precedentemente. Alcuni stili di interazione sono implementati direttamente tramite la messaggistica e altri devono essere implementati sopra la messaggistica.&#x20;

Quando un client e un servizio interagiscono utilizzando il pattern richiesta/risposta sincrono oppure asincrono, il client invia una richiesta e il servizio invia una risposta. La differenza tra i due stili di interazione è che con la richiesta/risposta sincrona chiaramente il client si aspetta che il servizio risponda immediatamente, mentre con la richiesta/risposta asincrona non c'è tale attesa.&#x20;

La messaggistica è per natura asincrona, quindi fornisce solo la richiesta/risposta asincrona (anche se va detto che un client potrebbe bloccarsi fino a quando non riceve una risposta). Il client e il servizio comunicando scambiandosi una coppia di messaggi. Come mostrato nella figura, il client invia un messaggio di comando, che specifica l'operazione da eseguire e i parametri a un canale di messaggistica punto a punto di proprietà di un servizio. Il servizio elabora le richieste e invia un messaggio di risposta, che contiene l'esito, a un canale punto a punto di proprietà del client.

Il client deve indicare al servizio dove inviare un messaggio di risposta e deve abbinare i messaggi di risposta alle richieste. Fortunatamente, risolvere questi due problemi non è così difficile. Il client invia un messaggio di comando che ha un'intestazione di canale di risposta. Il server scrive il messaggio di risposta, che contiene un identificativo di correlazione con lo stesso valore dell'identificativo del messaggio, al canale di risposta. Il client utilizza l'identificativo di correlazione per abbinare il messaggio di risposta alla richiesta. Poiché il client e il servizio comunicano tramite messaggistica, l'interazione è per natura asincrona. In teoria, un client di messaggistica potrebbe bloccarsi fino a quando non riceve una risposta, ma nella pratica il client elaborerà le risposte in modo asincrono. Inoltre, le risposte sono tipicamente elaborate da una qualsiasi delle istanze del client.

## SLIDE 03-02-14

L'implementazione delle notifiche unidirezionali è semplice utilizzando la messaggistica asincrona. Il client invia un messaggio, tipicamente di comando, a un canale punto a punto di proprietà del servizio. Il servizio si iscrive al canale e elabora il messaggio. Non invia indietro una risposta.

PUBLISH/SUBSCRIBE La messaggistica ha un supporto integrato per lo stile di interazione publish/subscribe. Un client pubblica un messaggio su un canale publish-subscribe che viene letto da più consumatori. I servizi utilizzano il publish/subscribe per pubblicare eventi di dominio, che rappresentano modifiche agli oggetti di business. Il servizio che pubblica gli eventi di dominio possiede un canale publish-subscribe, il cui nome deriva dalla classe di dominio. Ad esempio, il servizio Ordine pubblica eventi Ordine su un canale Ordine, e il servizio Consegna pubblica eventi Consegna su un canale Consegna. Un servizio interessato agli eventi di un particolare oggetto di dominio deve solamente iscriversi al canale appropriato.

PUBLISH/ASYNC RESPONSES Lo stile di interazione publish/async responses è uno stile di interazione di livello superiore che viene implementato combinando elementi di publish/subscribe e request/response. Un client pubblica un messaggio che specifica un'intestazione di canale di risposta su un canale publish-subscribe. Un consumer scrive un messaggio di risposta contenente un identificativo di correlazione sul canale di risposta. Il client raccoglie le risposte utilizzando l'identificativo di correlazione per abbinare i messaggi di risposta alla richiesta.

Ogni servizio nella nostra applicazione che ha un'API asincrona utilizzerà una o più di queste tecniche di implementazione.
