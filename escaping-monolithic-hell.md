# Escaping Monolithic Hell

Questa nostra storia inizia con Lisa, la CTO di UrbanEats, Inc. (UEI) e del suo senso di frustrazione di un lunedì mattina. Aveva trascorso la settimana precedente seguendo una bellissima conferenza, apprendendo le ultime tecniche e tecnologie per lo sviluppo software come la continous delivery o l'architettura a microservizi. La conferenza l'aveva lasciata con un senso di ottimismo e fiducia con l'ardente desiderio di migliorare il modo in cui UEI sviluppa il software.

Sfortunatamente, quella sensazione si era rapidamente dissolta. Aveva appena trascorso la prima mattinata al suo ritorno in ufficio in un altro doloroso incontro con gli ingegneri senior dell'area IT. Avevano passato metà mattina a discutere il motivo per cui il team di sviluppo avrebbe nuovamente mancato una data di rilascio critica, situazione sempre più comune negli ultimi anni.&#x20;

Nonostante l'adozione delle pratiche agili, il ritmo di sviluppo si stava rallentando, rendendo quasi impossibile raggiungere gli obiettivi aziendali. E, per peggiorare le cose, non sembrava esserci una soluzione semplice.&#x20;

La conferenza aveva fatto comprendere a Lisa che la UEI stava affrontando una sorta di "inferno monolitico" e che una possibile soluzione era adottare l'architettura a microservizi che però al suo stato attuale sembravano un sogno impossibile da raggiungere.&#x20;

Esaminiamo i problemi che UEI sta affrontando e come ci sono arrivati.

### La nostra architettura

UrbanEats è un'applicazione Java aziendale tipica, in figura vediamo la sua possibile architettura. L'applicazione di UrbanEats presenta un'architettura esagonale, uno pattern architetturale che vedremo in dettaglio più avanti. In un'architettura esagonale, il nucleo dell'applicazione è costituito dalla logica di business. Attorno alla logica di business ci sono vari adattatori che implementano interfacce utente e si integrano con sistemi esterni.

<figure><img src=".gitbook/assets/Screenshot 2023-08-11 alle 21.09.45.png" alt=""><figcaption></figcaption></figure>

La logica di business è composta da moduli, ognuno dei quali è una raccolta di oggetti di dominio. Esempi di moduli includono la Gestione degli Ordini, la Gestione delle Consegne, la Fatturazione e i Pagamenti. Ci sono diversi adapter che interagiscono con i sistemi esterni. Alcuni sono adattatori in ingresso, che gestiscono le richieste invocando la logica di business, tra cui gli adattatori API REST e UI Web. Altri sono adattatori in uscita, che consentono alla logica di business di accedere al database MySQL e di invocare servizi cloud come Twilio e Paypal.

Nonostante abbia un'architettura logicamente modulare, l'applicazione di UrbanEats è strutturata come un singolo file WAR. L'applicazione è un esempio dello stile di architettura software ampiamente utilizzato, noto come **monolitico**, che struttura un sistema come un singolo componente eseguibile o distribuibile.&#x20;

Se l'applicazione di UrbanEats fosse stata scritta nel linguaggio Go (GoLang), sarebbe stato un singolo eseguibile. Una versione dell'applicazione in Ruby o NodeJS sarebbe stata una singola gerarchia di directory di codice sorgente.&#x20;

L'architettura monolitica non è intrinsecamente negativa. Gli sviluppatori di UrbanEats hanno preso una decisione valida quando hanno scelto l'architettura monolitica per la loro applicazione.

### Benefici di un architettura monolitica

Nei primi giorni di UrbanEats, quando l'applicazione era relativamente piccola, l'architettura monolitica dell'applicazione aveva molti vantaggi:

* **Semplice** da sviluppare: gli IDE e altri strumenti per sviluppatori erano focalizzati sulla creazione di un'applicazione singola.
* Facile apportare **modifiche** radicali all'applicazione: è possibile modificare il codice e lo schema del database, compilare e distribuire.
* Semplice da **testare**: gli sviluppatori hanno scritto test end-to-end che avviavano l'applicazione, invocavano l'API REST e testavano l'interfaccia utente con Selenium.
* Semplice da **distribuire**: tutto ciò che un programmatore doveva fare era copiare il file WAR su un server con Tomcat installato.
* Facile da **scalare**: UrbanEats eseguiva più istanze dell'applicazione dietro un bilanciatore di carico. Col passare del tempo, tuttavia, lo sviluppo, il testing, la distribuzione e la scalabilità sono diventati molto più complessi. Vediamo perché.

\
Purtroppo, come gli sviluppatori di UrbanEats hanno scoperto, l'architettura monolitica presenta diversi limitazioni. Applicazioni di successo come quella di UrbanEats hanno la tendenza a superare l'architettura monolitica.&#x20;

