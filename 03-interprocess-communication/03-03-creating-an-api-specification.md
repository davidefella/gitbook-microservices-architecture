---
description: for a messaging-based service API
---

# 03-03-Creating an API specification

## SLIDE 03-03-01

<figure><img src="../.gitbook/assets/Screenshot 2023-08-19 alle 16.51.38.png" alt=""><figcaption></figcaption></figure>

La specifica per un'API asincrona di un servizio deve, come mostrato in figura, specificare i nomi dei canali di messaggi, i tipi di messaggi scambiati su ciascun canale e i loro formati. È anche necessario descrivere il formato dei messaggi utilizzando uno standard come JSON, XML.

Tuttavia, a differenza di REST e Open API, non esiste uno standard ampiamente adottato per documentare i canali e i tipi di messaggi. Invece, è necessario scrivere un documento informale. L'API asincrona di un servizio è composta da operazioni, invocate dai client, ed eventi, pubblicati dai servizi. Sono documentati in modi diversi. Vediamoli uno per uno, iniziando dalle operazioni.

Le operazioni di un servizio possono essere invocate utilizzando uno dei due stili di interazione differenti:&#x20;

* **API di stile richiesta/risposta asincrona**: questa consiste nel canale di messaggi di comando del servizio, nei tipi e formati dei tipi di messaggi di comando che il servizio accetta e nei tipi e formati dei messaggi di risposta inviati dal servizio.
* **API di stile notifica unidirezionale:** questa consiste nel canale di messaggi di comando del servizio e nei tipi e formati dei tipi di messaggi di comando che il servizio accetta. Un servizio potrebbe utilizzare lo stesso canale di richiesta sia per la richiesta/risposta asincrona che per la notifica unidirezionale.

Un servizio può anche pubblicare eventi utilizzando uno stile di interazione _publish/subscribe_. La specifica di questo stile di API consiste nel canale degli eventi e nei tipi e formati dei messaggi di evento che il servizio pubblica sul canale. Il modello di messaggi e canali della messaggistica è un'ottima astrazione e un buon modo per progettare l'API asincrona di un servizio. Tuttavia, per implementare un servizio è necessario scegliere una tecnologia di messaggistica e determinare come implementare il design

## SLIDE 03-03-02

<figure><img src="../.gitbook/assets/Screenshot 2023-08-19 alle 17.01.36.png" alt=""><figcaption></figcaption></figure>

Un'applicazione basata sulla messaggistica di solito utilizza un message broker cioè un servizio di infrastruttura attraverso il quale il servizio comunica. Tuttavia, un'architettura basata su broker non è l'unica architettura di messaggistica possibile. Si può anche utilizzare un'architettura di messaggistica senza broker (brokerless), in cui i servizi comunicano direttamente tra loro. I due approcci, come mostrato in figura, presentano trade-off diversi, ma di solito un'architettura basata su broker è un approccio migliore che è l'approccio su cui ci concentreremo in questo corso ma comunque può essere utile dare un'occhiata veloce all'architettura senza broker, perché potrebbero esserci scenari in cui la trovi utile.

## SLIDE 03-03-03

In un'architettura senza broker, i servizi possono scambiarsi messaggi direttamente. Ad esempio ZeroMQ ([http://zeromq.org](http://zeromq.org/)) è una tecnologia di messaggistica senza broker molto popolare ed è sia una specifica che un insieme di librerie per diversi linguaggi. L'architettura senza broker ha alcuni vantaggi:

* Consente un traffico di rete più **leggero** e una _migliore latenza_, poiché i messaggi vanno direttamente dal mittente al destinatario, invece di dover passare dal mittente al message broker e da lì al destinatario.
* Elimina la possibilità che il message broker sia un **collo di bottiglia** delle prestazioni o un singolo punto di failure.
* Presenta una _minore complessità operativa_, poiché non c'è bisogno di configurare e mantenere un message broker.

Come ogni tecnologia, la messaggistica senza broker ha svantaggi anche abbastanza significativi:

* I servizi devono conoscere le _posizioni reciproche_ e devono quindi utilizzare uno dei meccanismi di discovery
* Offre una disponibilità ridotta, poiché sia il mittente che il destinatario di un messaggio devono essere disponibili durante lo scambio del messaggio.
* L'implementazione di meccanismi come la consegna garantita è più complessa da realizzare.

