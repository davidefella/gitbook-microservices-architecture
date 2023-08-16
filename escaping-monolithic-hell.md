# Escaping Monolithic Hell

### SLIDE 1

Questa nostra storia inizia con Lisa, la CTO di UrbanEats, Inc. (UEI) e del suo senso di frustrazione di un lunedì mattina come tanti. Aveva trascorso la settimana precedente seguendo una bellissima conferenza, apprendendo le ultime tecniche e tecnologie per lo sviluppo software come la Continous Delivery e l'architettura a Microservizi. La conferenza l'aveva lasciata con un senso di ottimismo e fiducia con il desiderio di migliorare il modo in cui la sua azienda sviluppa il software.

Purtroppo quella sensazione svanì molto rapidamente, aveva appena trascorso la prima mattinata al suo ritorno in ufficio in un altro doloroso incontro con gli ingegneri senior dell'area IT, avevano passato metà mattina a discutere il motivo per cui il team di sviluppo avrebbe nuovamente mancato una data di rilascio importante, situazione sempre più comune negli ultimi anni.&#x20;

Nonostante l'adozione delle pratiche agili il ritmo di sviluppo stava rallentando di molto rendendo quasi impossibile raggiungere gli obiettivi aziendali. E, per peggiorare le cose, non sembrava esserci una soluzione semplice in vista.&#x20;

La conferenza aveva fatto comprendere a Lisa che la UrbanEats stava affrontando una sorta di "inferno monolitico" e che una possibile soluzione sarebbe stata quella di adottare l'architettura a microservizi che però al suo stato attuale sembravano un obiettivo impossibile da raggiungere.&#x20;

### La nostra architettura

UrbanEats è una tipica applicazione Java enterprise e di base è abbastanza semplice: i consumatori utilizzano il sito web o l'applicazione mobile di UrbanEats per effettuare ordini presso i vari ristoranti. UrbanEats coordina una rete di corrieri che consegnano gli ordini. È anche responsabile del pagamento dei corrieri e dei ristoranti. I ristoranti utilizzano il sito web di UrbanEats per modificare i loro menu e gestire gli ordini. L'applicazione utilizza vari servizi web, tra cui Stripe per i pagamenti, Twilio per i messaggi e Amazon Simple Email Service (SES) per le email.&#x20;

Nel diagramma vediamo una sua possibile architettura e notiamo che presenta un'architettura esagonale che è un pattern architetturale. In questa architettura il nucleo dell'applicazione è costituito da quella che è la logica di business e attorno ci sono vari adapter che implementano interfacce utente che si integrano con sistemi esterni.

<figure><img src=".gitbook/assets/Screenshot 2023-08-11 alle 21.09.45.png" alt=""><figcaption></figcaption></figure>

La logica di business è composta da moduli, ognuno dei quali è una raccolta di oggetti del dominio come la Gestione degli Ordini, la Gestione delle Consegne, la Fatturazione e i Pagamenti, abbiamo anche diversi adapter che interagiscono con i sistemi esterni, alcuni sono in ingresso, che gestiscono le richieste invocando i metodi della logica di business mentre altri sono adattatori in uscita, che consentono alla logica di business di accedere al database MySQL e di invocare servizi cloud come Twilio e Paypal.

Nonostante abbia un'architettura logicamente modulare, l'applicazione di UrbanEats nei fatti è strutturata come un singolo file WAR ed è un esempio dello stile di architettura software ampiamente utilizzato, noto come **monolitico**, che struttura un sistema come un singolo componente eseguibile o deployabile.&#x20;

L'architettura monolitica non è _intrinsecamente negativa_, è stata una decisione che era valida quando hanno scelto l'architettura monolitica per la loro applicazione.

### SLIDE 2

### Benefici di un architettura monolitica

Nei primi mesi di sviluppo della nostro applicazione l'architettura monolitica dell'applicazione aveva molti vantaggi:

* **Semplicità** di sviluppo: le risorse erano focalizzate sulla creazione di un'applicazione singola più lineare.
* **Flessibilità**: era facile apportare **modifiche** anche importanti al codice o allo schema del database, compilare e distribuire.
* **Testing**: avendo una sola applicazione, gli sviluppatori sono riusciti a scrivere test end-to-end anche con un buon coverage del codice
* **Deployment**: tutto ciò che un programmatore doveva fare era copiare il file WAR su un server con Tomcat installato.
* **Scalabilità**: UrbanEats eseguiva più istanze dell'applicazione dietro un load balancer

\
Purtroppo questo tipo di architettura presenta diversi limitazioni:

Ad ogni sprint, il team di sviluppo di implementava features in più il che rendeva la base di codice più ampia. Inoltre, man mano che l'azienda diventava sempre più grande, la dimensione del team di sviluppo cresceva costantemente. Ciò non solo aumentava il tasso di crescita della base di codice, ma aumentava anche l'onere della gestione.

### SLIDE 3

<figure><img src=".gitbook/assets/Screenshot 2023-08-11 alle 21.18.41.png" alt=""><figcaption></figcaption></figure>

In questa figura vediamo che quella che un tempo era un'applicazione UrbanEats piccola e semplice si è ingrandita nel corso degli anni, trasformandosi in un monolite. Allo stesso modo, il piccolo team di sviluppo è ora diventato composto da diversi team ciascuno dei quali lavora su un'area funzionale specifica. A causa di una crescita che ha superato la sua architettura, UrbanEats si trova ora in un **"inferno monolitico"** che ha reso lo sviluppo del codice e i rilasci lenti e dolorosi