Ad ogni sprint, il team di sviluppo di UrbanEats implementava alcune features in più, il che rendeva la base di codice più ampia. Inoltre, man mano che l'azienda diventava sempre più prospera, la dimensione del team di sviluppo cresceva costantemente. Ciò non solo aumentava il tasso di crescita della base di codice, ma aumentava anche l'onere della gestione.

Come illustrato in figura, quella che un tempo era un'applicazione UrbanEats piccola e semplice si è ingrandita nel corso degli anni, trasformandosi in un mostruoso monolite. Allo stesso modo, il piccolo team di sviluppo è ora diventato composto da diverse squadre Scrum, ciascuna delle quali lavora su un'area funzionale specifica.&#x20;

A causa di una crescita che ha superato la sua architettura, UrbanEats si trova ora in un **"inferno monolitico".** Lo sviluppo è lento e doloroso. Lo sviluppo e il rilascio agili sono diventati impossibili. Vediamo perché ciò è avvenuto.

<figure><img src=".gitbook/assets/Screenshot 2023-08-11 alle 21.18.41.png" alt=""><figcaption></figcaption></figure>

Per complicare ulteriormente la situazione, questa complessità travolgente tende a essere una spirale discendente. Se la base di codice è difficile da comprendere, uno sviluppatore non apporterà correttamente le modifiche. Ogni modifica rende la base di codice incrementalmente più complessa e difficile da capire. L'architettura pulita e modulare mostrata in precedenza non rispecchia la realtà. L'azienda sta gradualmente diventando un mostruoso groviglio incomprensibile di codice.&#x20;

A questo punto Lisa ricorda di aver recentemente partecipato a una conferenza in cui ha incontrato uno sviluppatore che stava scrivendo uno strumento per analizzare le dipendenze tra le migliaia di JAR nella loro applicazione di milioni di linee di codice (_Lines Of Code_). In quel momento, quel tool sembrava qualcosa che UrbanEats avrebbe potuto utilizzare. Ora, però, non ne è così sicura. Lisa sospetta che un approccio migliore sia migrare verso un'architettura più adatta a un'applicazione complessa: i microservizi.

### Lo sviluppo è rallentato

Oltre a dover affrontare una complessità schiacciante, gli sviluppatori di UrbanEats si trovano a dover affrontare una lentezza nelle attività di sviluppo quotidiane. La grande applicazione sovraccarica e rallenta il Visual Studio Code o Eclipse di turno.&#x20;

La compilazione dell'applicazione FoodieEats richiede anche molto tempo e l'applicazione impiega molto tempo per avviarsi. Di conseguenza, il ciclo di _modifica-compilazione-esecuzione-test_ richiede molto tempo, il che incide negativamente sulla produttività.

### **Dal commit al deploy in produzione**

Un altro problema dell'applicazione FoodieEats è che implementare le modifiche in produzione è un processo lungo e doloroso. Il team di solito effettua il deployment degli aggiornamenti in produzione una volta al mese, di solito a tarda notte del venerdì o del sabato.&#x20;

Intanto Lisa continua a leggere che lo stato dell'arte per queste applicazioni è il rilascio continuo: implementare le modifiche in produzione molte volte al giorno durante l'orario lavorativo. Fino al 2011, Amazon.com implementava un cambiamento in produzione ogni 11,6 secondi senza mai influenzare l'utente! Per gli sviluppatori di FoodieEats, aggiornare la produzione più di una volta al mese sembra un sogno lontano. E adottare il rilascio continuo sembra quasi impossibile.

UrbanEats ha nel tempo adottato parzialmente l'approccio agile, il team di ingegneria è diviso in squadre e utilizza sprint di due settimane però il percorso dalla completamento del codice all'esecuzione in produzione è lungo e difficile. Un problema di avere così tanti sviluppatori che contribuiscono alla stessa base di codice è che la compilazione è spesso in uno stato non rilasciabile.&#x20;

Quando gli sviluppatori di UrbanEats hanno cercato di risolvere questo problema utilizzando branches per funzionalità, il risultato è stato un tentativo non andato a buon fine. Di conseguenza, una volta che un team completa il suo sprint, segue un lungo periodo di test e stabilizzazione del codice.&#x20;

Un'altra ragione per cui ci vuole tanto tempo per implementare le modifiche in produzione è che i test richiedono molto tempo. Poiché la base di codice è così complessa e l'impatto di una modifica non è ben compreso, gli sviluppatori e il server di Integrazione Continua (CI) devono eseguire l'intera suite di test. Alcune parti del sistema richiedono persino test manuali. Inoltre, ci vuole del tempo per diagnosticare e risolvere la causa di un fallimento dei test. Di conseguenza, ci vogliono un paio di giorni per completare un ciclo di test.

