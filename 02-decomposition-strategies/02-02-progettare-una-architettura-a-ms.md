# 02-02-Progettare una Architettura a MS

## SLIDE 02-02-01

<figure><img src="../.gitbook/assets/Screenshot 2023-08-17 alle 15.16.36.png" alt=""><figcaption></figcaption></figure>

Come dovremmo progettare un'architettura a microservizi? Come con ogni sforzo di sviluppo software, i punti di partenza sono i requisiti scritti, idealmente gli esperti del dominio e nel caso un'applicazione esistente. Come gran parte dello sviluppo software, la definizione di un'architettura è più arte che scienza e non è un processo che si può seguire meccanicamente, è un processo iterativo che coinvolgerà anche un po' di creatività ed esperienza

Un'applicazione tendenzialmente esiste per gestire richieste degli utenti, quindi uno dei primi passi è estrarre i requisiti chiave dell'applicazione dalle richieste usando concetti più astratti su operazione di sistema cioè richieste che l'applicazione deve gestire. È o un comando, che aggiorna i dati, o una query, che recupera dati.

## SLIDE 02-02-02

<figure><img src="../.gitbook/assets/Screenshot 2023-08-17 alle 15.21.35.png" alt=""><figcaption></figcaption></figure>

Il secondo passo nel processo è determinare la suddivisione in servizi adottando le varie strategie a disposizione, una di queste consiste nel definire servizi corrispondenti alle capacità aziendali. Un'altra strategia è organizzare i servizi attorno a sottodomini ma comunque risultato finale sono servizi organizzati intorno a concetti aziendali piuttosto che concetti tecnici.

## SLIDE 02-02-03

<figure><img src="../.gitbook/assets/Screenshot 2023-08-17 alle 15.24.16.png" alt=""><figcaption></figcaption></figure>

Il terzo passo nella definizione dell'architettura dell'applicazione è determinare l'API di ciascun servizio, assegniamo a ciascuna operazione di sistema identificata nel primo passo un servizio che potrebbe implementare un'operazione completamente in autonomia.&#x20;

In alternativa, potrebbe essere necessario collaborare con altri servizi e in questo caso stabiliamo come i servizi devono collaborare, il che richiede tipicamente che i servizi supportino operazioni aggiuntive. Dobbiamo anche decidere quale dei meccanismi di comunicazione tra processi implementare per l'API di ciascun servizio.&#x20;

Ci sono diversi ostacoli alla suddivisione del dominio in microservizi:&#x20;

* Latenza di rete: potremmo ad esempio scoprire che una particolare suddivisione sarebbe impraticabile a causa di troppi viaggi di andata e ritorno tra i servizi.
* Comunicazione sincrona: questo potrebbe ridurre la disponibilità tra i vari servizi
* Coerenza tra i servizi e i dati&#x20;
* Gestione della classi utilizzate in tutto l'applicazione anche se queste si possono eliminare applicando design orientato al dominio (God classes)

## SLIDE 02-02-04

<figure><img src="../.gitbook/assets/Screenshot 2023-08-17 alle 16.00.50 (1).png" alt=""><figcaption></figcaption></figure>

Come già accennato, il primo passo nella definizione dell'architettura di un'applicazione è definire le operazioni di sistema usando come punto di partenza i requisiti dell'applicazione, inclusi gli user story e i relativi scenari utente associati

Il primo step crea il modello di dominio ad alto livello composto dalle classi chiave che forniscono un vocabolario con cui descrivere le operazioni di sistema e il secondo passo identifica le operazioni di sistema e ne descrive il comportamento in termini di modello di dominio che è principalmente derivato dai sostantivi degli user story, mentre le operazioni di sistema sono in gran parte derivate dai verbi.&#x20;

Il comportamento di ogni operazione di sistema è descritto in termini del suo effetto su uno o più oggetti di dominio e sulle relazioni tra di loro. Un'operazione di sistema può creare, aggiornare o eliminare oggetti di dominio, oltre a creare o distruggere relazioni tra di essi.

## SLIDE 02-02-05

Il primo passo nel processo di definizione delle operazioni di sistema è creare un modello di dominio ad alto livello per l'applicazione, che di base è molto più semplice di quello che verrà implementato alla fine.&#x20;

