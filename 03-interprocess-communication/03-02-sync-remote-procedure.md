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
