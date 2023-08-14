# Architettura a microservizi in soccorso

Lisa ha concluso che UrbanEats deve migrare all'architettura dei microservizi. Curiosamente, l'architettura del software ha ben poco a che fare con i requisiti funzionali. È possibile implementare un insieme di casi d'uso, ovvero i requisiti funzionali di un'applicazione, con qualsiasi tipo di architettura.&#x20;

In realtà, è comune che applicazioni di successo, come l'applicazione di UrbanEats, si presentino come grandi ammassi disorganizzati. L'architettura, però, è importante a causa del suo impatto sui cosiddetti requisiti di qualità del servizio, chiamati anche requisiti non funzionali, attributi di qualità.

Man mano che l'applicazione di UrbanEats è cresciuta, vari attributi di qualità hanno risentito di problemi, soprattutto quelli che influiscono sulla velocità di rilascio del software: manutenibilità, estensibilità e testabilità.&#x20;

Da una parte, un team disciplinato può rallentare la velocità del suo declino verso l'"inferno monolitico". I membri del team possono impegnarsi per mantenere la modularità dell'applicazione. Possono scrivere test automatici esaustivi.&#x20;

D'altra parte, non possono evitare i problemi legati a un grande team che lavora su un'unica applicazione monolitica. Non possono nemmeno risolvere il problema di una pila tecnologica sempre più obsoleta. Il meglio che un team possa fare è ritardare l'inevitabile.&#x20;

Per uscire dall'"inferno monolitico", devono migrare verso una nuova architettura: l'architettura a Microservizi. Oggi esiste un crescente consenso nel ritenere che, se si sta sviluppando un'applicazione grande e complessa, si dovrebbe considerare l'utilizzo dell'architettura a microservizi.&#x20;

_Ma cosa sono esattamente i microservizi?_ Purtroppo, il nome non aiuta poiché mette troppo l'accento sulla dimensione. Esistono numerose definizioni dell'architettura a microservizi. Alcune interpretazioni prendono il nome troppo letteralmente e sostengono che un servizio dovrebbe essere minuscolo, ad esempio 100 linee di codice. Altre affermano che un servizio dovrebbe richiedere solo due settimane di sviluppo e c'è chi definisce l'architettura a microservizi come un'architettura orientata ai servizi composta da elementi debolmente accoppiati con contesti delimitati. Questa non è una cattiva definizione, ma risulta un po' generica

### Scale cube e microservizi

Una delle definizioni più accettate su cosa sia un'architettura a microservizi è ispirata dal libro di Martin Abbott e Michael Fisher, "The Art of Scalability" (2015). Questo libro descrive un modello di scalabilità tridimensionale utile: il cubo della scalabilità, illustrato nella. Il modello definisce tre modi per scalare un'applicazione: X, Y e Z.

<figure><img src="../.gitbook/assets/Screenshot 2023-08-11 alle 21.45.07.png" alt=""><figcaption></figcaption></figure>



* La scalabilità sull'asse X è un modo comune per scalare un'applicazione monolitica. La figura 1.4 mostra come funziona la scalabilità sull'asse X. Si eseguono molteplici istanze dell'applicazione dietro un bilanciamento del carico. Il bilanciamento del carico distribuisce le richieste tra le N istanze identiche dell'applicazione. Questo è un ottimo modo per migliorare la capacità e la disponibilità di un'applicazione.