L'applicazione non avrà nemmeno un singolo modello di dominio perché ogni servizio ha il suo. Nonostante sia una semplificazione, un modello ad alto livello è utile in questa fase perché definisce il vocabolario per descrivere il comportamento delle operazioni di sistema. Una delle tecniche è l'analisi dei sostantivi nelle storie e negli scenari e la consulenza degli esperti di dominio.&#x20;

Consideriamo ad esempio la storia "Effettua ordine". Possiamo espandere questa storia in numerosi scenari utente, incluso questo:&#x20;

{% code fullWidth="true" %}
```
Dato un cliente
    E un ristorante
    E un indirizzo/orario di consegna che può essere servito da quel ristorante
    E un totale ordine che soddisfa il requisito minimo d'ordine del ristorante
Quando il cliente effettua un ordine per il ristorante
Allora la carta di credito del cliente viene autorizzata
    E un ordine viene creato nello stato di "PENDING_ACCEPTANCE" (in attesa di accettazione)
    E l'ordine è associato al cliente
    E l'ordine è associato al ristorante
```
{% endcode %}

I sostantivi in questo scenario utente suggeriscono l'esistenza di varie classi, tra cui **Consumatore**, **Ordine**, **Ristorante** e **Carta di Credito**. Allo stesso modo, la storia di Accettazione dell'Ordine può essere espansa in uno scenario come questo

{% code fullWidth="true" %}
```
Dato un ordine nello stato di PENDING_ACCEPTANCE
    e un corriere disponibile per consegnare l'ordine
Quando un ristorante accetta un ordine con la previsione di prepararlo entro un determinato orario
Allora lo stato dell'ordine viene modificato in ACCETTATO
    E il tempo previsto dell'ordine viene aggiornato
    E il corriere viene ingaggiato per consegnare l'ordine
```
{% endcode %}

## SLIDE 02-02-06

<figure><img src="../.gitbook/assets/Screenshot 2023-08-17 alle 16.26.24.png" alt=""><figcaption></figcaption></figure>

Questo scenario suggerisce l'esistenza delle classi _Courier_ e _Delivery_. Il risultato finale dopo alcune iterazioni di analisi sarà un modello di dominio che comprende queste classi e altre, come MenuItem e Address. In figura vediamo un diagramma delle classi che mostra le classi chiave. Le responsabilità di ciascuna classe sono le seguenti:

* **Consumer**: un cliente che effettua ordini.
* **Order**: un ordine effettuato da un cliente. Descrive l'ordine e ne tiene traccia dello stato.
* **OrderLineItem**: una riga dell'ordine.
* **DeliveryInfo**: l'orario e il luogo di consegna di un ordine.
* **Restaurant**: un ristorante che prepara ordini per la consegna ai consumatori.
* **MenuItem**: un elemento nel menu del ristorante.
* **Courier**: un corriere che consegna ordini ai consumatori. Traccia la disponibilità del corriere e la sua posizione attuale.
* **Address**: l'indirizzo di un cliente o di un ristorante.
* **Location**: la latitudine e la longitudine di un corriere.

Un diagramma delle classi come questo ci illustra un aspetto dell'architettura di un'applicazione ma il passo successivo consiste nel definire le operazioni di sistema, che corrispondono agli scenari architetturali.

## SLIDE 02-02-06

<figure><img src="../.gitbook/assets/Screenshot 2023-08-17 alle 16.38.31.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Screenshot 2023-08-17 alle 16.38.35.png" alt=""><figcaption></figcaption></figure>

Quindi, una volta definito un modello di dominio ad alto livello, il passo successivo consiste nell'identificare le richieste che l'applicazione deve gestire. I dettagli dell'interfaccia utente esulano da questo corso ma è possibile immaginare che in ogni scenario dell'utente, l'interfaccia utente effettuerà richieste alla logica di business del backend per recuperare e/o aggiornare i dati.&#x20;

UrbanEats è principalmente un'applicazione web, il che significa che la maggior parte delle richieste sono basate su HTTP, ma è possibile che alcuni client utilizzino la messaggistica. Pertanto, anziché impegnarsi in un protocollo specifico, ha senso utilizzare la nozione più astratta di operazione di sistema per rappresentare le richieste.

