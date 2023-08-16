# Processi e organizzazione

## SLIDE 04-01

<figure><img src="../.gitbook/assets/Screenshot 2023-08-16 alle 12.23.14.png" alt=""><figcaption></figcaption></figure>

Per un'applicazione grande e complessa come già detto più volte l'architettura a microservizi è potenzialmente la scelta migliore, ma oltre ad avere la giusta architettura, lo sviluppo software di successo richiede anche un'adeguata organizzazione e processi di sviluppo e consegna. La Figura mostra le relazioni tra processo, organizzazione e architettura.

Il successo di un servizio e quindi della sua applicazione comporta inevitabilmente che il team di sviluppo crescerà. Da un lato questo è positivo perché più sviluppatori possono fare di più ma Il problema con i team numerosi è che il carico di comunicazione di un team di dimensioni N elementi è O(N^2). Se il team diventa troppo grande, diventerà inefficiente a causa del carico di comunicazione. Immaginate, ad esempio, di cercare di fare una riunione quotidiana con 20 persone.&#x20;

La soluzione è quella di suddividere un grande team in mini-team più piccoli e coesi, ognuno di questi è composto da non più di 8-12 persone. Ha una obiettivo chiaramente orientato al business: sviluppare e possibilmente operare uno o più servizi che implementano una specifica funzionalità. Il team è cross-funzionale e può sviluppare, testare e distribuire i suoi servizi senza dover comunicare o coordinarsi frequentemente con altre squadre.

La velocità del team suddiviso così è significativamente più elevata rispetto a quella di un singolo grande team. L'architettura a microservizi svolge un ruolo chiave nel permettere ai vari sotto team di essere più autonomi.&#x20;

Ognuno di questi può sviluppare, distribuire e scalare i propri servizi senza doversi necessariamente coordinare con altri sotto team. Inoltre, è molto chiaro chi contattare quando un servizio non sta rispettando i suoi SLA.&#x20;

Si migliora la scalabilità perché è possibile far crescere l'organizzazione aggiungendo altri sotto team. Se un singolo team diventa troppo grande, lo si divide insieme al suo servizio o servizi associati. Poiché i vari team sono scarsamenti accoppiati, si evita l'onere della comunicazione di un grande team. Di conseguenza, è possibile aggiungere persone senza influire sulla produttività.

Utilizzare l'architettura a microservizi con un processo waterfall è come guidare una Ferrari trainata da cavalli: si spreca gran parte dei vantaggi, se si desidera sviluppare un'applicazione con l'architettura a microservizi, è essenziale adottare pratiche di sviluppo e distribuzione agili come Scrum o Kanban e si devono implementare pratiche di CD/CI, che fa parte del DevOps.

## SLIDE 04-02

Possiamo definire la continous delivery nel seguente modo: la capacità di portare in produzione, o nelle mani degli utenti, cambiamenti di ogni tipo, inclusi nuove funzionalità, modifiche alla configurazione, correzioni di bug ed esperimenti, in modo sicuro e rapido in modo sostenibile.&#x20;

Una caratteristica chiave è che il software è sempre pronto per il rilascio e si basa su un alto livello di automazione, compresi i test.&#x20;

Le organizzazioni ad alte prestazioni che praticano CI/CD effettuano più rilasci al giorno in produzione, hanno molto meno interruzioni e si riprendono rapidamente da quelle non previste.o.

Uno degli obiettivi della CI/CD  è rilasciare rapidamente ma in modo affidabile il software. Quattro utili metriche per valutare lo sviluppo del software sono le seguenti:&#x20;

* **Frequenza** di deploy: quanto spesso il software viene rilasciato in produzione&#x20;
* Tempo di **attesa**: il tempo che intercorre dal momento in cui uno sviluppatore effettua una modifica al momento in cui tale modifica viene rilasciata&#x20;
* Tempo medio di **ripristino**: il tempo necessario per ripristinare un problema in produzione&#x20;
* Tasso di **fallimento** delle modifiche: percentuale di modifiche che causano problemi in produzione In un'organizzazione tradizionale, la frequenza di deploy è bassa e il tempo di attesa è alto.

## SLIDE 04-03

L'adozione dell'architettura a microservizi cambia la nostra architettura, l'organizzazione e i processi di sviluppo. Possiamo descrivere il concetto di transizione come un modello di transizione a tre fasi:

* _Conclusione, Perdita e Rilascio:_ Il periodo di disagio emotivo e resistenza quando le persone vengono presentate a un cambiamento che le costringe ad uscire dalla loro zona di comfort. Spesso un po' si rimpiange la perdita del vecchio modo di fare le cose.&#x20;
* _Zona Neutra:_ Lo stadio intermedio tra il vecchio e il nuovo modo di fare le cose, in cui le persone spesso sono confuse. Spesso stanno lottando per imparare il nuovo modo di fare le cose.
* _Nuovo Inizio:_ Lo stadio finale in cui le persone hanno abbracciato entusiasticamente il nuovo modo di fare le cose e stanno iniziando a sperimentare i benefici.

Nel nostro studio di caso, UrbanEats sta sicuramente soffrendo dell'inferno monolitico e ha bisogno di migrare verso un'architettura a microservizi. Deve anche cambiare la propria organizzazione e i processi di sviluppo. Tuttavia, affinché UrbanEats possa realizzare con successo questo obiettivo, deve tenere conto del modello di transizione e considerare le persone.&#x20;
