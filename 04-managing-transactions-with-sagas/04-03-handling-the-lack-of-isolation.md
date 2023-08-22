# 04-03 - Handling the lack of isolation

## SLIDE 04-03-01

La "I" in ACID sta per isolamento che ci garantisce che l'esito dell'esecuzione di più transazioni contemporaneamente sia lo stesso come se fossero eseguite in un certo ordine seriale. Il database fornisce l'illusione che ogni transazione ACID abbia accesso esclusivo ai dati. L'isolamento rende molto più facile scrivere logiche di business che si eseguono in modo concorrente.

La sfida nell'uso delle saghe è che mancano della proprietà di isolamento delle transazioni ACID e questo chiaramente perché gli aggiornamenti effettuati da ciascuna transazione locale di una saga sono immediatamente visibili ad altre saghe una volta che quella transazione è confermata.&#x20;

Questo comportamento può causare due problemi. In primo luogo, altre saghe possono modificare i dati accessibili dalla saga mentre essa è in esecuzione e altre saghe possono leggere i suoi dati prima che la saga abbia completato gli aggiornamenti, e di conseguenza possono essere esposte a dati incoerenti.&#x20;

Si può considerare una saga come ACD:

* **Atomicità**: l'implementazione della saga garantisce che tutte le transazioni vengano eseguite o che tutte le modifiche vengano annullate.
* **Coerenza**: l'integrità referenziale all'interno di un servizio è gestita dai database locali. L'integrità referenziale tra servizi è gestita dai servizi stessi.
* **Durabilità**: gestita dai database locali.

Questa mancanza di isolamento potenzialmente provoca quello che la letteratura sui database chiama **anomalia**. Un'anomalia si verifica quando una transazione legge o scrive dati in un modo che non farebbe se le transazioni fossero eseguite una alla volta. Quando si verificano anomalie, l'esito dell'esecuzione di saghe in modo concorrente è diverso da quello se fossero eseguite in modo seriale.

Nella pratica è comune che gli sviluppatori accettino un isolamento ridotto in cambio di prestazioni migliori. Ad esempio un RDBMS consente di specificare il livello di isolamento per ciascuna transazione e di solito il livello predefinito è più debole rispetto all'isolamento completo, noto anche come transazioni serializzabili.&#x20;

Le transazioni di database nel mondo reale spesso differiscono dalle definizioni teoriche delle transazioni ACID. Le strategie di progettazione delle saghe che affrontano la mancanza di isolamento vengono chiamate come **contromisure**.&#x20;

Alcune contromisure implementano l'isolamento a livello _applicativo,_ altre contromisure riducono il rischio aziendale della mancanza di isolamento. Utilizzando queste contromisure, è possibile scrivere logiche aziendali basate su saghe che funzionano correttamente.&#x20;

## SLIDE 04-03-02

La mancanza di isolamento può causare le seguenti **tre anomalie**:

* Perdita di **aggiornamenti**: Una saga sovrascrive senza leggere le modifiche apportate da un'altra saga.
* **Letture sporche**: Una transazione o una saga legge le modifiche apportate da una saga che non ha ancora completato tali aggiornamenti.
* **Letture fuzzy/non ripetibili:** Due passaggi diversi di una saga leggono gli stessi dati e ottengono risultati diversi perché un'altra saga ha apportato aggiornamenti.&#x20;

Tutte e tre le anomalie possono verificarsi, ma le prime due sono le più comuni e le più problematiche.&#x20;

#### Perdita di Aggiornamento

Si verifica quando una saga sovrascrive un aggiornamento effettuato da un'altra saga. Considera, ad esempio, lo scenario seguente:&#x20;

1. Il primo passo della saga di Creazione Ordine crea un Ordine.
2. Mentre quella saga è in esecuzione, la saga di Annullamento Ordine annulla l'Ordine.
3. Il passo finale della saga di Creazione Ordine approva l'Ordine.&#x20;

In questo scenario, la saga di Creazione Ordine ignora l'aggiornamento effettuato dalla saga di Annullamento Ordine e lo sovrascrive. Di conseguenza, l'applicazione UrbanEats spedirà un ordine che il cliente aveva annullato.&#x20;

#### Letture Sporche

Una lettura sporca si verifica quando una saga legge dati che sono in fase di aggiornamento da parte di un'altra saga. Considera, ad esempio, una versione dell'applicazione UrbanEats in cui i consumers hanno un limite di credito. In questa applicazione, una saga che annulla un ordine è composta dalle seguenti transazioni:

* _Servizio Consumatore:_ Aumenta il credito disponibile.
* _Servizio Ordine_: Cambia lo stato dell'Ordine in annullato.
* _Servizio Consegna_: Annulla la consegna.&#x20;

Immaginiamo uno scenario che intervalla l'esecuzione delle saghe di Annullamento Ordine e Creazione Ordine, e la saga di Annullamento Ordine viene annullata perché è troppo tardi per annullare la consegna.&#x20;

È possibile che la sequenza di transazioni che invoca il Servizio Consumer sia la seguente:&#x20;