Come accennato in precedenza, esistono due tipi di operazioni di sistema:

* **Comandi**: operazioni di sistema che creano, aggiornano ed eliminano dati
* **Query**: operazioni di sistema che leggono (effettuano query) dati

Alla fine queste operazioni corrisponderanno a endpoint REST, RPC o di messaggistica, ma per ora è più utile pensarci in modo astratto. Vediamo se possibile identifichiamo innanzitutto alcuni comandi di sistema.

Un buon punto di partenza per identificare i comandi di sistema è analizzare i verbi nelle storie e negli scenari dell'utente. Considera, ad esempio, la storia "Effettua un ordine". Essa suggerisce chiaramente che il sistema deve fornire un'operazione "Crea ordine". Molte altre storie si mappano direttamente in singole operazioni di sistema.&#x20;

## SLIDE 02-02-07

<figure><img src="../.gitbook/assets/Screenshot 2023-08-17 alle 16.58.52.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Screenshot 2023-08-17 alle 16.59.37.png" alt=""><figcaption></figcaption></figure>

Un comando ha una specifica che ne definisce i parametri, il valore restituito e il comportamento in termini delle classi del modello di dominio. Abbiamo precondizioni che devono essere vere quando l'operazione viene invocata e post-condizioni che sono vere dopo che l'operazione è stata invocata.

## SLIDE 02-02-08

Una strategia per creare un'architettura a microservizi è la decomposizione in base alle capacità aziendali cioè ciò che un'azienda fa al fine di generare valore e che ovviamente dipende dal tipo di attività. Ad esempio, le capacità di una compagnia di assicurazioni includono tipicamente l'Underwriting (valutazione del rischio), la gestione dei sinistri, la fatturazione, la conformità e così via. Le capacità di un negozio online includono la gestione degli ordini, la gestione dell'inventario, la spedizione e così via.

Le capacità aziendali di un'organizzazione vengono identificate analizzando lo scopo, la struttura e i processi aziendali e sono importanti perchè ogni capacità aziendale può essere considerata come un servizio, ma orientato al business anziché al tecnico. La sua specifica è composta da vari componenti, inclusi input, output e SLA.&#x20;

Nel caso di UrbanEats le capacità aziendali potrebbero essere le seguenti:&#x20;

* Gestione dei fornitori
  * Gestione dei corrieri
  * Gestione delle informazioni sui ristoranti, sui menu e altre informazioni come posizione e gli orari
* Gestione dei consumatori&#x20;
* Presa e soddisfazione degli ordini
  * Gestione degli ordini
  * Gestione degli ordini presso il ristorante&#x20;
  * Logistica
  * Gestione della disponibilità dei corrieri
  * Gestione delle consegne
* Contabilità
  * Contabilità dei consumatori&#x20;
  * Contabilità dei ristoranti
  * Contabilità dei corrieri

Le capacità di livello superiore includono Gestione dei fornitori, Gestione dei consumatori, Presa e soddisfazione degli ordini e Contabilità e ce ne sarebbero molte altre capacità di livello superiore, incluse quelle legate al marketing.

Un aspetto interessante di questa gerarchia è che ci sono tre capacità diverse relative ai ristoranti: Gestione delle informazioni sui ristoranti, Gestione degli ordini presso il ristorante e Contabilità dei ristoranti. Questo perché rappresentano tre aspetti molto diversi delle operazioni del ristorante

## SLIDE 02-02-09

<figure><img src="../.gitbook/assets/Screenshot 2023-08-17 alle 17.20.04.png" alt=""><figcaption></figcaption></figure>

Una volta identificate le capacità aziendali, è necessario definire un servizio per ognuna di queste capacità o gruppo di capacità correlate.In figura vediamo la corrispondenza tra le capacità e i servizi per l'applicazione di UrbanEats. Alcune capacità di livello superiore, come la capacità di Contabilità, sono mappate su servizi.

La decisione su quale livello della gerarchia delle capacità mappare su servizi è in parte soggettiva e potrebbe variare. La mia giustificazione per questa mappatura particolare è la seguente:

* Abbiamo mappato le sottocapacità della Gestione dei fornitori su due servizi, poiché i Ristoranti e i Corrieri sono tipi di fornitori molto diversi.
* Abbiamo mappato la capacità di Prenotazione ordine su tre servizi, ognuno responsabile di diverse fasi del processo. Ho combinato le capacità di Gestione della disponibilità dei corrieri e Gestione delle consegne e le ho mappate su un singolo servizio perché sono strettamente intrecciate.
* Contabilità ha senso che stia su un proprio servizio, poiché i diversi tipi di contabilità sembrano simili. In seguito, potrebbe avere senso separare i pagamenti (dei Ristoranti e dei Corrieri) e la fatturazione (dei Consumatori).

Un vantaggio chiave nell'organizzare i servizi attorno alle capacità è che, poiché sono stabili, l'architettura risultante sarà anch'essa relativamente stabile. I singoli componenti dell'architettura possono evolversi man mano che cambia l'aspetto operativo del business, ma l'architettura rimane invariata.

Detto questo, ricordiamoci che i servizi mostrati sono solo il primo tentativo di definire l'architettura e che possono evolvere nel tempo man mano che apprendiamo di più sul dominio dell'applicazione.&#x20;

## SLIDE 02-02-10

<figure><img src="../.gitbook/assets/Screenshot 2023-08-17 alle 17.27.55.png" alt=""><figcaption></figcaption></figure>

Vediamo ora un approccio alternativo e complementare alla suddivisione basata su capacità aziendali.&#x20;

Il Domain-driven design (DDD) è un approccio per costruire applicazioni software complesse e pone il focus sulla creazione di un modello di dominio orientato agli oggetti. Abbiamo due concetti che ci tornano utili quando si applica l'architettura a microservizi: i sotto-domini e i contesti delimitati (bounded contexts), che vedremo a breve

DDD è piuttosto diverso dall'approccio tradizionale alla modellazione aziendale che crea un singolo modello per l'intera azienda dove ad esempio possiamo trovare una singola definizione per ogni entità aziendale, come cliente, ordine, e così via. Il problema è che ottenere diverse parti di un'organizzazione a concordare su un unico modello è un compito non banale, inoltre significa che dal punto di vista di una data parte dell'organizzazione, il modello è eccessivamente complesso per le loro esigenze. DDD evita questi problemi definendo più modelli di dominio, ognuno con uno scopo esplicito.

"Scomporre per sotto-dominio" e "Scomporre per capacità aziendali" sono i due principali modelli per definire l'architettura a microservizi di un'applicazione. Tuttavia, ci sono alcune linee guida utili per la decomposizione che hanno radici nella progettazione orientata agli oggetti.

## SLIDE 02-02-11

Possiamo anche adattare e utilizzare un paio di principi provenienti dal design orientato agli oggetti quando applichiamo il pattern dell'architettura a microservizi, il primo principio è il Principio di Singola Responsabilità (SRP) e secondo è il Principio di Chiusura Comune (CCP)

**PRINCIPIO DI SINGOLA RESPONSABILITÀ** Uno degli obiettivi principali dell'architettura e del design del software è determinare le responsabilità di ciascun elemento software.&#x20;

Il Principio di Singola Responsabilità è il seguente: "Una classe dovrebbe avere una sola ragione per cambiare." Robert C. Martin&#x20;

Ogni responsabilità che una classe ha è una potenziale ragione per cui quella classe potrebbe cambiare. Se una classe ha più responsabilità che cambiano indipendentemente, la classe non sarà stabile. Si definiscono classi ciascuna con una sola responsabilità e quindi una sola ragione per il cambiamento. Possiamo applicare il SRP quando definiamo un'architettura a microservizi e creiamo servizi piccoli e coesi, ciascuno con una sola responsabilità. Questo ridurrà le dimensioni dei servizi e aumenterà la loro stabilità. La nuova architettura UrbanEats è un esempio di SRP in azione. Ogni aspetto di portare il cibo a un consumatore, come la presa dell'ordine, la preparazione dell'ordine e la consegna, è la responsabilità di un servizio separato.

