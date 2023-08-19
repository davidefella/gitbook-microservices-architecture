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
