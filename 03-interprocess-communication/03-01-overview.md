# 03-01-Overview

## SLIDE 03-01-01

<figure><img src="../.gitbook/assets/Screenshot 2023-08-18 alle 15.43.31.png" alt=""><figcaption></figcaption></figure>

Lisa e il suo team di ingegneri avevano qualche esperienza con i meccanismi di comunicazione inter-processo, la loro applicazione dispone di diverse API REST utilizzate dalle applicazioni mobili e dal JavaScript lato browser. Utilizza anche vari servizi cloud, come Twilio e Stripe.&#x20;

Tuttavia, all'interno di un'applicazione monolitica come quella di UrbanEats, i moduli si invocano reciprocamente tramite chiamate di metodo o funzione a livello di linguaggio. In generale non sono state previste reali comunicazioni IPC

_Nota_: L'IPC (Inter-Process Communication) è il processo attraverso il quale diversi processi o componenti di un sistema comunicano tra loro, scambiando dati e informazioni. Questa comunicazione può avvenire in diversi modi, come richieste e risposte sincrone (come nelle chiamate API REST), oppure in modo asincrono attraverso l'invio e la ricezione di messaggi (come nei sistemi di messaggistica).

Al contrario, come abbiamo visto fino ad ora, l'architettura a microservizi struttura un'applicazione come un insieme di servizi indipendenti che devono collaborare per gestire una richiesta e visto che le istanze di servizio sono tipicamente processi in esecuzione su più macchine, devono interagire utilizzando l'IPC.&#x20;

Questo gioca un ruolo molto più importante in un'architettura a microservizi rispetto a un'applicazione monolitica. Di conseguenza, mentre migrano la loro applicazione verso una architettura a microservizi, Lisa e il resto degli sviluppatori dovranno dedicare molto più tempo a pensare all'IPC. Abbiamo diversi meccanismi IPC tra cui scegliere, oggi la tecnologia più affermata è quella REST utilizzando il formato JSON che però presenta alcuni compromessi da tenere in considerazione.

È utile iniziare pensando allo **stile di interazione** tra un servizio e i suoi client prima di selezionare un meccanismo IPC, ci aiuterà a concentrarci sui requisiti e ad evitare di rimanere bloccato nei dettagli di una particolare tecnologia IPC.Ci sono vari stili di interazione come possiamo vedere in tabella e possiamo categorizzarli in due dimensioni:&#x20;

La _prima dimensione_ riguarda l'interazione uno-a-uno o uno-a-molti:

* Uno a uno: ogni richiesta del client viene elaborata esattamente da un servizio.&#x20;
* Uno a molti: ogni richiesta viene elaborata da più servizi.

La seconda dimensione riguarda se l'interazione sia sincrona o asincrona:

* **Sincrona**: Il client si aspetta una risposta tempestiva dal servizio e potrebbe persino bloccarsi durante l'attesa.&#x20;
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

## SLIDE 03-01-03

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

## SLIDE 03-01-04

L'essenza della comunicazione tra processi (IPC) è lo **scambio di messaggi** che di solito contengono dati, e quindi una decisione di design importante riguarda proprio il formato di quei dati. La scelta del formato del messaggio può influenzare l'efficienza e usabilità di questo scambio. \


Se stiamo utilizzando un sistema di messaggistica o protocolli come HTTP, possiamo scegliere il formato del messaggi mentre alcuni come ad esempio gRPC, che vedremo, potrebbero imporre il formato del messaggio (e quindi potremmo avere meno libertà). In entrambi i casi, è essenziale utilizzare un formato di messaggio che sia il più cross-language possibile, anche se stai scrivendo i tuoi microservizi in un singolo linguaggio oggi, è probabile che utilizzerai altri linguaggi in futuro. Vuol dire che ad esempio non dovremmo utilizzare la serializzazione di Java.

Esistono due principali categorie di formati di messaggi: testuali e binari

**FORMATI DI MESSAGGIO BASATI SU TESTO** come JSON e XML. Un vantaggio di questi formati è che non solo sono leggibili dall'essere umano, ma sono anche abbastanza auto-descrittivi. Un messaggio JSON è una collezione di proprietà denominate e allo stesso modo un messaggio XML è essenzialmente una collezione di elementi e valori denominati. Questo formato consente ad un consumer di un messaggio di selezionare i valori di interesse e ignorare il resto. Di conseguenza, molte modifiche allo schema del messaggio possono essere facilmente compatibili all'indietro.

La struttura dei documenti XML è specificata da uno schema XML ([www.w3.org/](http://www.w3.org/) XML/Schema) e nel tempo la comunità degli sviluppatori si è resa conto che anche JSON ha bisogno di un meccanismo simile. Una delle opzioni popolari è utilizzare lo standard JSON Schema ([http://json-schema.org](http://json-schema.org/)) che definisce i nomi e i tipi delle proprietà di un messaggio e se sono opzionali o obbligatori. Oltre ad essere una documentazione utile, uno schema JSON può essere utilizzato da un'applicazione per convalidare i messaggi in ingresso.

Uno svantaggio dell'utilizzo di un formato di messaggio basato su testo è che i messaggi tendono ad essere verbosi, specialmente XML. Ogni messaggio ha il costo di contenere i nomi degli attributi oltre ai loro valori. Un altro svantaggio è il costo del parsing del testo, specialmente quando i messaggi sono grandi. Di conseguenza, se l'efficienza e le prestazioni sono importanti, potresti voler considerare l'uso di un formato binario.

**FORMATI DI MESSAGGIO BINARI** Ci sono diversi formati binari tra cui scegliere. I formati popolari includono **Protocol Buffers** ([https://developers.google.com/protocol-buffers/docs/overview](https://developers.google.com/protocol-buffers/docs/overview)) e **Avro** ([https://avro.apache.org](https://avro.apache.org/)). Entrambi i formati forniscono un IDL tipizzato per definire la struttura dei  messaggi. Un compilatore genera quindi il codice che serializza e deserializza i messaggi.&#x20;

Una differenza tra questi due formati binari è che Protocol Buffers utilizza campi con tag, mentre un consumer Avro deve conoscere lo schema per interpretare i messaggi. Di conseguenza, la gestione dell'evoluzione dell'API è più facile con Protocol Buffers che con Avro.

## SLIDE 03-01-05



<figure><img src="../.gitbook/assets/Screenshot 2023-08-18 alle 18.07.01.png" alt=""><figcaption></figcaption></figure>

Quando si utilizza un meccanismo di IPC basato su _invocazione remota_ (Remote Procedure Invocation, RPI), un client invia una richiesta a un servizio e il servizio elabora la richiesta e invia indietro una risposta. Alcuni client possono bloccarsi in attesa di una risposta, mentre altri potrebbero avere un'architettura reattiva e non bloccante.&#x20;

Ma a differenza dell'utilizzo di messaggi, il client assume che la risposta arriverà tempestivamente. La logica di business nel client invoca un'interfaccia proxy, implementata da una classe Adapter. Il proxy RPI effettua una richiesta al servizio. La richiesta è gestita da una classe adapter che invoca la logica aziendale del servizio tramite un'interfaccia. Invia quindi una risposta al proxy RPI, che restituisce il risultato alla logica aziendale del client.

L'interfaccia proxy solitamente racchiude il protocollo di comunicazione sottostante e ce ne diversi tra cui scegliere noi vedremo REST e gRPC.
