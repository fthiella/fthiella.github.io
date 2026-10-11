---
title: "Nomi di colonna con spazi: fastidio estetico o problema di sicurezza?"
date: 2021-11-29
categories: [sql, database]
legacy: true
source: "Commento su un blog"
---

> Commento su un blog, in risposta a una discussione sui nomi
> di colonna non convenzionali nei database.

La maggior parte dei sistemi di database lo permette, qualsiasi
carattere può essere utilizzato per il nome della colonna, ma non
è necessariamente una buona idea.

Generalmente mi danno un senso di poca professionalità, e se
utilizzate con sistemi o framework poco professionali il disastro
è servito (es. export su CSV senza utilizzare librerie adeguate
per il CSV).

Se il sistema è solido però nessun problema, solo il fastidio
estetico per delle query "brutte" e poco eleganti che necessitano
di caratteri di `quoting` non standard tra i diversi db.

Anche se devo ammettere che di query brutte ce ne sono tante,
e non è sempre il problema degli spazi...

---

>Anni dopo, lavorando a `filtersql`, mi sono trovato ad affrontare
>lo stesso problema da un'altra angolazione: non più come "fastidio
>estetico", ma come vero e proprio problema di sicurezza.

