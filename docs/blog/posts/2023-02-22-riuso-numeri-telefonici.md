---
title: "Numeri di telefono riciclati: una falla di sicurezza?"
date: 2023-02-22
categories: [sicurezza]
legacy: true
source: "Commento su un blog, febbraio 2023"
---

> Commento del febbraio 2023 su un blog, in risposta a un articolo
> che raccontava il caso di un utente che aveva ereditato per caso
> l'account WhatsApp di un'altra persona a causa del riutilizzo
> del numero di telefono.

Quando mi è stato assegnato il primo cellulare di lavoro, molti
anni fa, ho cominciato a ricevere SMS e telefonate da amici che
non sapevo di avere, che mi invitavano a feste ed eventi vari.

Una signora, che si spacciava per mia nonna, mi chiamava spesso
piangendo per dirmi che, anche se mi rifiutavo di riconoscerla,
per lo meno avevo ripreso a risponderle al telefono.

Infine un ragazzo mi accusava di avergli rubato cellulare e
numero di telefono.

Gli operatori telefonici riutilizzano sempre i numeri di telefono,
nel mio caso devono averlo fatto un po' troppo in fretta.

## Il problema

Per quanto riguarda questa falla di sicurezza, mi sembra talmente
scontata che mi stupisco che sia stata sottovalutata: nella
maggior parte dei servizi di messaggistica il numero di telefono
viene verificato **solo in fase di attivazione** (tramite chiamata
o SMS) e poi l'applicazione viaggia su binari diversi rispetto
alla SIM.

Puoi cambiare SIM, numero di telefono, puoi anche togliere la SIM,
etc. ma a WhatsApp la cosa non interessa. Se un numero viene
riassegnato, WhatsApp come fa a sapere se sei il vecchio utente
di prima che ha cambiato telefono o un nuovo assegnatario del numero?

Le policy di cancellazione automatica per inattività (come i canonici 45 giorni previsti da alcune piattaforme) mitigano solo in parte il fenomeno, ma restano una toppa parziale.

## Lo stesso problema con i domini

Ah, il problema succede anche con i domini internet. Se un dominio
viene riassegnato, cosa che avviene per diverse ragioni non sempre
volontarie, il nuovo proprietario può non solo leggere tutta la
tua (nuova) corrispondenza, ma anche prendere accesso a tutti gli
account esterni se registrati con una casella di posta legata al
dominio "perduto".

Il pattern è sempre lo stesso: una risorsa viene riassegnata,
ma i servizi che ci si appoggiano non lo sanno.