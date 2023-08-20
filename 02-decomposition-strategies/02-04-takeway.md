# 🤝 02-04-Takeway

## SLIDE 02-04

L'architettura determina le abilità della nostra applicazione, inclusa la manutenibilità, la testabilità e la capacità di deployment, che influenzano direttamente la velocità di sviluppo.&#x20;

L'architettura a microservizi è uno stile architetturale che conferisce all'applicazione elevata manutenibilità, testabilità e capacità di deployment.&#x20;

I servizi in un'architettura a microservizi sono organizzati attorno a preoccupazioni aziendali, come capacità aziendali o sottodomini, piuttosto che preoccupazioni tecniche.&#x20;

Ci sono due modelli di decomposizione:

* Decomposizione per capacità aziendali, che ha origine nell'architettura aziendale
* Decomposizione per sottodominio, basata su concetti del domain-driven design

È possibile eliminare le classi onnipotenti (god classes), che causano dipendenze complesse che ostacolano la decomposizione, applicando il domain-driven design (DDD) e definendo un modello di dominio separato per ciascun servizio.