### Scalare diventa difficile

Il team di UrbanEats ha problemi di scalabilità della propria applicazione perché i diversi moduli dell'applicazione hanno requisiti di risorse contrastanti.&#x20;

Ad esempio, i dati dei ristoranti sono memorizzati in un grande database in memoria, che idealmente viene implementato su server con molta memoria. Al contrario, il modulo di elaborazione delle immagini è intensivo in termini di CPU e migliore da implementare su server con molta CPU. Poiché questi moduli fanno parte della stessa applicazione, UrbanEats deve fare compromessi sulla configurazione del server.

### Costruire un monolite affidabile è una sfida&#x20;

Un altro problema dell'applicazione di UrbanEats è la mancanza di affidabilità e conseguenza, sono frequenti gli arresti in produzione. Una delle ragioni per cui non è affidabile è che testare l'applicazione in modo approfondito è difficile a causa delle sue dimensioni considerevoli.&#x20;

Questa mancanza di testabilità fa sì che i bug arrivino in produzione. A peggiorare le cose, l'applicazione manca di isolamento dei guasti, poiché tutti i moduli vengono eseguiti nello stesso processo. Ogni tanto, un bug in un modulo - ad esempio, una perdita di memoria - fa crashare tutte le istanze dell'applicazione, una dopo l'altra.&#x20;

Nota a margine: gli sviluppatori di UrbanEats non gradiscono essere chiamati nel cuore della notte a causa di un'interruzione in produzione ma le persone dell'area business non gradiscono ancora di più la perdita di entrate e fiducia.

### Vincolo ad uno stack tecnologico sempre più obsoleto

L'aspetto finale di questo inferno monolitico è che l'architettura li costringe a utilizzare una pila tecnologica che diventa sempre più obsoleta. L'architettura monolitica rende difficile l'adozione di nuovi framework e linguaggi.&#x20;

Sarebbe estremamente costoso e rischioso riscrivere l'intera applicazione monolitica in modo che utilizzi una tecnologia nuova e presumibilmente migliore. Di conseguenza, gli sviluppatori sono vincolati alle scelte tecnologiche fatte all'inizio del progetto.&#x20;

Il framework Spring ha continuato a evolversi mantenendo la retrocompatibilità, quindi in teoria UrbanEats potrebbe aver avuto la possibilità di effettuare un aggiornamento. Purtroppo, l'applicazione di UrbanEats utilizza versioni di framework incompatibili con le versioni più recenti di Spring. Il team di sviluppo non ha mai trovato il tempo di effettuare l'aggiornamento di quei framework. Di conseguenza, parti importanti dell'applicazione sono state scritte utilizzando framework sempre più datati. Inoltre, agli sviluppatori di UrbanEats piacerebbe sperimentare con linguaggi non basati su Java come GoLang e NodeJS. Purtroppo, ciò non è possibile con un'applicazione monolitica.

### La lenta marcia verso l'inferno monolitico

Dall'avvio alla fine del 2005, UrbanEats ha registrato una crescita esplosiva. Oggi è una delle principali aziende di consegna di cibo online negli Stati Uniti. L'azienda ha persino pianificato di espandersi all'estero, anche se questi piani sono in pericolo a causa dei ritardi nell'implementazione delle funzionalità necessarie. Alla base, l'applicazione di UrbanEats è abbastanza semplice. I consumatori utilizzano il sito web o l'applicazione mobile di UrbanEats per effettuare ordini di cibo presso ristoranti locali. UrbanEats coordina una rete di corrieri che consegnano gli ordini. È anche responsabile del pagamento dei corrieri e dei ristoranti. I ristoranti utilizzano il sito web di UrbanEats per modificare i loro menu e gestire gli ordini. L'applicazione utilizza vari servizi web, tra cui Stripe per i pagamenti, Twilio per i messaggi e Amazon Simple Email Service (SES) per le email. Come molte altre applicazioni aziendali invecchiate, l'applicazione di UrbanEats è un monolite, composto da un singolo file Java Web Application Archive (WAR). Nel corso degli anni, è diventata un'applicazione grande e complessa. Nonostante gli sforzi del team di sviluppo di UrbanEats, è diventata un esempio del pattern Big Ball of Mud ([www.laputan.org/mud/](http://www.laputan.org/mud/)). Per citare Foote e Yoder, gli autori di quel pattern, è una "struttura caotica, estesa, disordinata, fatta di nastro adesivo e filo di ferro, un groviglio di codice spaghetti." Il ritmo di consegna del software si è rallentato. Per complicare ulteriormente le cose, l'applicazione di UrbanEats è stata sviluppata utilizzando alcuni framework sempre più obsoleti. L'applicazione di UrbanEats sta manifestando tutti i sintomi dell'inferno monolitico".



