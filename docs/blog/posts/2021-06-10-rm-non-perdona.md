---
title: "Quando cd fallisce, rm non perdona"
date: 2021-06-10
categories: [shell, sicurezza]
legacy: true
source: "Commento su un blog"
---

> Commento su un blog, in risposta a una discussione su
> script schedulati e cattive pratiche in shell scripting.

Ho visto di peggio (ma non sono stato io!), ad esempio il seguente
task schedulato - semplifico un po':
```bash
    cd /var/programma/temp
    rm -rf *
```
Nessun errore, tutto corretto no? Tutto funziona alla perfezione
per diverso tempo.

Sennonché un giorno il disco dove si trova la cartella temp va
in fault, ma niente di grave: alla fine è solo una cartella temp...

...poi però parte il task schedulato, il comando cd va in errore
ma lo script non intercetta l'errore e continua...

---

*Nota tecnica: il `rm -rf *` viene eseguito nella directory corrente.
In cron, la directory corrente è la home dell'utente (o `/` se lo
script gira come root). Il risultato è che lo script cancella tutto
quello che trova lì, non quello che credeva di cancellare.*

*Due caratteri fanno la differenza:*
```bash
    cd /var/programma/temp && rm -rf *
```
*L'operatore `&&` esegue il secondo comando solo se il primo è
andato a buon fine. Se `cd` fallisce, `rm` non viene eseguito.*