**PRINCIPIO DI CHIUSURA COMUNE** L'altro principio utile è il Principio di Chiusura Comune: "Le classi in un pacchetto dovrebbero essere chiuse insieme contro gli stessi tipi di cambiamenti. Una modifica che riguarda un pacchetto riguarda tutte le classi in quel pacchetto." Robert C. Martin&#x20;

L'idea è che se due classi cambiano insieme a causa della stessa ragione sottostante, allora appartengono allo stesso pacchetto.&#x20;

Possiamo applicare il CCP quando creiamo un'architettura a microservizi e raggruppiamo i componenti che cambiano per la stessa ragione nello stesso servizio. Facendo ciò, si ridurrà al minimo il numero di servizi che devono essere modificati e distribuiti quando cambia un requisito. Idealmente, una modifica influirà solo su un singolo team e un singolo servizio. CCP è l'antidoto all'anti-pattern del monolite distribuito.

## SLIDE 02-02-12

La definizione di sotto domini o capacità aziendale può sembra facile e intuitiva ma ci sono alcuni ostacoli:&#x20;

* **Latenza di rete**: Potresti scoprire che una particolare suddivisione in servizi comporta un gran numero di round-trip tra due servizi. A volte è possibile ridurre la latenza a un livello accettabile implementando un'API batch per recuperare più oggetti in un singolo round-trip
* **Comunicazione** **sincrona**: Un altro problema riguarda il modo in cui implementare la comunicazione tra i servizi in modo da non ridurre la disponibilità. Ad esempio, il modo più diretto per implementare l'operazione createOrder() è far sì che il servizio Ordine invochi in modo sincrono gli altri servizi tramite REST. Lo svantaggio dell'utilizzo di un protocollo come REST è che riduce la disponibilità del servizio Ordine. Non sarà in grado di creare un ordine se uno qualsiasi di quei servizi è indisponibile.
* **Coerenza dei dati**: Alcune operazioni di sistema devono aggiornare i dati in più servizi. Ad esempio, quando un ristorante accetta un ordine, gli aggiornamenti devono avvenire sia nel servizio Cucina che nel servizio Consegna. Il servizio Cucina modifica lo stato del Ticket. Il servizio Consegna pianifica la consegna dell'ordine. Entraggi gli aggiornamenti devono essere fatti in modo atomico (vedremo più avanti come risolvere questo problema)&#x20;
* **Visione consistente dei dati**: Un altro ostacolo alla decomposizione è l'incapacità di ottenere una visione veramente coerente dei dati attraverso più database. In un'applicazione monolitica, le proprietà delle transazioni ACID garantiscono che una query restituirà una visione coerente del database. Al contrario, in un'architettura a microservizi, anche se il database di ciascun servizio è coerente, non è possibile ottenere una visione globalmente coerente dei dati. Vedremo che comunque nella pratica questo raramente è un problema

## SLIDE 02-02-13

<figure><img src="../.gitbook/assets/Screenshot 2023-08-17 alle 18.52.56.png" alt=""><figcaption></figcaption></figure>

Un altro ostacolo alla decomposizione è l'esistenza delle cosiddette classi "god" o "divine". Queste classi "god" sono classi sovradimensionate che vengono utilizzate in tutta l'applicazione. Di solito implementano logiche aziendali per molti aspetti diversi dell'applicazione. Normalmente hanno un gran numero di campi mappati su una tabella di database con molte colonne. La maggior parte delle applicazioni ha almeno una di queste classi, ciascuna rappresentando un concetto centrale nel dominio. La classe Order è un ottimo esempio di una classe "god" nell'applicazione di UrbanEats dato che lo scopo è consegnare ordini di cibo ai clienti.&#x20;

Una soluzione è impacchettare la classe Order in una libreria e creare un database centrale per gli ordini. Tutti i servizi che elaborano gli ordini utilizzano questa libreria e accedono al database centrale. Il problema di questo approccio è che viola uno dei principi chiave dell'architettura a microservizi e comporta un accoppiamento stretto indesiderabile. Ad esempio, qualsiasi modifica allo schema dell'Ordine richiede che i team aggiornino il proprio codice in sincronia.