1. Saga di Annullamento Ordine: Aumenta il credito disponibile. &#x20;
2. Saga di Creazione Ordine: Riduce il credito disponibile.&#x20;
3. Saga di Annullamento Ordine: Una transazione compensativa che riduce il credito disponibile.&#x20;

In questo scenario, la saga di Creazione Ordine effettua una lettura sporca del credito disponibile che consente al consumatore di effettuare un ordine che supera il proprio limite di credito. È probabile che questo rappresenti un rischio inaccettabile per l'azienda.&#x20;

## SLIDE 04-03-03

È responsabilità degli sviluppatori (e quindi nostra) scrivere saghe in modo tale da prevenire le anomalie o ridurne al minimo l'impatto sul business utilizzando strategie di prevenzione.

L'uso degli stati _\*\_PENDING_ di un Ordine, come APPROVAL\_PENDING, è un esempio di una possibile strategia. Le saghe che aggiornano gli Ordini, come la Saga di Creazione Ordine, iniziano impostando lo stato di un Ordine su _\*\_PENDING_. Lo stato _\*\_PENDING_ comunica ad altre transazioni che l'Ordine è oggetto di un aggiornamento da parte di una saga e di agire di conseguenza.&#x20;

L'uso degli stati \*\_PENDING di un Ordine è un esempio di ciò che possiamo chiamare **"contromisura a blocco semantico"**. Il paper descrive come affrontare la mancanza di isolamento delle transazioni in architetture multi-database che non utilizzano transazioni distribuite. Molti dei suoi concetti sono utili nella progettazione delle saghe.&#x20;

Esso descrive un insieme di contromisure per gestire le anomalie causate dalla mancanza di isolamento che prevengono una o più anomalie o ne riducono al minimo l'impatto sul business. Le contromisure che possiamo trovare in letteratura sono:

* _Blocco semantico:_ Un blocco a livello applicativo
* _Aggiornamenti commutativi:_ Progetta le operazioni di aggiornamento in modo che siano eseguibili in qualsiasi ordine
* _Visione pessimistica:_ Riordina i passaggi di una saga per minimizzare il rischio commerciale
* _Rilettura del valore_: Previene scritture sporche rileggendo i dati per verificare che siano invariati prima di sovrascriverli
* _File di versione:_ Registra gli aggiornamenti a un record in modo che possano essere riordinati
* _Per valore:_ Utilizza il rischio commerciale di ciascuna richiesta per selezionare dinamicamente il meccanismo di concorrenza

## SLIDE 04-03-04

<figure><img src="../.gitbook/assets/Screenshot 2023-08-22 alle 16.09.19.png" alt=""><figcaption></figcaption></figure>

Le contromisure menzionate nella sezione precedente definiscono un modello utile per la struttura di una saga. Nel modello illustrato in questa slide vediamo una saga come sia composta da tre tipi di transazioni:&#x20;

* **Transazioni compensabili**: possono essere potenzialmente annullate utilizzando una transazione compensativa
* **Transazione pivot**: il punto di _go/no-go_ in una saga. Se la transazione pivot viene eseguita, la saga verrà eseguita fino al completamento può essere una transazione che non è né compensabile né ripetibile. In alternativa, può essere l'ultima transazione compensabile o la prima transazione ripetibile
* **Transazioni ripetibili:** transazioni che seguono la transazione pivot e sono garantite di avere successo.

Nella Saga di _Creazione Ordine_, i passaggi _createOrder()_, _verifyConsumerDetails()_ e _createTicket()_ sono transazioni **compensabili**.&#x20;

Le transazioni _createOrder()_ e _createTicket()_ hanno transazioni **compensative** che annullano i loro aggiornamenti.&#x20;

La transazione _verifyConsumerDetails()_ è in sola lettura, quindi non necessita di una transazione compensativa.&#x20;

La transazione _authorizeCreditCard()_ è la transazione _pivot_ di questa saga. Se la carta di credito del consumatore può essere autorizzata, questa saga è garantita.&#x20;

I passaggi _approveTicket()_ e _approveOrder()_ sono transazioni _ripetibili_ che seguono la transazione pivot.&#x20;

Esaminiamo ciascuna contromisura, iniziando dalla contromisura del blocco semantico.

## SLIDE 04-03-05

Quando si utilizza la contromisura a blocco semantico, la transazione compensabile di una saga imposta un flag su ogni record che crea o aggiorna e indica che il record non è stato ancora confermato e potrebbe subire modifiche.&#x20;

Il flag può essere sia un blocco che impedisce ad altre transazioni di accedere al record, sia un avviso che indica che altre transazioni dovrebbero trattare quel record con sospetto. Viene cancellato sia da una transazione ripetibile - la saga sta completando con _successo_ - sia da una transazione compensativa: la saga sta _annullando_.

Il _campo Order.state_ ad esempio è un ottimo esempio di un blocco semantico. Gli stati _\*\_PENDING_, come _APPROVAL\_PENDING_ e _REVISION\_PENDING_, implementano un blocco semantico, comunicano ad altre saghe che accedono a un Ordine che una saga sta aggiornando quell'ordine.&#x20;

