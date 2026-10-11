---
title: "L'IP era vero, il mittente no"
date: 2018-10-26
categories: [privacy, web]
legacy: true
source: "Commenti su un blog, 26 ottobre 2018"
---

> Commenti del 26 ottobre 2018 su un blog, in una discussione sul
> tracciamento degli utenti. Li ho riorganizzati in un unico testo,
> tagliando le ripetizioni.

Il discorso è complesso, e forse si fa confusione perché si mescolano
due problemi diversi. Io li vedo così:

- molte aziende su internet trattano nostri dati personali che noi
  non sappiamo di aver dato loro (e con questi dati ci campano, ecco
  perché la professione di data scientist è sempre più richiesta e
  sempre più pagata...)
- molte aziende non sono nemmeno in grado di trattare e proteggere i
  dati che invece abbiamo affidato loro

Vale anche per i dati sanitari, ad esempio... purtroppo privacy,
GDPR etc. vengono visti come adempimenti burocratici da applicare
alla lettera senza capire che cosa si sta facendo e perché, col
risultato che si possono anche avere sistemi che sulla carta sono
compliant ma in pratica fanno acqua da tutte le parti...

---

Un esempio che mi è capitato in prima persona molti anni fa, e che
forse può far capire meglio il senso del mio discorso. Era il 1999,
e via mail arrivava un virus. A me e a molti partecipanti di un
newsgroup. Il mittente era anonimo, ma l'IP era vero.

A quel tempo non si discuteva su Disqus o sui forum, ma sui newsgroup.
Un newsgroup era un elenco di post, cioè nuove discussioni o risposte
ad altre discussioni. Ad ogni post era associato il suo autore - anche
fittizio, non serviva registrazione - e di ogni post era possibile
ottenere l'indirizzo IP da cui era stato inviato.

Volevo capire chi era il vero mittente a partire dall'IP, ma come fare
senza scomodare il provider?

Ho pensato di scaricare qualche migliaio di discussioni dai newsgroup,
e le ho caricate su di un database con IP, data ed ora. Ho confrontato
gli IP delle mail anonime con gli IP nel database, e la cosa ha
funzionato perfettamente!

E ho trovato diverse cose interessanti: utenti diversi che
condividevano lo stesso IP, un utente che si spacciava per utenti
diversi, e l'autore inconsapevole della mail - il vero autore non
sapeva di aver mandato il virus, perché il suo PC era infetto.

Corollario: discutere sui newsgroup rendeva pubblicamente associabile
il tuo IP al tuo nickname!

Oggi i newsgroup non vanno più di moda, l'IP potrebbe non voler dire
molto e gli IP delle discussioni in ogni caso non sono più pubblici
(ma il gestore del forum li ha eccome) e solo una piccola percentuale
di utenti partecipa alle discussioni - ci sono però altre soluzioni
più moderne.

Non sto dicendo che tutti commettano illeciti, sto solo dicendo di
fare attenzione che i fornitori di servizi dispongono di molti più
dati su di noi rispetto a quelli che noi pensiamo di dare, e i dati
sono come i maiali, non si butta via niente!

Questo lavoro è probabilmente la cosa più importante che ho fatto
in quegli anni, perché dimostra di aver capito come funziona davvero
il mondo: le tracce digitali raccontano una verità che spesso non
coincide con l'intenzione di chi le ha lasciate.

---

Il punto però non è solo Facebook o Google presi singolarmente.
Il punto è come funziona il web nel suo insieme.

Oggi i newsgroup sono stati sostituiti,
ma il meccanismo di correlazione non è scomparso:
si è semplicemente spostato sui circuiti pubblicitari e
sui widget di terze parti.

Se un utente visita in sequenza siti diversi che condividono
lo stesso sistema di commenti, lo stesso fornitore di statistiche
o la medesima rete di banner pubblicitari, il fornitore
terzo dispone di tutti gli elementi per ricostruire la
cronologia di navigazione, anche in assenza di un account
registrato o di un'autenticazione esplicita.

Le tecnologie cambiano, ma la dinamica resta la stessa:
le informazioni che condividiamo sono molte più
di quelle che pensiamo di trasmettere.