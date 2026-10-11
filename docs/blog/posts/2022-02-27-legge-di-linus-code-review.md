---
title: "\"Dato un sufficiente numero di occhi, i bug tendono a zero\" non funziona"
date: 2022-02-27
categories: [programmazione, opensource]
legacy: true
source: "Commento su un blog"
---

> Commento su un blog del 2022, in risposta a una discussione sulla
> Legge di Linus e sul ruolo del code review nell'open source.

L'idea che *"dato un sufficiente numero di occhi, i bug tendono a zero"*
non funziona.

Trovare bug è un lavoro a tempo pieno, e spesso complesso. Non è che
ti metti a leggere il codice sorgente sotto l'ombrellone e sei in grado
di scoprire e risolvere i bug. Qualcuno forse, ma solo se il bug è
banale o il codice è stato scritto da un principiante.

## Il problema della Legge di Linus

La frase di Eric Raymond — *"given enough eyeballs, all bugs are shallow"* —
è una semplificazione affascinante ma fuorviante. Funziona per i bug
evidenti, quelli che si vedono leggendo il codice con attenzione. Non
funziona per i bug che contano davvero.

I bug seri sono spesso:

- **di concorrenza**, si manifestano solo in determinate condizioni
  di timing, e non si vedono leggendo il codice;
- **di stato**, dipendono da una sequenza specifica di operazioni, che
  il lettore non può ricostruire senza eseguire il programma;
- **di casi limite**, coinvolgono input che il lettore non ha motivo
  di considerare, perché non sono nel percorso principale;
- **di integrazione**, emergono solo quando il codice interagisce con
  altri componenti, versioni diverse, ambienti diversi.

Questi bug non si trovano "guardando". Si trovano testando, strumentando,
eseguendo. E il code review, da solo, non basta.

## Il caso del codice offuscato

Se poi qualcuno vuole **intenzionalmente** offuscare del codice e
nascondere delle funzionalità, il discorso cambia completamente.

Il code review diventa inefficace: il codice è scritto apposta per
sembrare innocuo. Servono analisi dinamiche, reverse engineering,
strumenti di tracing. E anche con tutto questo, dipende caso per caso:
potrebbe essere molto difficile da scoprire.

Questo vale sia per il codice open source che per quello proprietario.
Anzi, nel caso del codice proprietario la situazione è peggiore,
perché non hai nemmeno la possibilità di leggere il sorgente.

## Conclusione

La Legge di Linus ha un fondo di verità — più occhi possono trovare più
bug — ma è una verità statistica, non una garanzia. E vale solo per i
bug che si possono trovare leggendo. Gli altri richiedono lavoro,
strumenti, e tempo.

Trovare bug è un mestiere. Non è una passeggiata sotto l'ombrellone.