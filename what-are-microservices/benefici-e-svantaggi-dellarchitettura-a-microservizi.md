# Benefici e svantaggi dell'architettura a microservizi

L'architettura a microservizi presenta i seguenti vantaggi

#### Consente la consegna e il rilascio continui di applicazioni grandi e complesse.

Tra i benefici più importanti dell'architettura a microservizi è che consente la consegna e il rilascio continui di applicazioni grandi e complesse. Come vedremo più avanti, la consegna/deploy continuo fa parte di DevOps, un insieme di pratiche per la consegna rapida, frequente e affidabile del software. Le organizzazioni DevOps ad alte prestazioni di solito effettuano rilasci in produzione con pochi problemi. Ci sono tre modi in cui l'architettura a microservizi consente la consegna/il rilascio continui:

1. **Testabilità necessaria per la consegna/il rilascio continui**: I test automatizzati sono una pratica chiave della consegna/il rilascio continui. Poiché ogni servizio in un'architettura a microservizi è relativamente piccolo, i test automatizzati sono molto più facili da scrivere e più veloci da eseguire. Di conseguenza, l'applicazione avrà meno bug.
2. **La deployabilità necessaria per la consegna/il rilascio continui**: Ogni servizio può essere distribuito indipendentemente dagli altri servizi. Se gli sviluppatori responsabili di un servizio devono rilasciare una modifica specifica per quel servizio, non è necessario coordinarsi con altri sviluppatori. Possono eseguire il rilascio delle loro modifiche. Di conseguenza, è molto più facile rilasciare frequentemente le modifiche in produzione.
3. **Consente alle squadre di sviluppo di essere autonome e debolmente accoppiate**: Puoi strutturare l'organizzazione ingegneristica come una collezione di piccole squadre (ad esempio, squadre di due pizze). Ogni squadra è unicamente responsabile dello sviluppo e del rilascio di uno o più servizi correlati. Come mostra la figura 1.8, ogni squadra può sviluppare, rilasciare e scalare i propri servizi indipendentemente da tutte le altre squadre. Di conseguenza, la velocità di sviluppo è molto più elevata.\


La capacità di effettuare consegne e rilasci continui comporta diversi vantaggi per l'attività aziendale:

1. **Riduzione del tempo di commercializzazione**: Ciò consente all'azienda di reagire rapidamente ai feedback dei clienti, riducendo il tempo necessario per portare nuove funzionalità sul mercato.
2. **Offerta di un servizio affidabile**: L'azienda può fornire il tipo di servizio affidabile che i clienti odierni si aspettano, contribuendo a costruire la fiducia e la soddisfazione del cliente.
3. **Aumento della soddisfazione dei dipendenti**: I dipendenti sono più soddisfatti poiché dedicano più tempo a sviluppare funzionalità di valore anziché a gestire situazioni d'emergenza.

Di conseguenza, l'architettura a microservizi è diventata una componente fondamentale per qualsiasi azienda che dipenda dalla tecnologia del software.

#### Ogni servizio è relativamente piccolo e facile da mantenere

Un altro vantaggio dell'architettura a microservizi è che ogni servizio è relativamente piccolo. Il codice risulta più comprensibile per uno sviluppatore. La dimensione ridotta del codice non rallenta l'IDE, aumentando la produttività degli sviluppatori. Inoltre, ogni servizio di solito si avvia molto più velocemente rispetto a un grande monolite, il che migliora anche la produttività degli sviluppatori e accelera i rilasci.

<figure><img src="../.gitbook/assets/Screenshot 2023-08-14 alle 20.53.11.png" alt=""><figcaption></figcaption></figure>

#### I servizi sono scalabili in maniera indipendente

Ogni servizio in un'architettura a microservizi può essere scalato in modo indipendente dagli altri servizi utilizzando il clonaggio dell'asse X e la suddivisione dell'asse Z. Inoltre, ogni servizio può essere distribuito su hardware che si adatta meglio ai suoi requisiti di risorse. Questo è molto diverso dall'utilizzo di un'architettura monolitica, in cui componenti con requisiti di risorse molto diversi, ad esempio intensivi per la CPU rispetto a intensivi per la memoria, devono essere distribuiti insieme.

#### Migliore tolleranza ai guasti

L'architettura a microservizi offre una migliore isolamento dei guasti. Ad esempio, una perdita di memoria in un servizio influisce solo su quel servizio. Gli altri servizi continueranno a gestire le richieste normalmente. In confronto, un componente mal funzionante in un'architettura monolitica farà collassare l'intero sistema.

#### Permette di sperimentare e adottare facilmente nuove tecnologie.

Ultimo ma non meno importante, l'architettura a microservizi elimina qualsiasi lock a lungo termine verso uno stack tecnologico. In linea di principio, quando si sviluppa un nuovo servizio, gli sviluppatori sono liberi di scegliere il linguaggio e i framework più adatti per quel servizio. In molte organizzazioni ha senso limitare le scelte, ma il punto chiave è che non si è vincolati dalle decisioni del passato. Inoltre, dato che i servizi sono piccoli, riscriverli utilizzando linguaggi e tecnologie migliori diventa fattibile. Se il tentativo di una nuova tecnologia fallisce, è possibile scartare quel lavoro senza mettere a rischio l'intero progetto. Questo è molto diverso dall'utilizzo di un'architettura monolitica, dove le scelte tecnologiche iniziali limitano gravemente la capacità di utilizzare linguaggi e framework diversi in futuro.

