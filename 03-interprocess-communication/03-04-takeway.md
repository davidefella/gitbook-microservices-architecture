# 🤝 03-04-Takeway

* L'architettura a microservizi è un'architettura distribuita, quindi la comunicazione tra processi svolge un ruolo chiave.&#x20;
* È essenziale gestire attentamente l'evoluzione dell'API di un servizio. Le modifiche compatibili all'indietro sono le più semplici da effettuare perché non influiscono sui client. Se apporti una modifica incisiva all'API di un servizio, probabilmente dovrà supportare sia le vecchie che le nuove versioni finché i client non saranno aggiornati.
* Esistono numerose tecnologie di comunicazione tra processi, ognuna con diversi compromessi. Una decisione chiave di progettazione è scegliere tra un modello di invocazione di procedure remote sincrone o il modello asincrono di messaggistica. I protocolli di invocazione sincrona di procedure remote, come REST, sono i più facili da utilizzare. Tuttavia, i servizi dovrebbero idealmente comunicare utilizzando la messaggistica asincrona per aumentare la disponibilità.&#x20;
* Per prevenire che i guasti si propaghino attraverso un sistema, un client di servizio che utilizza un protocollo sincrono deve essere progettato per gestire guasti parziali, cioè quando il servizio invocato è giù o presenta latenza elevata. In particolare, deve utilizzare timeout durante le richieste, limitare il numero di richieste in sospeso e utilizzare il modello Circuit Breaker per evitare di effettuare chiamate a un servizio in errore.&#x20;
* Un'architettura che utilizza protocolli sincroni deve includere un meccanismo di scoperta del servizio affinché i client possano determinare la posizione di rete di un'istanza del servizio. L'approccio più semplice è utilizzare il meccanismo di scoperta del servizio implementato dalla piattaforma di distribuzione: i modelli Server-side Discovery e 3rd Party Registration. Tuttavia, un approccio alternativo è implementare la scoperta del servizio a livello di applicazione: i modelli Client-side Discovery e Self Registration. È più impegnativo, ma gestisce lo scenario in cui i servizi vengono eseguiti su piattaforme di distribuzione multiple
* Un buon modo per progettare un'architettura basata sulla messaggistica è utilizzare il modello di messaggi e canali, che astrae i dettagli del sistema di messaggistica sottostante. È quindi possibile mappare tale progettazione su un'infrastruttura di messaggistica specifica, di solito basata su message broker (broker di messaggi).
* Una delle principali sfide nell'uso della messaggistica è l'aggiornamento atomico del database e la pubblicazione di un messaggio. Una buona soluzione è utilizzare il pattern Transactional Outbox e scrivere prima il messaggio nel database come parte della transazione del database. Un processo separato recupera quindi il messaggio dal database utilizzando il pattern Polling Publisher o il pattern Transaction Log Tailing e lo pubblica sul broker di messaggi.