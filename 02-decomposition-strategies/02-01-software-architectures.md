# 02-01-Software Architectures

Nel precedente capitolo abbiamo introdotto l'architettura a microservizi come una forma di decomposizione funzionale, in cui un'applicazione di grandi dimensioni è strutturata come una raccolta di servizi più piccoli. Sebbene questa prospettiva sia utile, solleva ulteriori domande su come l'architettura a microservizi si inserisca nel contesto più ampio dell'architettura del software, cosa costituisca un servizio e quale sia il ruolo delle dimensioni del servizio.

Cerchiamo di rispondere a queste domande facendo però prima un passo indietro per esaminare il concetto di architettura del software perché chiaramente la scelta di una architettura invece che un'altra influenza le caratteristiche della nostra applicazione

## SLIDE 02-01-01

Esistono numerose definizioni di architettura del software tra quelle più complete: "L'architettura del software di un sistema informatico è costituita dall'insieme delle strutture necessarie per comprendere il sistema. Queste strutture includono gli elementi software presenti nel sistema, le relazioni che intercorrono tra di essi e le caratteristiche che definiscono sia gli elementi che le relazioni."

Questa è chiaramente una definizione astratta ma il punto è che l'architettura di un'applicazione è la sua scomposizione in parti e relazioni tra queste parti. La scomposizione è importante per almeno un paio di ragioni:

* Agevola la divisione del lavoro e della conoscenza: consente a più sviluppatori (o a più team) con conoscenze eventualmente specializzate di lavorare insieme in modo produttivo su un'applicazione.
* Definisce come interagiscono gli elementi software.

Più concretamente, l'architettura di un'applicazione può essere analizzata da diverse prospettive allo stesso modo in cui l'architettura di un edificio può essere esaminata da punti di vista strutturali, idraulici, elettrici e altri ancora.

Generalmente an'applicazione ha almeno due macro categorie di requisiti, la prima include i requisiti **funzionali**, che definiscono cosa deve fare l'applicazione espressi in genere come casi d'uso o storie degli utenti. L'architettura ha relativamente poco a che fare con i requisiti funzionali ma invece consente consente all'applicazione di soddisfare la seconda categoria di requisiti: i suoi requisiti di qualità del servizio (o attributi di qualità) come la scalabilità e l'affidabilità. \


## SLIDE 02-01-02

<figure><img src="../.gitbook/assets/Screenshot 2023-08-17 alle 12.18.30.png" alt=""><figcaption></figcaption></figure>

Il modello 4+1 che vediamo qui nella slide definisce quattro diverse prospettive di un'architettura software. Ciascuna prospettiva descrive un aspetto specifico dell'architettura e consiste in un insieme particolare di elementi software e delle relazioni tra di essi.

Abbiamo:

* **Vista logica**: Gli elementi software creati dagli sviluppatori. Nei linguaggi orientati agli oggetti, questi elementi sono classi e pacchetti. Le relazioni tra di essi sono le relazioni tra classi e pacchetti, incluse l'ereditarietà, le associazioni e le dipendenze.
* **Vista di implementazione**: L'output del sistema di compilazione. Questa vista è costituita da moduli, che rappresentano il codice confezionato, e componenti, che sono unità eseguibili o distribuibili composte da uno o più moduli. In Java, un modulo è un file JAR, e un componente è tipicamente un file WAR o un file JAR eseguibile. Le relazioni tra di essi includono le relazioni di dipendenza tra i moduli e le relazioni di composizione tra i componenti e i moduli.
* **Vista dei processi**: I componenti in esecuzione. Ogni elemento è un processo, e le relazioni tra i processi rappresentano la comunicazione tra processi.
* **Vista di deployment**: Come i processi sono mappati sulle macchine. Gli elementi in questa vista sono costituiti da macchine fisiche o virtuali e dai processi. Le relazioni tra le macchine rappresentano la connettività di rete. Questa vista descrive anche la relazione tra i processi e le macchine.