Ad esempio, il primo passo della Create Order Saga, che è una transazione compensabile, crea un Ordine in uno stato APPROVAL\_PENDING. L'ultimo passo della Create Order Saga, che è una transazione ripetibile, modifica il campo in APPROVED. Una transazione compensativa modificherebbe il campo in REJECTED.

La gestione del blocco è solo metà del problema. È necessario decidere caso per caso come una saga dovrebbe gestire un record bloccato. Prendi ad esempio il comando di sistema cancelOrder(). Un cliente potrebbe invocare questa operazione per annullare un Ordine che si trova nello stato _APPROVAL\_PENDING_.

Ci sono alcune opzioni diverse per gestire questo scenario. Un'opzione è far fallire il comando di sistema cancelOrder() e chiedere al cliente di riprovare più tardi. Il principale vantaggio di questo approccio è che è semplice da implementare. Lo svantaggio, tuttavia, è che rende più complesso il client applicativo in quanto deve implementare la logica di ripetizione.

Un'altra opzione è far sì che cancelOrder() si blocchi fino al rilascio del blocco. Un vantaggio nell'utilizzare i blocchi semantici è che essi ricreano essenzialmente l'isolamento fornito dalle transazioni ACID.&#x20;

Le saghe che aggiornano lo stesso record vengono serializzate (cioè eseguite una dopo l'altra), riducendo significativamente lo sforzo di programmazione. Un altro vantaggio è che rimuovono l'onere dei tentativi dal cliente. Lo svantaggio è che l'applicazione deve gestire i blocchi. Deve inoltre implementare un algoritmo di rilevamento dei **deadlock** che esegue un rollback di una saga per interrompere un deadlock e rieseguirla.

## SLIDE 04-03-06

Una contromisura diretta è progettare le operazioni di aggiornamento in modo che siano **commutative**.cioè se possono essere eseguite in qualsiasi ordine. Le operazioni di _debito()_ e _credito()_ di un Account sono _commutative_ (se si ignorano i controlli di scoperto). Questa contromisura è utile perché elimina gli aggiornamenti persi.&#x20;

Considera, ad esempio, uno scenario in cui è necessario eseguire il rollback di una saga dopo che una transazione compensativa ha addebitato (o accreditato) un conto. La transazione compensativa può semplicemente accreditare (o addebitare) il conto per annullare l'aggiornamento. Non c'è la possibilità di sovrascrivere gli aggiornamenti effettuati da altre saghe.

Un altro modo per gestire la mancanza di isolamento è la contromisura della "**visualizzazione pessimistica**". Questa contromisura _riordina_ i passaggi di una saga per ridurre al minimo il rischio di business dovuto a una lettura sporca.&#x20;

Considera, ad esempio, lo scenario precedente per descrivere l'anomalia della lettura sporca. In tale scenario, la saga Create Order ha effettuato una lettura sporca del credito disponibile e ha creato un ordine che superava il limite di credito del consumatore. Per ridurre il rischio che ciò accada, questa contromisura riordinerebbe la saga Cancel Order:&#x20;

1. Order Service: Cambia lo stato dell'ordine a annullato
2. DeliveryService: Annulla la consegna
3. CustomerService: Aumenta il credito disponibile

In questa versione riordinata della saga, il credito disponibile viene aumentato in una transazione ripetibile, eliminando la possibilità di una lettura sporca

## SLIDE 04-03-07

La contromisura "**version file**" prende questo nome perché registra le operazioni eseguite su un record in modo che possano essere riordinate. È un modo per trasformare operazioni non commutative in operazioni commutative.&#x20;

Per capire come funziona questa contromisura, considera uno scenario in cui la saga Create Order viene eseguita in modo concorrente con una saga Cancel Order. A meno che le saghe non utilizzino la contromisura del blocco semantico, è possibile che la saga Cancel Order annulli l'autorizzazione della carta di credito del consumatore prima che la saga Create Order autorizzi la carta.

Un modo per far fronte a queste richieste fuori sequenza nell'Accounting Service è registrare le operazioni man mano che arrivano e quindi eseguirle nell'ordine corretto. In questo scenario, registrerebbe prima la richiesta di Annullamento Autorizzazione. Poi, quando il servizio Accounting riceve la successiva richiesta di Autorizzazione Carta, noterebbe di aver già ricevuto la richiesta di Annullamento Autorizzazione e saltarebbe l'autorizzazione della carta di credito.

L'ultima contromisura è la contromisura "**by value**". È una strategia per selezionare i meccanismi di concorrenza in base al rischio di business. Un'applicazione che utilizza questa contromisura utilizza le proprietà di ciascuna richiesta per decidere se utilizzare saghe o transazioni distribuite. Esegue richieste a basso rischio utilizzando saghe, applicando forse le contromisure descritte fino ad ora.&#x20;

Ma esegue richieste ad alto rischio che coinvolgono, ad esempio, grandi importi di denaro, utilizzando transazioni distribuite. Questa strategia consente a un'applicazione di prendere dinamicamente decisioni in merito al rischio, alla disponibilità e alla scalabilità.
