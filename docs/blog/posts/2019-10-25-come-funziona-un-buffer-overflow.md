---
title: "Come funziona un buffer overflow"
date: 2019-10-25
categories: [sicurezza, programmazione, asm]
legacy: true
source: "Commenti su un blog, 25 ottobre 2019"
---

> Commenti del 25 ottobre 2019 su un blog, in una discussione sulla
> sicurezza dei file audio e sui malware. Li ho riorganizzati in un
> unico testo.

Il buffer overflow è stato uno dei vettori di attacco più comuni, e
non è che sia del tutto sparito. I vari sistemi operativi recenti
offrono protezioni aggiuntive che, in combinazione con funzionalità
hardware, rendono sempre più difficile la cosa. Ma vale la pena
capire come funziona.

## Un errore classico

Storicamente funzionava così:

- un programmatore dichiarava un buffer di una certa dimensione, es.
  80 caratteri
- un attaccante malevolo passava in input al programma un parametro
  troppo lungo, es. 800 caratteri
- il programma, causa bug, non applicava tutti i controlli necessari
  e copiava l'input da 800 caratteri sul buffer da 80
- i caratteri eccedenti andavano a sovrascrivere aree di memoria non
  previste, che con un po' di fortuna potevano essere associate ad
  esempio ad altre variabili - ecco che con questo trucco potremmo
  andare a cambiare dei valori cui non dovremmo avere accesso, e
  quindi cambiare le logiche di esecuzione del programma
- meglio ancora, le aree di memoria non previste potevano essere
  associate allo stack. Quando richiamo una funzione (call) viene
  inserita nello stack la posizione dell'istruzione che segue la call,
  in modo da ritornare lì (ret) una volta terminata la funzione. Ecco che
  modificando i valori inseriti nello stack, tra cui l'elenco delle
  istruzioni cui riprendere l'esecuzione terminata una funzione,
  possiamo dire ad un programma di eseguire le istruzioni di una
  qualsiasi area di memoria, magari proprio all'interno di questi
  800 caratteri.

Ecco, quanto descritto è un metodo "classico" causato da un errore
"classico": buffer troppo corto, mancanza di adeguati controlli.

## Perché funziona

Devi ragionare in termini di linguaggio macchina, e devi pensare a
come il tuo codice C o C++ è compilato e a come viene poi eseguito.
L'attacco che ho descritto è - era? - molto comune, ed è stato usato
con successo diverse volte: la maggior parte dei worm che si
diffondevano in rete sfruttavano proprio questa tecnica.

L'attacco consiste nell'inviare una stringa molto più lunga di quanto
il programma ha previsto, che contiene:

1. il codice macchina malevolo da eseguire
2. l'indirizzo del codice macchina al punto 1

In particolare, l'indirizzo al punto 2 oltre a dover essere calcolato
precisamente, è indispensabile che sia perfettamente allineato, in
modo che si sovrapponga alla struttura dati che contiene le
"informazioni di controllo dell'esecuzione" del programma.

Il buffer può trovarsi dove si vuole, ma la sua posizione relativa
potrebbe essere sempre la stessa, ecco perché il sistema funziona.
Vari sistemi, come l'Address Space Layout Randomization, cercano
appunto di rendere le posizioni relative non prevedibili, per cui
questo tipo di attacco è molto più difficile (ma non impossibile).

## Un crash non è la fine

In sostanza, è vero che la maggior parte delle volte un bug di questo
tipo produce un errore di segmentation fault, ma inviando dei dati
malvagi, preparati con grande accuratezza, è possibile far eseguire
al programma qualche cosa di non previsto.

Non sempre questi bug sono sfruttabili (exploitabili): si potrebbe
riuscire a mandare in crash il programma ma potrebbe non essere
possibile controllarne l'esecuzione. Comunque un bel crash promette
sempre bene.

Un errore di segmentation fault è un punto di partenza, da solo
serve poco se non per mettere in piedi attacchi DoS (denial of
service), ma significa che il programma ha qualche bug che potrebbe
essere sfruttato per un attacco.

## Le protezioni moderne

Questo metodo è sempre più difficile da applicare:

- i compilatori potrebbero segnalare dei warning
- gli strumenti di analisi del codice dovrebbero essere in grado di
  segnalare probabili bug di questo tipo
- i sistemi operativi nonché i processori offrono protezioni
  aggiuntive, come l'ASLR, il DEP, lo stack canary

Ci sono comunque anche diverse strategie di attacco.

## Nota sui file audio e sulle immagini

Un'ultima precisazione: la frase "non è possibile infettarsi
semplicemente scaricando e ascoltando un file audio WAV" andrebbe
esplicitata meglio.

Nemmeno con i file JPG, HTML, o Flash sarebbe teoricamente stato
possibile infettarsi, nemmeno con le e-mail (sono solo dei banali
testi!) o con le chiavette USB, eppure causa programmi vulnerabili
è successo più di qualche volta.