Un'altra soluzione è quella di incapsulare il database degli Ordini in un servizio Ordine, che viene invocato dagli altri servizi per recuperare e aggiornare gli ordini. Il problema di questo design è che il servizio Ordine sarebbe un servizio dati con un modello di dominio anemico contenente poca o nessuna logica aziendale. Nessuna di queste opzioni è attraente, ma fortunatamente, il DDD fornisce una soluzione.

## SLIDE 02-02-14

<figure><img src="../.gitbook/assets/Screenshot 2023-08-17 alle 19.02.39.png" alt=""><figcaption></figcaption></figure>

Un approccio molto migliore è applicare il DDD e trattare ogni servizio come un sottodominio separato con il proprio modello di dominio. Questo significa che ciascuno dei servizi nell'applicazione UrbanEats che ha a che fare con gli ordini ha il proprio modello di dominio con la sua versione della classe Ordine. Un ottimo esempio del vantaggio di avere modelli di dominio multipli è il servizio di consegna. La sua visione di un Ordine, mostrata nella slide, è estremamente semplice: indirizzo di ritiro, orario di ritiro, indirizzo di consegna e orario di consegna. Inoltre, anziché chiamarlo Ordine, il servizio di consegna utilizza il nome più appropriato di _Consegna_. Notiamo che servizio di consegna non è interessato ad alcun altro attributo di un ordine.

<figure><img src="../.gitbook/assets/Screenshot 2023-08-17 alle 19.03.45.png" alt=""><figcaption></figcaption></figure>

Anche il servizio Cucina ha una visione molto più semplice di un ordine. La sua versione di un Ordine è chiamata Biglietto. Come mostra la figura un Ticket consiste semplicemente in uno stato, l'orario di consegna richiesto, un orario di preparazione e un elenco di voci che dicono al ristorante cosa preparare. Non gli interessa il cliente, il pagamento, la consegna e così via.

<figure><img src="../.gitbook/assets/Screenshot 2023-08-17 alle 19.05.09.png" alt=""><figcaption></figcaption></figure>

Il servizio Ordine ha la visione più complessa di un ordine, come mostrato in figura. Anche se ha molti campi e metodi, è comunque molto più semplice rispetto alla versione originale vista poco fa. La classe Ordine in ciascun modello di dominio rappresenta aspetti diversi della stessa entità commerciale Ordine.&#x20;

Chiaramente l'applicazione deve mantenere la coerenza tra questi oggetti diversi in diversi servizi. Ad esempio, una volta che il servizio Ordine ha autorizzato la carta di credito del consumatore, deve innescare la creazione del Biglietto nel servizio Cucina. Allo stesso modo, se il ristorante rifiuta l'ordine tramite il servizio Cucina, deve essere annullato nel servizio Ordine e al cliente viene accreditato il pagamento nel servizio di fatturazione. Vedremo più avanti come mantenere questa coerenza.&#x20;

## SLIDE 02-02-15

<figure><img src="../.gitbook/assets/Screenshot 2023-08-18 alle 09.58.11.png" alt=""><figcaption></figcaption></figure>

Finora abbiamo analizzato una lista di operazioni di sistema e una lista di servizi potenziali, il passo successivo è definire l'API di ciascun servizio cioè capire le operazioni e gli eventi associati. Alcune operazioni corrispondono ad operazioni di sistema invocate da client esterni e forse da altri servizi. Le altre operazioni invece esistono per supportare la collaborazione tra i servizi e sono invocate solo da altri servizi.&#x20;

Il punto di partenza per definire le API dei servizi è mappare ciascuna operazione di sistema su un servizio. Successivamente decidiamo se un servizio deve collaborare con gli altri per implementare un'operazione di sistema. Se è necessaria la collaborazione, determiniamo quindi quali API devono fornire gli altri servizi per supportare la collaborazione. Iniziamo vedendo come assegnare le operazioni di sistema ai servizi.

Il primo passo è decidere quale servizio è il punto di ingresso iniziale per una richiesta, molte operazioni di sistema si mappano perfettamente su un servizio, ma talvolta la mappatura è meno ovvia. Considera, ad esempio, l'operazione _noteUpdatedLocation()_, che aggiorna la posizione del corriere. Da un lato, poiché è correlata ai corrieri, questa operazione dovrebbe essere assegnata al servizio di Corrieri. D'altra parte, è il Servizio di Consegna che ha bisogno della posizione del corriere. In tabella vediamo quali servizi nell'applicazione UberEats sono responsabili di quali operazioni. Dopo aver assegnato le operazioni ai servizi, il passo successivo è decidere come i servizi collaborano per gestire le varie operazione di sistema.