Oltre a queste quattro viste, ci sono gli scenari, rappresentati dal "+1" nel modello 4+1. Gli scenari danno vita alle viste. Ogni scenario descrive come i vari componenti architetturali all'interno di una vista particolare collaborano per gestire una richiesta. Ad esempio, uno scenario nella vista logica mostra come le classi collaborano tra loro. Allo stesso modo, uno scenario nella vista dei processi mostra come i processi collaborano tra loro. In sostanza, gli scenari sono casi d'uso specifici che dimostrano come i componenti architetturali lavorano insieme in situazioni reali.

## SLIDE 02-01-03

Nel mondo reale l'architettura di un edificio spesso segue uno stile particolare, come ad esempio il Vittoriano, ogni stile è un insieme di decisioni di progettazione che vincolano le caratteristiche e i materiali da costruzione di un edificio.&#x20;

Questo concetto di stile architettonico si applica anche al software e ci fornisce una limitata gamma di elementi (componenti) e relazioni (connettori) con cui è possibile definire una visione dell'applicazione che utilizza tipicamente una combinazione di stili architettonici.

Il classico esempio di uno stile architettonico è quello a **strati** che organizza gli elementi in layer ognuno di questi ha un insieme ben definito di responsabilità e vincoli. Uno strato può dipendere solo dall'unico strato immediatamente sottostante o da qualsiasi strato sottostante.

L'architettura a tre livelli è tra quelle più comuni e si applica vista logica e prevede l'organizzazione delle classi dell'applicazione nei seguenti livelli o strati:&#x20;

* Strato di presentazione: contiene il codice che implementa l'interfaccia utente o le API esterne
* Strato di logica di business: contiene la logica di business
* Strato di persistenza: implementa la logica di interazione con il database

Questa organizzazione ha alcuni svantaggi importanti:

* Singolo strato di presentazione: non rappresenta il fatto che un'applicazione è probabile che venga invocata da più di un singolo sistema
* Singolo strato di persistenza: non rappresenta il fatto che un'applicazione è probabile che interagisca con più di un singolo database
* Definisce lo strato di logica di business come dipendente dallo strato di persistenza: in teoria, questa dipendenza impedisce di testare la logica di business senza il database.

## SLIDE 02-01-04

<figure><img src="../.gitbook/assets/Screenshot 2023-08-17 alle 14.31.21.png" alt=""><figcaption></figcaption></figure>

Vediamo quella che è **l'architettura esagonale** introdotta precedentemente che è una possibile alternativa allo stile architetturale a strati. Abbiamo una organizzazione della vista logica in modo da posizionare la logica di business al centro.&#x20;

Invece di uno strato di presentazione, l'applicazione ha uno o più adattatori di ingresso che gestiscono le richieste provenienti dall'esterno invocando la logica di business. Allo stesso modo, invece di uno strato di persistenza dei dati, l'applicazione ha uno o più adattatori di uscita che vengono invocati dalla logica di business e a loro volta invocano applicazioni esterne. Una caratteristica chiave e un beneficio di questa architettura è che la logica di business non dipende dagli adattatori. _Al contrario, sono gli adattatori a dipendere da essa._

La logica di business ha uno o più interfacce (o porte) che definiscono un insieme di operazioni ed è il modo in cui la logica di business interagisce con ciò che è esterno ad essa. In Java, ad esempio molto spesso si parla di interface.

_Ci sono due tipi di porte:_ di ingresso e di uscita. Una porta di ingresso è un'API esposta dalla logica di business che può essere invocata da applicazioni esterne mentre una porta di uscita è il modo in cui la logica di business invoca sistemi esterni.

Intorno alla logica di business ci sono gli **adattatori** e anche qui esistono due tipi: inbound se la richiesta viene dal mondo esterno (vedi controller MVC) e outbound che gestisce le richieste provenienti dalla logica di business.&#x20;

Un importante vantaggio dello stile architetturale esagonale è che disaccoppia la dipendenza della logica di business dalla logica di presentazione e dall'accesso ai dati negli adattatori. La logica di business non dipende né dalla logica di presentazione né dalla logica di accesso ai dati.

