# 02-03-Coupling types

## SLIDE 02-03-01

Come abbiamo già accennato più volte, i concetti di **accoppiamento** e **coesione** sono ovviamente correlati perché se la funzionalità correlata è distribuita in tutto il nostro sistema, le modifiche a questa funzionalità si propagheranno attraverso il dominio e questo implica un accoppiamento più stretto. C'è una legge empirica chiamata "La legge di Constantine" che ci dice che "Una struttura è stabile se la coesione è forte e l'accoppiamento è basso."

Il concetto di stabilità chiaramente qui è importante per noi per raggiungere l'obiettivo di deploy indipendenti, di lavorare in parallelo e ridurre il coordinamento tra i vari team. Se il contratto che un microservizio espone cambia costantemente in modo incompatibile con le versioni precedenti obblighiamo il cambiamento anche in altri punti del software

La coesione si applica alla relazione tra le cose all'interno di un confine preciso (un microservizio nel nostro contesto), mentre l'accoppiamento descrive la relazione tra le cose attraverso un dominio. Non esiste un modo assolutamente migliore per organizzare il nostro codice; l'accoppiamento e la coesione sono solo un modo per esprimere i vari compromessi che facciamo su dove raggruppare il codice e perché. Tutto ciò a cui possiamo aspirare è trovare il giusto equilibrio tra queste due idee, un equilibrio che abbia più senso per il contesto specifico e per i problemi che stai affrontando attualmente.

Inoltre ricordiamoci che il mondo non è statico: è possibile che, man mano che i requisiti del  sistema cambiano, si troveranno ragioni per riesaminare le decisioni. A volte alcune parti del tuo sistema potrebbero essere soggette a così tanti cambiamenti da rendere impossibile la stabilità.&#x20;

Vediamo alcuni tipo di coupling più comuni:&#x20;

* Domain&#x20;
* Temporal
* Pass-through&#x20;
* Common&#x20;
* Content

## SLIDE 02-03-02

<figure><img src="../.gitbook/assets/Screenshot 2023-08-18 alle 12.14.08.png" alt=""><figcaption></figcaption></figure>

L'accoppiamento di dominio descrive una situazione in cui un microservizio deve interagire con un altro microservizio, perché il primo microservizio ha bisogno di utilizzare la funzionalità fornita dall'altro microservizio. In figura vediamo parte di come vengono gestiti gli ordini per i CD all'interno di una ipotetica azienda che produce musica. In questo esempio, il processore degli ordini chiama il microservizio del magazzino per riservare la merce e il microservizio di pagamento per effettuare il pagamento. Il processore degli ordini è quindi dipendente e accoppiato ai microservizi del magazzino e di pagamento per questa operazione. Non vediamo accoppiamento tra il magazzino e il pagamento, poiché non interagiscono.

In un'architettura a microservizi, questo tipo di interazione è in gran parte inevitabile perché ci basiamo su diversi microservizi che collaborano affinché possa svolgere il suo lavoro. Ricordiamo comunque che l'obiettivo è ridurre al minimo questo tipo di interazione

Come regola generale, l'accoppiamento di dominio è considerato una forma di accoppiamento meno stretta, anche se anche qui possono sorgere problemi. Un microservizio che deve comunicare con molti microservizi sottostanti potrebbe indicare una situazione in cui troppa logica è stata centralizzata.&#x20;

Come regola, condividi solo ciò che devi assolutamente condividere e invia solo la quantità minima di dati di cui hai bisogno.

## SLIDE 02-03-03

<figure><img src="../.gitbook/assets/Screenshot 2023-08-18 alle 12.21.11.png" alt=""><figcaption></figcaption></figure>

Un'altra forma di accoppiamento che si può verificare è l'**accoppiamento temporale**. Da una prospettiva incentrata sul codice si riferisce a una situazione in cui i concetti sono raggruppati insieme semplicemente perché si verificano nello stesso momento. L'accoppiamento temporale ha un significato leggermente diverso nel contesto di un sistema distribuito, dove si riferisce a una situazione in cui un microservizio ha bisogno di un altro microservizio per fare qualcosa allo stesso tempo affinché l'operazione possa essere completata.

Entrambi i microservizi devono essere attivi e disponibili per comunicare tra loro nello stesso momento affinché l'operazione possa essere completata. Ad esempio in questa figura dove il Processore degli ordini della nostra MusicCorp sta effettuando una chiamata HTTP sincrona al servizio Magazzino, il servizio Magazzino deve essere attivo e disponibile allo stesso tempo in cui viene effettuata la chiamata.