## SLIDE 02-02-16

<figure><img src="../.gitbook/assets/Screenshot 2023-08-18 alle 10.02.50.png" alt=""><figcaption></figcaption></figure>

In tabella vediamo la lista delle operazioni e notiamo che alcune di queste vengono gestite interamente da un singolo servizio. Ad esempio, nell'applicazione UrbanEats, il servizio _Consumer_ gestisce l'operazione _createConsumer()_ completamente da solo. Altre operazioni di sistema invece coinvolgono più servizi e i dati necessari per gestire una di queste richieste potrebbero essere distribuiti su più servizi.&#x20;

Ad esempio, per implementare l'operazione _createOrder()_, il servizio Order deve invocare i seguenti servizi per verificare le sue precondizioni e rendere _vere_ le postcondizioni:

* **Servizio Consumer**: verifica che il cliente possa effettuare un ordine e ottiene le informazioni di pagamento.
* **Servizio Ristorante**: valida gli articoli dell'ordine, verifica che l'indirizzo/orario di consegna sia nell'area di servizio del ristorante, verifica che l'importo minimo dell'ordine sia raggiunto e ottiene i prezzi degli articoli dell'ordine.
* **Servizio Cucina**: crea il Ticket (che sarebbe l'ordine da preparare) .
* **Servizio Contabilità**: autorizza la carta di credito del cliente.&#x20;

Allo stesso modo per implementare l'operazione _acceptOrder()_, il servizio Kitchen deve invocare il servizio Delivery per pianificare un corriere per la consegna dell'ordine. La Tabella nella slide mostra i servizi, le loro API riviste e i loro collaboratori. Per definire completamente le API dei servizi, è necessario analizzare ciascuna operazione di sistema e determinare quale tipo di collaborazione è richiesta.

Finora abbiamo identificato i servizi e le operazioni che ciascun servizio implementa. Tuttavia, è importante ricordare che l'architettura che abbiamo delineato è ancora molto astratta perché non abbiamo scelto nessuna tecnologia, inoltre dobbiamo considerare per le varie "_operazioni_" se abbiamo meccanismi sincroni o asincroni.&#x20;

## SLIDE 02-02-17

Come abbiamo già accennato più volte, i concetti di **accoppiamento** e **coesione** sono ovviamente correlati perché se la funzionalità correlata è distribuita in tutto il nostro sistema, le modifiche a questa funzionalità si propagheranno attraverso il dominio e questo implica un accoppiamento più stretto. C'è una legge empirica chiamata "La legge di Constantine" che ci dice che "Una struttura è stabile se la coesione è forte e l'accoppiamento è basso."

Il concetto di stabilità chiaramente qui è importante per noi per raggiungere l'obiettivo di deploy indipendenti, di lavorare in parallelo e ridurre il coordinamento tra i vari team. Se il contratto che un microservizio espone cambia costantemente in modo incompatibile con le versioni precedenti obblighiamo il cambiamento anche in altri punti del software

La coesione si applica alla relazione tra le cose all'interno di un confine preciso (un microservizio nel nostro contesto), mentre l'accoppiamento descrive la relazione tra le cose attraverso un dominio. Non esiste un modo assolutamente migliore per organizzare il nostro codice; l'accoppiamento e la coesione sono solo un modo per esprimere i vari compromessi che facciamo su dove raggruppare il codice e perché. Tutto ciò a cui possiamo aspirare è trovare il giusto equilibrio tra queste due idee, un equilibrio che abbia più senso per il contesto specifico e per i problemi che stai affrontando attualmente.

Inoltre ricordiamoci che il mondo non è statico: è possibile che, man mano che i requisiti del  sistema cambiano, si troveranno ragioni per riesaminare le decisioni. A volte alcune parti del tuo sistema potrebbero essere soggette a così tanti cambiamenti da rendere impossibile la stabilità.&#x20;