Un altro vantaggio è che riflette in modo più accurato l'architettura di un'applicazione moderna perché ad esempio la logica di business può essere invocata tramite più adattatori, ciascuno dei quali implementa una particolare API o interfaccia utente.&#x20;

La logica di business può a sua volta invocare più adattatori, ciascuno dei quali invoca un sistema esterno diverso. L'architettura esagonale è un ottimo modo per descrivere l'architettura di ciascun servizio in un'architettura a microservizi.

## SLIDE 02-01-05

<figure><img src="../.gitbook/assets/Screenshot 2023-08-17 alle 14.56.01.png" alt=""><figcaption></figcaption></figure>

Quindi, sia l'architettura monolitica che quella a microservizi abbiamo capito essere stili architetturali, abbiamo capito che in quella monolitica abbiamo un solo componente (un eseguibile o un WAR) mentre quella a microservizi prevede una serie di componenti multipli, che sono servizi e connettori che seguono protocolli di comunicazione con un forte vincolo imposto sul basso accoppiamento tra servizi.&#x20;

Diamo a questo punto una definizione di **servizio**, che è un componente software autonomo e indipendentemente deployable che implementa una funzionalità utile. Qui nella slide vediamo  la vista esterna di un servizio, che in questo esempio è il servizio di ordinazione.&#x20;

Un servizio ha un'API che fornisce ai suoi clienti accesso alla sua funzionalità con due tipi di operazioni: comandi ed interrogazioni. L'API è composta quindi da comandi, query ed eventi. Un comando, come createOrder(), esegue azioni e aggiorna i dati. Una query, come findOrderById(), recupera i dati. Un servizio pubblica anche eventi, come OrderCreated, che vengono consumati dai client.&#x20;

Cosa importante è che L'API di un servizio incapsula la sua implementazione interna, a differenza di un monolite, un developer non può scrivere codice che bypassa questa API che è la nostra interfaccia. Di conseguenza, l'architettura a microservizi **impone** la modularità dell'applicazione.&#x20;

Ogni servizio in un'architettura a microservizi ha la propria architettura e, potenzialmente, la propria tecnologia però tipicamente ritroviamo spesso l'architettura esagonale. La sua API è implementata da adattatori che interagiscono con la logica di business del servizio. L'adattatore delle operazioni invoca la logica di business e l'adattatore degli eventi pubblica gli eventi emessi dalla logica di business. Il componente potrebbe essere un processo autonomo, un'applicazione web o una funzione serverless nel cloud. Un requisito essenziale, tuttavia, è che un servizio abbia un'API e sia indipendentemente deployable.

Ricordiamoci anche che caratteristica fondamentale dell'architettura a microservizi è appunto che i servizi devono essere **debolmente accoppiati** e tutte le interazioni con un servizio devono avvenire tramite la sua API, che incapsula i dettagli implementativi. Come accennato in precedenza, questo requisito porta ad evitare la comunicazione tra i servizi  tramite un database. Dovete trattare i dati persistenti di un servizio come i campi di una classe e mantenerli privati. Vedremo più avanti che uno svantaggio della non condivisione dei database è che il mantenimento della coerenza dei dati e le interrogazioni tra servizi diventano più complessi.

Un altro punto da tenere in considerazione quando si parla di servizi sono le librerie condivise. Spesso come programmatori ci troviamo a raggruppare funzionalità in una librerie o moduli in modo che possano essere riutilizzate da più applicazioni senza duplicare il codice, pensate a Maven o npm. Viene naturale quindi pensare di utilizzare anche librerie condivise nell'architettura a microservizi ma bisogna stare attenti a non introdurre accidentalmente un accoppiamento tra  servizi.

Infine, questione del **termine "micro"** nella parola "microservizio", questa parola suggerisce che un servizio dovrebbe essere molto piccolo ma in realtà, la dimensione non è una metrica utile. Un obiettivo molto migliore è definire un servizio ben progettato come un servizio in grado di essere sviluppato da un piccolo team con tempi di sviluppo minimi e con collaborazione minima con altri team. In teoria, un team potrebbe essere responsabile solo di un singolo servizio, quindi quel servizio non è affatto "micro".