### Svantaggi dell'architettura a microservizi

Nessuna tecnologia è una soluzione miracolosa, e l'architettura a microservizi presenta una serie di notevoli svantaggi e problematiche.&#x20;

#### Trovare il giusto insieme di servizi è sfidante.&#x20;

Una sfida nell'utilizzo dell'architettura a microservizi è che non esiste un algoritmo concreto e ben definito per decomporre un sistema in servizi. Come gran parte dello sviluppo software, è un po' un'arte. Per complicare ulteriormente le cose, se si decompone un sistema in modo errato, si costruirà un monolite distribuito, ovvero un sistema composto da servizi accoppiati che devono essere deployati insieme. Un monolite distribuito presenta gli svantaggi sia dell'architettura monolitica che dell'architettura a microservizi.

#### I sistemi distribuiti sono complessi.

Un altro problema nell'utilizzo dell'architettura a microservizi è che gli sviluppatori devono affrontare l'ulteriore complessità di creare un sistema distribuito. I servizi devono utilizzare un meccanismo di comunicazione tra processi. Questo è più complesso rispetto a una semplice chiamata di metodo. Inoltre, un servizio deve essere progettato per gestire fallimenti parziali e gestire il servizio remoto che potrebbe essere non disponibile o mostrare alta latenza.

Implementare casi d'uso che coinvolgono più servizi richiede l'uso di tecniche poco familiari. Ogni servizio ha il proprio database, il che rende difficile implementare transazioni e interrogazioni che coinvolgono diversi servizi. Come descritto nel capitolo 4, un'applicazione basata su microservizi deve utilizzare ciò che sono noti come "saga" per mantenere la coerenza dei dati tra i servizi. Il capitolo 7 spiega che un'applicazione basata su microservizi non può recuperare dati da diversi servizi tramite semplici interrogazioni. Deve invece implementare interrogazioni utilizzando la composizione di API o viste CQRS.

Gli IDE (Ambienti di Sviluppo Integrati) e altri strumenti di sviluppo sono focalizzati sulla costruzione di applicazioni monolitiche e non forniscono un supporto esplicito per lo sviluppo di applicazioni distribuite. Scrivere test automatizzati che coinvolgono più servizi è sfidante. Questi sono tutti problemi specifici dell'architettura a microservizi. Di conseguenza, gli sviluppatori della tua organizzazione devono avere competenze sofisticate di sviluppo software e di distribuzione per utilizzare con successo i microservizi.

L'architettura a microservizi introduce anche una significativa complessità operativa. Molte più componenti in movimento - molteplici istanze di diversi tipi di servizio - devono essere gestite in produzione. Per distribuire con successo i microservizi, è necessario un alto livello di automazione. Dovrai utilizzare tecnologie come le seguenti:

* Strumenti di distribuzione automatizzata, come Netflix Spinnaker
* Una piattaforma PaaS pronta all'uso, come Pivotal Cloud Foundry o Red Hat OpenShift
* Una piattaforma di orchestrazione Docker, come Docker Swarm o Kubernetes

#### Il deploy di funzionalità che coinvolgono più servizi richiede una coordinazione attenta.

Un'altra sfida nell'utilizzo dell'architettura a microservizi è che il rilascio di funzionalità che coinvolgono più servizi richiede una coordinazione attenta tra i vari team di sviluppo. È necessario creare un piano di distribuzione che ordina i rilasci dei servizi in base alle dipendenze tra di essi. Questo è molto diverso da un'architettura monolitica, in cui è possibile distribuire facilmente aggiornamenti a più componenti in modo atomico.

#### Capire quando adottarli è difficile

Un altro problema nell'utilizzo dell'architettura a microservizi è la decisione su quando durante il ciclo di vita dell'applicazione dovresti utilizzare questa architettura. Nella fase di sviluppo della prima versione di un'applicazione, spesso non si hanno i problemi che questa architettura risolve. Inoltre, l'uso di un'architettura elaborata e distribuita rallenterà lo sviluppo. Questo può essere un grande dilemma per le startup, dove il problema più grande è di solito come far evolvere rapidamente il modello di business e l'applicazione correlata. L'utilizzo dell'architettura a microservizi rende molto più difficile iterare rapidamente. Una startup dovrebbe quasi certamente iniziare con un'applicazione monolitica.

In seguito, però, quando il problema è come gestire la complessità, è a quel punto che ha senso scomporre funzionalmente l'applicazione in un insieme di microservizi. Potresti trovare difficile il refactoring a causa delle dipendenze intricate. Il capitolo 13 illustra le strategie per rifattorizzare un'applicazione monolitica in microservizi.

Come puoi vedere, l'architettura a microservizi offre molti vantaggi, ma ha anche alcuni drawback significativi. A causa di questi problemi, l'adozione di un'architettura a microservizi non dovrebbe essere presa alla leggera. Tuttavia, per applicazioni complesse, come un'applicazione web rivolta ai consumatori o un'applicazione SaaS, è di solito la scelta giusta. Siti ben noti come eBay, Amazon, Groupon sono tutti evoluti da un'architettura monolitica a un'architettura a microservizi.

Devi affrontare numerosi problemi di progettazione e architettura quando utilizzi l'architettura a microservizi. Inoltre, molti di questi problemi hanno diverse soluzioni, ognuna con un diverso insieme di compromessi. Non esiste una soluzione perfetta unica.&#x20;