In effetti, alcuni di questi svantaggi, come la ridotta disponibilità e la necessità di discovery dei servizi, sono gli stessi riscontrati nell'uso di una comunicazione sincrona e a causa di queste limitazioni, la maggior parte delle applicazioni enterprise utilizza un'architettura basata su message broker come vedremo a breve.

## SLIDE 03-03-04

Un message broker è un intermediario attraverso il quale **scorrono tutti i messaggi**. Un mittente scrive il messaggio al message broker e quest'ultimo lo consegna al destinatario. Un beneficio importante nell'utilizzo di un message broker è che il mittente non ha bisogno di conoscere la posizione di rete del consumatore. Un altro vantaggio è che il message broker memorizza i messaggi finché il consumatore è in grado di elaborarli realizzando quindi una sorta di buffering. Ci sono molti message broker tra cui scegliere. Ecco alcuni esempi di popolari message broker open source:

* ActiveMQ ([http://activemq.apache.org](http://activemq.apache.org/))
* RabbitMQ ([https://www.rabbitmq.com](https://www.rabbitmq.com/))
* Apache Kafka ([http://kafka.apache.org](http://kafka.apache.org/)) Esistono anche servizi di messaggistica basati su cloud, come AWS Kinesis ([https://aws.amazon.com/kinesis/](https://aws.amazon.com/kinesis/)) e AWS SQS ([https://aws.amazon.com/sqs/](https://aws.amazon.com/sqs/)).&#x20;

Nella scelta di un message broker, ci sono diversi fattori da considerare, tra cui:

* _Linguaggi di programmazione supportati_
* _Standard di messaggistica supportati_: il message broker supporta standard open source oppure è proprietario?
* _Ordinamento_ dei messaggi
* _Garanzie_ di consegna
* _Persistenza_
* _Durabilità_: se un consumatore si riconnette al message broker, riceverà i messaggi inviati mentre era disconnesso?
* Scalabilità
* Latenza
* Consumatori in parallelo: il message broker supporta consumatori in competizione?&#x20;

Ogni broker fa scelte diverse. Ad esempio, un broker a latenza molto bassa potrebbe non preservare l'ordinamento, non garantire la consegna dei messaggi e memorizzare i messaggi solo in memoria.&#x20;

Un broker di messaggistica che garantisce la consegna e memorizza in modo affidabile i messaggi su disco probabilmente avrà una latenza più elevata. Quale tipo di message broker sia più adatto dipende dalle esigenze della tua applicazione e non è da escludere che diverse parti della tua applicazione abbiano requisiti di messaggistica diversi.&#x20;

Ci sono molti vantaggi nell'utilizzo della messaggistica basata su broker:

* **Accoppiamento basso**: un client invia una richiesta semplicemente inviando un messaggio al canale appropriato ed è completamente all'oscuro delle istanze di servizio. Non è necessario utilizzare un meccanismo di scoperta per determinare la posizione di un'istanza di servizio.
* **Buffering** dei messaggi: il message broker memorizza i messaggi finché possono essere elaborati. Con un protocollo di richiesta/risposta sincrona come HTTP, sia il client che il servizio devono essere disponibili per tutta la durata dello scambio. Con la messaggistica, invece, i messaggi si accumuleranno in coda finché non possono essere elaborati dal consumer. Ciò significa, ad esempio, che un negozio online può accettare ordini dai clienti anche quando il sistema di evasione degli ordini è lento o non disponibile. I messaggi si accumuleranno semplicemente fino a quando potranno essere elaborati.
* Comunicazione **flessibile**: la messaggistica supporta tutti gli stili di interazione descritti in fino a questo punto.

&#x20;Ci sono naturalmente anche alcuni svantaggi:

* Possibile **collo di bottiglia** delle prestazioni: c'è il rischio che il message broker possa essere un collo di bottiglia delle prestazioni anche se va detto che molti message broker moderni sono progettati per essere altamente scalabili.
* Possibile single point of failure: è essenziale che il message broker sia disponibile, altrimenti l'affidabilità del sistema ne risentirà (resilienza ai guasti)
* Complessità operativa aggiuntiva: il sistema di messaggistica è un altro componente di sistema che deve essere installato, configurato e gestito.

## SLIDE 03-03-05

<figure><img src="../.gitbook/assets/Screenshot 2023-08-20 alle 15.55.28.png" alt=""><figcaption></figcaption></figure>

Ogni message broker implementa il concetto di canale di messaggistica in modo diverso. Come mostra vediamo in tabella, i message broker JMS come ActiveMQ hanno meccanismi di code e topic.&#x20;

I message broker basati su AMQP, come RabbitMQ, hanno scambi e code. Apache Kafka ragiona per topic, AWS Kinesis ha stream e AWS SQS ha code. Quasi tutti i message broker descritti qui supportano sia canali punto-punto che canali di pubblicazione/sottoscrizione. L'unico caso eccezionale è AWS SQS, che supporta solo canali punto-punto. Ora vediamo i vantaggi e gli svantaggi della messaggistica basata su broker.

Ora vediamo alcune questioni di progettazione che potresti affrontare.

## SLIDE 03-03-06

<figure><img src="../.gitbook/assets/Screenshot 2023-08-20 alle 16.22.10.png" alt=""><figcaption></figcaption></figure>

Una sfida è rappresentata dal modo di scalare i vari receivers di messaggi preservando però l'ordine dei messaggi. È un requisito comune avere più istanze di un servizio al fine di elaborare messaggi in modo concorrente. Inoltre, anche una singola istanza del servizio probabilmente utilizzerà thread per elaborare contemporaneamente più messaggi. L'uso di thread multipli e istanze di servizio per elaborare messaggi in modo concorrente aumenta il throughput dell'applicazione. Tuttavia, la sfida nell'elaborare messaggi in modo concorrente è assicurarsi che ciascun messaggio venga elaborato una volta e nell'ordine corretto.

Ad esempio, immagina che ci siano tre istanze di un servizio che leggono dallo stesso canale point-to-point e che un mittente pubblichi messaggi di eventi "_Ordine Creato_", "_Ordine Aggiornato_" e "_Ordine Annullato_" in sequenza. Un'implementazione di messaggistica semplificata potrebbe consegnare ciascun messaggio contemporaneamente ma a riceventi diversi. A causa di ritardi dovuti a problemi di rete o garbage collection, i messaggi potrebbero essere elaborati fuori sequenza, causando comportamenti anomali. In teoria, un'istanza del servizio potrebbe elaborare il messaggio "Ordine Annullato" prima che un'altra istanza del servizio elabori il messaggio "Ordine Creato"!

Una soluzione comune, utilizzata dai moderni message broker come _Apache Kafka_ e _AWS Kinesis_, è l'uso di canali _sharded_ (partizionati). In figura vediamo come funziona. Ci sono tre parti nella soluzione:

1. Un canale **partizionato** è costituito da due o più shard, ognuno dei quali si comporta come un canale.
2. Il mittente specifica una **chiave** di shard nell'intestazione del messaggio, che è tipicamente una stringa arbitraria o una sequenza di byte. Il message broker utilizza questa chiave per assegnare il messaggio a una specifica partizione. Potrebbe, ad esempio, selezionare lo shard calcolando l'hash della chiave
3. Il message broker **raggruppa** insieme più istanze di un ricevitore e li tratta come lo stesso ricevitore logico. Ad esempio, Apache Kafka utilizza il termine "_consumer group_". Il message broker assegna ciascun shard a un singolo ricevitore. Riassegna gli shard quando i ricevitori si avviano e si spengono.

In questo esempio, ciascun messaggio di evento "_Ordine_" ha _l'orderId_ come chiave di _shard_. Ogni evento per un ordine specifico è pubblicato sullo stesso shard, che è letto da un'unica istanza del consumatore. Di conseguenza, questi messaggi sono garantiti essere elaborati in ordine.

Un'altra sfida che devi affrontare quando utilizzi la messaggistica è gestire i **messaggi duplicati.** Idealemente, un message broker dovrebbe consegnare ogni messaggio solo una volta, ma garantire la messaggistica esattamente una volta è di solito troppo costoso. Invece, la maggior parte dei message broker promette di consegnare un messaggio _almeno_ una volta.&#x20;

Quando il sistema sta funzionando normalmente, un message broker che garantisce la consegna almeno una volta consegnerà appunto ogni messaggio solo una volta. Tuttavia, un guasto di un client, della rete o del message broker può comportare la consegna multipla di un messaggio. Supponiamo che un client si blocchi dopo aver elaborato un messaggio e aggiornato il suo database, ma prima di aver confermato il messaggio. Il message broker quindi consegnerà nuovamente il messaggio non confermato.&#x20;

Idealmente, dovresti utilizzare un message broker che preservi l'ordine quando vengono consegnati nuovamente i messaggi. Immagina che il cliente elabori un evento "_Ordine creato_" seguito da un evento "_Ordine annullato_" per lo stesso ordine, e che in qualche modo l'evento "_Ordine creato_" non sia stato confermato. Il message broker dovrebbe consegnare nuovamente sia l'evento "Ordine creato" che "Ordine annullato". Se consegnasse solo l'evento "Ordine creato", il cliente potrebbe annullare l'annullamento dell'ordine. Ci sono un paio di modi diversi per gestire i messaggi duplicati:

* Scrivere gestori di messaggi idempotenti.
* Tenere traccia dei messaggi e scartare i duplicati.

Se la logica dell'applicazione che elabora i messaggi è idempotente, allora i messaggi duplicati non causano danni. La logica dell'applicazione è idempotente se chiamarla più volte con gli stessi valori in input non ha effetti aggiuntivi.&#x20;

Ad esempio, annullare un ordine già annullato è un'operazione idempotente. Lo è anche la creazione di un ordine con un ID fornito dal cliente. Un gestore di messaggi idempotente può essere eseguito in modo sicuro più volte, a condizione che il message broker preservi l'ordine quando consegna nuovamente i messaggi.&#x20;

Purtroppo, spesso la logica dell'applicazione non è idempotente oppure potremmo utilizzare un message broker che non preserva l'ordine quando consegna nuovamente i messaggi. Messaggi duplicati o fuori sequenza possono causare bug. In questa situazione, devi scrivere gestori di messaggi che tengono traccia dei messaggi e scartano i messaggi duplicati.

## SLIDE 03-03-07

<figure><img src="../.gitbook/assets/Screenshot 2023-08-20 alle 16.37.14.png" alt=""><figcaption></figcaption></figure>

Considera, ad esempio, un gestore di messaggi che autorizza una carta di credito del consumer. Deve autorizzare la carta esattamente una volta per ogni ordine. Questo esempio di logica dell'applicazione ha un effetto diverso ogni volta che viene invocato. Se i messaggi duplicati facessero sì che il gestore di messaggi eseguisse questa logica più volte, l'applicazione avrebbe un comportamento errato. Il gestore di messaggi che esegue questo tipo di logica dell'applicazione deve diventare idempotente rilevando e scartando i messaggi duplicati.

Una soluzione semplice è che un consumatore di messaggi tenga traccia dei messaggi che ha elaborato utilizzando l'ID del messaggio e scarti eventuali duplicati. Ad esempio, potrebbe archiviare l'ID del messaggio di ogni messaggio consumato in una tabella del database. Vediamo in figura come farlo con una tabella dedicata.&#x20;

Quando un consumer elabora un messaggio, _registra l'ID_ del messaggio nella tabella del database come parte della transazione che crea e aggiorna le entità aziendali. In questo esempio, il consumatore inserisce una riga contenente l'ID del messaggio in una tabella PROCESSED\_MESSAGES. Se un messaggio è un duplicato, l'INSERT fallirà e il consumatore potrà scartare il messaggio.

Un'altra opzione è che un gestore di messaggi registri gli ID dei messaggi in una tabella dell'applicazione invece di una tabella dedicata. Questo approccio è particolarmente utile quando si utilizza un database NoSQL che ha un modello di transazione più limitato, quindi non supporta l'aggiornamento di due tabelle come parte di una transazione di database.

## SLIDE 03-03-08

<figure><img src="../.gitbook/assets/Screenshot 2023-08-20 alle 17.15.22.png" alt=""><figcaption></figcaption></figure>

Spesso un servizio deve pubblicare messaggi come parte di una transazione che aggiorna il database. Sia l'aggiornamento del database che l'invio del messaggio devono avvenire all'interno di una transazione. In caso contrario, un servizio potrebbe aggiornare il database e poi bloccarsi, ad esempio, prima di inviare il messaggio. Se il servizio non esegue queste due operazioni atomicamente, un errore potrebbe lasciare il sistema in uno stato incoerente.

Una applicazione moderna deve utilizzare un meccanismo affidabile, vediamo come potrebbe funzionare questo meccanismo.

Immaginiamo che la nostra applicazione stia utilizzando un database relazionale classico. Un modo diretto per pubblicare messaggi in modo affidabile è utilizzare il pattern **Transactional Outbox**. Questo pattern utilizza una tabella del database come coda temporanea per i messaggi. Come vedete in figura un servizio che invia messaggi ha una tabella di database OUTBOX. Come parte della transazione del database che crea, aggiorna ed elimina oggetti di business, il servizio invia messaggi inserendoli nella tabella OUTBOX. L'atomicità è garantita perché si tratta di una transazione ACID locale.

La tabella OUTBOX funge da coda temporanea per i messaggi. Il MessageRelay è un componente che legge la tabella OUTBOX e pubblica i messaggi su un message broker.

Puoi utilizzare un approccio simile con alcuni database NoSQL. Ogni entità di business memorizzata come record nel database ha un attributo che è una lista di messaggi da pubblicare. Quando un service aggiorna un'entità nel database, aggiunge un messaggio a quella lista. Questo è atomico perché viene fatto con un'unica operazione di database.&#x20;

Ci sono un paio di modi diversi per spostare i messaggi dal database al message broker, vediamo quali

## SLIDE 03-03-09

Se l'applicazione utilizza un database relazionale, un modo molto semplice per pubblicare i messaggi inseriti nella tabella OUTBOX è far sì che il _MessageRelay_ interroghi periodicamente la tabella per i messaggi non pubblicati. Esso effettua interrogazioni periodiche sulla tabella:

```
SELECT * FROM OUTBOX ORDERED BY ... ASC
```

Successivamente, il _MessageRelay_ pubblica questi messaggi sul message broker, inviandone uno al canale dei messaggi di destinazione. Infine, elimina quei messaggi dalla tabella OUTBOX:

```
BEGIN
 DELETE FROM OUTBOX WHERE ID in (....)
COMMIT
```

L'interrogazione del database è un approccio semplice che funziona abbastanza bene a bassa intensità. Lo svantaggio è che interrogare frequentemente il database può essere oneroso. Inoltre, se è possibile utilizzare questo approccio con un database NoSQL dipende dalle sue capacità di interrogazione. Questo perché anziché interrogare una tabella OUTBOX, l'applicazione deve interrogare le entità di business, e potrebbe essere o meno possibile farlo in modo efficiente. A causa di questi svantaggi e limitazioni, spesso è meglio e in alcuni casi necessario utilizzare l'approccio più sofisticato e performante con i log delle transazioni del database.

## SLIDE 03-03-10

<figure><img src="../.gitbook/assets/Screenshot 2023-08-20 alle 17.32.30.png" alt=""><figcaption></figcaption></figure>

Una soluzione sofisticata è far sì che il MessageRelay segua il _log delle transazioni del database_ (detto anche log delle commit). Ogni aggiornamento effettuato da un'applicazione e confermato è rappresentato come una voce nel log delle transazioni del database. Un "**transaction log miner**" può leggere il log delle transazioni e pubblicare ogni modifica come un messaggio sul message broker, come potete vedere in figura.&#x20;

\
Questo miner legge le voci nel log delle transazioni e converte ogni voce rilevante corrispondente a un messaggio inserito in un messaggio e pubblica quel messaggio sul message broker. Questo approccio può essere utilizzato per pubblicare messaggi scritti in una tabella OUTBOX in un RDBMS o messaggi aggiunti ai record in un database NoSQL.

Anche se questo approccio è poco noto, funziona bene. La sfida sta nel fatto che la sua implementazione richiede uno molto sviluppo. Si potrebbe, ad esempio, scrivere del codice a basso livello che chiama le API specifiche del database. In alternativa, si possono utilizzare framework open source.&#x20;
