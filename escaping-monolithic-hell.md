# Escaping Monolithic Hell

Questa nostra storia inizia con Lisa, la CTO di UrbanEats, Inc. (UEI) e del suo senso di frustrazione di un lunedì mattina. Aveva trascorso la settimana precedente seguendo una bellissima conferenza, apprendendo le ultime tecniche e tecnologie per lo sviluppo software come la continous delivery o l'architettura a microservizi. La conferenza l'aveva lasciata con un senso di ottimismo e fiducia con l'ardente desiderio di migliorare il modo in cui UEI sviluppa il software.

Sfortunatamente, quella sensazione si era rapidamente dissolta. Aveva appena trascorso la prima mattinata al suo ritorno in ufficio in un altro doloroso incontro con gli ingegneri senior dell'area IT. Avevano passato metà mattina a discutere il motivo per cui il team di sviluppo avrebbe nuovamente mancato una data di rilascio critica, situazione sempre più comune negli ultimi anni.&#x20;

Nonostante l'adozione delle pratiche agili, il ritmo di sviluppo si stava rallentando, rendendo quasi impossibile raggiungere gli obiettivi aziendali. E, per peggiorare le cose, non sembrava esserci una soluzione semplice.&#x20;

La conferenza aveva fatto comprendere a Lisa che la UEI stava affrontando una sorta di "inferno monolitico" e che una possibile soluzione era adottare l'architettura a microservizi che però al suo stato attuale sembravano un sogno impossibile da raggiungere.&#x20;

Esaminiamo i problemi che UEI sta affrontando e come ci sono arrivati.

### La lenta marcia verso l'inferno monolitico

Dall'avvio alla fine del 2005, UrbanEats ha registrato una crescita esplosiva. Oggi è una delle principali aziende di consegna di cibo online negli Stati Uniti. L'azienda ha persino pianificato di espandersi all'estero, anche se questi piani sono in pericolo a causa dei ritardi nell'implementazione delle funzionalità necessarie. Alla base, l'applicazione di UrbanEats è abbastanza semplice. I consumatori utilizzano il sito web o l'applicazione mobile di UrbanEats per effettuare ordini di cibo presso ristoranti locali. UrbanEats coordina una rete di corrieri che consegnano gli ordini. È anche responsabile del pagamento dei corrieri e dei ristoranti. I ristoranti utilizzano il sito web di UrbanEats per modificare i loro menu e gestire gli ordini. L'applicazione utilizza vari servizi web, tra cui Stripe per i pagamenti, Twilio per i messaggi e Amazon Simple Email Service (SES) per le email. Come molte altre applicazioni aziendali invecchiate, l'applicazione di UrbanEats è un monolite, composto da un singolo file Java Web Application Archive (WAR). Nel corso degli anni, è diventata un'applicazione grande e complessa. Nonostante gli sforzi del team di sviluppo di UrbanEats, è diventata un esempio del pattern Big Ball of Mud ([www.laputan.org/mud/](http://www.laputan.org/mud/)). Per citare Foote e Yoder, gli autori di quel pattern, è una "struttura caotica, estesa, disordinata, fatta di nastro adesivo e filo di ferro, un groviglio di codice spaghetti." Il ritmo di consegna del software si è rallentato. Per complicare ulteriormente le cose, l'applicazione di UrbanEats è stata sviluppata utilizzando alcuni framework sempre più obsoleti. L'applicazione di UrbanEats sta manifestando tutti i sintomi dell'inferno monolitico".



