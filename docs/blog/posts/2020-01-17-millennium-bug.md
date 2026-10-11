---
title: "Millennium Bug? Quale Millennium Bug?"
date: 2020-01-17
categories: [storia, informatica]
legacy: true
source: "Commento su un blog, 17 gennaio 2020"
---

> Commento del 17 gennaio 2020 su un blog, in risposta a una
> discussione sul Millennium Bug e sugli scenari apocalittici del 1999.

Io c'ero, e sono convinto che gli scenari apocalittici che si
sarebbero dovuti verificare esattamente allo scoccare della
mezzanotte fossero in gran parte superstizione.

Poi è assolutamente vero che migliaia di programmi (gestionali,
contabilità, etc.) calcolavano le date con 2 cifre invece di 4 e
che quindi dovevano essere corretti. All'epoca abbiamo lavorato
mesi su questo fronte. Alcuni programmi però avrebbero
dato risultati errati già mesi prima della fatidica presunta fine
del mondo, altri avrebbero dato risultati errati mesi dopo.
Soprattutto, non era necessario aspettare la mezzanotte per saperlo,
si poteva testare tutto agevolmente in anticipo.

(a dire la verità ci sono programmi che ancora oggi fanno confusione
ogni volta che cambia l'anno, e per capire come mai basta guardare
gli snippet di codice che girano per il web e che i programmatori
inesperti copiano ed incollano sui loro progetti: la maggior parte
di questi snippet sono buggati! Anche il calcolo degli anni bisestili
non è sempre corretto, ed anche il cambio ora legale/ora solare ha
spesso effetti deleteri)

Quella notte i colleghi del servizio informativo
avevano una reperibilità speciale, con tanto di radio a pile per
essere raggiungibili anche in caso di blackout. Ricordo che il
pomeriggio di quel fatidico 31 dicembre i colleghi più anziani,
che poi sarebbero rimasti reperibili tutta la notte, erano sollevati
dal fatto che le notizie provenienti dall'Asia fossero positive:
nessun grave problema. Io, che sono malvagio, li ho spaventati
facendo notare come tutti i sistemi unix compresi i nostri dovessero
avere il clock sincronizzato con UTC, per cui in Asia non avevano ancora
raggiunto la "vera" mezzanotte, anzi nella mia malvagità avevo
sganciato una bomba, facendo presente che semmai ci
fosse stato qualche disastro questo si sarebbe verificato
alla mezzanotte UTC in contemporanea su tutti i sistemi
del pianeta. Poi sono andato a festeggiare in montagna :)

*Precisazione tecnica: i sistemi Unix sincronizzano tipicamente
il clock con UTC (Coordinated Universal Time), che per semplicità
potremmo considerare equivalente al fuso orario di Greenwich (GMT),
anche se dal punto di vista scientifico non è la stessa cosa.
Il mio ragionamento scherzoso non era quindi
del tutto sballato: il momento critico era lo stesso per tutti i
sistemi sincronizzati a UTC, indipendentemente dal fuso locale. Ma
nessuno ci aveva pensato.*

Per chiarire meglio il mio pensiero, è vero che di programmi buggati
ce n'erano parecchi ed è stato un bene averli sistemati. Clock
hardware buggati invece ne sono stati trovati? Gli articoli
giornalistici dell'epoca erano più dettati dalla superstizione che
non dalla reale conoscenza dei sistemi.

Infine, il vero Millennium Bug avverrà nel 2038, alle 03:14:07 UTC
del 19 gennaio, e non a mezzanotte come nel 1999. Ed è ancora
troppo sottovalutato.