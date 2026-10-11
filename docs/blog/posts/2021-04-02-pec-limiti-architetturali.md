---
title: "La PEC: progettata male e realizzata peggio"
date: 2021-04-02
categories: [pec, sicurezza]
legacy: true
source: "Commento su un blog, 2 aprile 2021"
---

La PEC rappresenta il tentativo di trasporre la raccomandata con
ricevuta di ritorno dal mondo analogico a quello digitale,
con tutti gli identici problemi (inevitabili, ma conosciuti e accettati)
più una miriade di problemi nuovi.

Ne ho viste di tutti i colori, caselle PEC piene di enti pubblici,
importanti aziende, privati... (ma per quanto tempo sei tenuto a
mantenere i messaggi?):

- Ricevute arrivate, ma PEC sparite nel nulla.
- Gente che pretende di inviare allegati da 200 MB (ogni provider ha
  regole diverse).
- Gente che invia allegati in formati strani e creativi.
  Cosa succede se il destinatario non è in grado di leggerli?

Oltre al problema, noto, ovvero che **non sempre si conosce l'associazione
tra casella di posta e soggetto giuridico che ci sta dietro**.

A livello di sicurezza poi non ci siamo. I server di posta tradizionali
— Postfix, Exim, Dovecot — sono software open source, usati da milioni
di persone nel mondo, aggiornati regolarmente, e comunque presentano
costantemente numerosi bug. Figuriamoci un software proprietario,
chiuso, gestito da pochi fornitori: chi lo sviluppa? Chi lo aggiorna?
Quanti occhi lo guardano? Se i bug si trovano anche nei progetti
open source più grandi e controllati, cosa ci si può aspettare da un
software di nicchia che nessuno può vedere?

Poi certo agli utenti piace, sostanzialmente perché possono inviare
infinite "raccomandate elettroniche" a costo quasi zero ed il
destinatario ha l'obbligo di rispondere.

Purtroppo si tratta di uno strumento che cerca di risolvere
un'esigenza reale, però è progettato male e realizzato peggio.