Se per qualche motivo il servizio Magazzino non può essere raggiunto dal Processore degli ordini, allora l'operazione fallisce, poiché non possiamo riservare i CD da spedire. Il Processore degli ordini dovrà anche bloccarsi e attendere una risposta dal servizio Magazzino, potenzialmente causando problemi in termini di contesa di risorse.

L'accoppiamento temporale _non è sempre negativo_; è solo qualcosa di cui essere consapevoli. Man mano che hai più microservizi, con interazioni più complesse tra di loro, le sfide dell'accoppiamento temporale possono aumentare fino a diventare più difficili da gestire e scalare il sistema. Uno dei modi per evitare l'accoppiamento temporale è utilizzare una forma di comunicazione asincrona, come un **message broker.**

## SLIDE 02-03-04

<figure><img src="../.gitbook/assets/Screenshot 2023-08-18 alle 12.26.38.png" alt=""><figcaption></figcaption></figure>

_il_ Pass-Through Coupling (o _"L'accoppiamento di passaggio")_ si verifica quando un microservizio trasferisce dati a un altro microservizio esclusivamente perché tali dati sono richiesti da un ulteriore microservizio più in fondo nella catena. Questa forma di accoppiamento è tra le più problematiche in termini di implementazione, poiché implica che il chiamante non solo è consapevole che il microservizio che sta invocando richiama un altro microservizio, ma potrebbe anche essere obbligato a comprendere come funzioni questo microservizio intermedio.

Come esempio di accoppiamento di passaggio, analizziamo più da vicino parte di come funziona l'elaborazione degli ordini di MusicCorp. In figura abbiamo un Processore degli Ordini che invia una richiesta all'entità Magazzino per preparare un ordine da spedire. Come parte del carico di richiesta, inviamo un Manifesto di Spedizione che contiene non solo l'indirizzo del cliente ma anche il tipo di spedizione. Il magazzino passa questo manifesto al microservizio di Spedizione a valle.

Il problema principale dell'accoppiamento di passaggio è che una modifica ai dati richiesti a valle può causare una modifica più significativa a monte. Nel nostro esempio, se ora Spedizione ha bisogno che il formato o il contenuto dei dati venga modificato, è probabile che sia necessario modificare sia Magazzino che Processore degli Ordini.

Ci sono alcuni modi per risolvere questo problema. Il primo è considerare se ha senso che il microservizio chiamante bypassi semplicemente l'intermediario oppure in alternativa delegare il lavoro direttamente al sesrvizio invocato&#x20;

## SLIDE 02-03-05

<figure><img src="../.gitbook/assets/Screenshot 2023-08-18 alle 12.36.52.png" alt=""><figcaption></figcaption></figure>

**Common Coupling** o "accoppiamento comune" si verifica quando due o più microservizi utilizzano un insieme di dati in comune tipicamente in un database condiviso, ma potrebbe manifestarsi anche nell'uso di memoria condivisa o di un sistema di file condiviso.&#x20;

Il principale problema è che le modifiche alla struttura dei dati possono influenzare contemporaneamente più microservizi. Se lo schema di questo database cambiassse in modo incompatibile, richiederebbe modifiche a ciascun utilizzatore del database. In pratica, i dati condivisi di questo tipo tendono a essere molto difficili da modificare come risultato.

Anche le risorse condivise possono rappresentare fonti di accoppiamento comune e, di conseguenza, potenziali fonti di conflitto per le risorse.

Più microservizi che utilizzano lo stesso sistema di file o database condiviso potrebbero sovraccaricare quella risorsa condivisa, causando potenzialmente problemi significativi se la risorsa condivisa diventa lenta o addirittura non disponibile affatto. Un database condiviso è particolarmente incline a questo problema, poiché più consumatori possono eseguire query arbitrarie contro il database stesso

Se stai lavorando su un microservizio, è fondamentale avere una chiara separazione tra ciò che può essere cambiato liberamente e ciò che non può. In modo esplicito, come sviluppatore devi sapere quando stai cambiando la funzionalità che fa parte del contratto che il tuo servizio espone all'esterno. Devi assicurarti che, apportando modifiche, non romperai i consumatori a monte. La funzionalità che non influisce sul contratto che il tuo microservizio espone può essere cambiata senza preoccupazioni.