Questa complessità tende a essere una **spirale discendente** perché se la base di codice è difficile da comprendere, uno sviluppatore non apporterà correttamente o nei tempi previsti le modifiche. Ogni modifica rende la base di codice incrementalmente più complessa e difficile da capire.&#x20;

Oltre a dover affrontare una complessità schiacciante, gli sviluppatori di UrbanEats si trovano a dover affrontare una **lentezza** nelle attività di sviluppo quotidiane perché l'applicazione è sovraccarica e rallenta ad esempio il Visual Studio Code o Eclipse di turno e inoltre la compilazione dell'intera applicazione richiede anche molto tempo e l'applicazione impiega molto tempo per avviarsi.&#x20;

Quindi il ciclo di _modifica-compilazione-esecuzione-test_ richiede molto tempo, il che incide negativamente sulla produttività.

### **SLIDE 4**

Dall'avvio dei primi anni duemila UrbanEats ha registrato una crescita esplosiva diventando una delle principali aziende di consegna di cibo online negli Stati Uniti pianificando di espandersi all'estero, anche se questi piani sono diventano difficile a causa dei ritardi nell'implementazione delle funzionalità necessarie.&#x20;

Come molte altre applicazioni aziendali invecchiate nel corso degli anni, è diventata un'applicazione grande e complessa diventando un esempio del pattern Big Ball of Mud cioè una "struttura caotica, estesa, disordinata, fatta di nastro adesivo e filo di ferro, un groviglio di codice spaghetti." Il ritmo di consegna del software si è rallentato. Per complicare ulteriormente le cose, l'applicazione di UrbanEats è stata sviluppata utilizzando alcuni framework sempre più obsoleti. L'applicazione di UrbanEats sta manifestando tutti i sintomi dell'inferno monolitico.

Vediamo quelli che sono i problemi di questa applicazione

### **SLIDE 5**

Uno dei problemi più importanti dell'applicazione è che implementare le modifiche in produzione è un processo complesso, il team di solito effettua il deployment degli aggiornamenti in produzione una volta al mese e molto spesso di solito a tarda notte del venerdì o del sabato chiaramente per non interrompere il servizio dell'applicativo. &#x20;

Intanto Lisa continua a documentarsi e scopre che lo stato dell'arte per queste applicazioni è il rilascio continuo ovvero implementare le modifiche in produzione molte volte al giorno durante l'orario lavorativo. Legge infatti che fino al 2011 Amazon implementava un cambiamento in produzione ogni 11,6 secondi senza mai influenzare l'esperienza utente

UrbanEats ha nel tempo adottato parzialmente l'approccio agile e quindi il team di ingegneria è diviso in mini-team e utilizza sprint di due settimane però uno dei problemi dell'avere così tanti sviluppatori che contribuiscono alla stessa base di codice è che la compilazione è spesso in uno stato **non consistente**.&#x20;

Quando gli sviluppatori di UrbanEats hanno cercato di risolvere questo problema utilizzando branches per funzionalità, il risultato è stato un tentativo che non è riuscito a risolvere il problema. Di conseguenza, una volta che un team completa il suo sprint, segue un lungo periodo di test e stabilizzazione del codice.&#x20;

I test richiedono molte risorse perché la base di codice è complessa e l'impatto di una modifica spesso non è ben compreso. Gli sviluppatori devono eseguire l'intera suite di test ad ogni rilascio e alcune parti del sistema richiedono persino test manual ed inoltre ci vuole del tempo per diagnosticare e risolvere la causa di un eventuale fallimento dei test. Spesso si arriva ad impiegare anche un paio di giorni per completare un ciclo di test.

Il team di UrbanEats ha problemi di **scalabilità** della propria applicazione perché i diversi moduli dell'applicazione hanno requisiti di risorse contrastanti.&#x20;

Ad esempio, i dati dei ristoranti sono memorizzati in un grande database in memoria che idealmente viene implementato su server con molta memoria mentre al contrario il modulo di elaborazione delle immagini è intensivo in termini di CPU e migliore da implementare su server con molta CPU. Poiché questi moduli fanno parte della stessa applicazione, UrbanEats deve fare compromessi sulla configurazione del server.

Un altro problema dell'applicazione di UrbanEats è la mancanza di affidabilità sono frequenti gli arresti in produzione perché testare l'applicazione in modo approfondito è difficile a causa delle sue dimensioni considerevoli e quindi spesso i bug arrivino in produzione. A peggiorare le cose, l'applicazione manca di isolamento dei guasti, poiché tutti i moduli vengono eseguiti nello stesso processo.&#x20;

Nota a margine: gli sviluppatori di UrbanEats non gradiscono essere chiamati nel cuore della notte a causa di un'interruzione in produzione ma le persone dell'area business non gradiscono ancora di più la perdita di entrate e fiducia.

Questa architettura costringe il team a utilizzare una pila tecnologica che diventa via via sempre più **obsoleta**. L'architettura monolitica rende difficile l'adozione di nuovi framework e linguaggi. Sarebbe estremamente costoso e rischioso riscrivere l'intera applicazione monolitica in modo che utilizzi una tecnologia nuova e presumibilmente migliore. Di conseguenza, gli sviluppatori sono vincolati alle scelte tecnologiche fatte all'inizio del progetto.&#x20;

Il team di sviluppo non ha mai trovato le risorse per effettuare l'aggiornamento dei vari framework come Spring. Di conseguenza, parti importanti dell'applicazione sono state scritte utilizzando framework sempre più datati.&#x20;

Inoltre, agli sviluppatori di UrbanEats piacerebbe sperimentare con linguaggi non basati su Java come GoLang e NodeJS. Purtroppo, ciò non è possibile con un'applicazione monolitica.

###



