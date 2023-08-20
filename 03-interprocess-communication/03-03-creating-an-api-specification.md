---
description: for a messaging-based service API
---

# 03-03-Creating an API specification

## SLIDE 03-03-01

<figure><img src="../.gitbook/assets/Screenshot 2023-08-19 alle 16.51.38.png" alt=""><figcaption></figcaption></figure>

La specifica per un'API asincrona di un servizio deve, come mostrato in figura, specificare i nomi dei canali di messaggi, i tipi di messaggi scambiati su ciascun canale e i loro formati. È anche necessario descrivere il formato dei messaggi utilizzando uno standard come JSON, XML.

Tuttavia, a differenza di REST e Open API, non esiste uno standard ampiamente adottato per documentare i canali e i tipi di messaggi. Invece, è necessario scrivere un documento informale. L'API asincrona di un servizio è composta da operazioni, invocate dai client, ed eventi, pubblicati dai servizi. Sono documentati in modi diversi. Vediamoli uno per uno, iniziando dalle operazioni.

Le operazioni di un servizio possono essere invocate utilizzando uno dei due stili di interazione differenti:&#x20;

* **API di stile richiesta/risposta asincrona**: questa consiste nel canale di messaggi di comando del servizio, nei tipi e formati dei tipi di messaggi di comando che il servizio accetta e nei tipi e formati dei messaggi di risposta inviati dal servizio.
* **API di stile notifica unidirezionale:** questa consiste nel canale di messaggi di comando del servizio e nei tipi e formati dei tipi di messaggi di comando che il servizio accetta. Un servizio potrebbe utilizzare lo stesso canale di richiesta sia per la richiesta/risposta asincrona che per la notifica unidirezionale.

Un servizio può anche pubblicare eventi utilizzando uno stile di interazione _publish/subscribe_. La specifica di questo stile di API consiste nel canale degli eventi e nei tipi e formati dei messaggi di evento che il servizio pubblica sul canale. Il modello di messaggi e canali della messaggistica è un'ottima astrazione e un buon modo per progettare l'API asincrona di un servizio. Tuttavia, per implementare un servizio è necessario scegliere una tecnologia di messaggistica e determinare come implementare il design

## SLIDE 03-03-02

<figure><img src="../.gitbook/assets/Screenshot 2023-08-19 alle 17.01.36.png" alt=""><figcaption></figcaption></figure>

Un'applicazione basata sulla messaggistica di solito utilizza un message broker cioè un servizio di infrastruttura attraverso il quale il servizio comunica. Tuttavia, un'architettura basata su broker non è l'unica architettura di messaggistica possibile. Si può anche utilizzare un'architettura di messaggistica senza broker (brokerless), in cui i servizi comunicano direttamente tra loro. I due approcci, come mostrato in figura, presentano trade-off diversi, ma di solito un'architettura basata su broker è un approccio migliore che è l'approccio su cui ci concentreremo in questo corso ma comunque può essere utile dare un'occhiata veloce all'architettura senza broker, perché potrebbero esserci scenari in cui la trovi utile.

## SLIDE 03-03-03



