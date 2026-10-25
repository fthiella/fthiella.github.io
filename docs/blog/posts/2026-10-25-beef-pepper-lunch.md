---
layout: post
title: "Il colloquio per la startup da 1 miliardo di dollari, il direttore chiuso in magazzino e un Beef Pepper Rice"
date: 2026-10-25
categories:
  - ingegneria-software
  - carriere
  - python
  - filtersql
---

Nel 2018 Honestbee era una delle startup del Sud-Est asiatico più promettenti,
era valutata quasi un miliardo di dollari ed era pronta a rivoluzionare la spesa
a domicilio. Non li conoscevo, ma i miei parenti a Singapore mi hanno confermato
di aver usufruito dei loro servizi. Ho scoperto dopo che in quel periodo assumevano
a ritmi folli, pagavano bene, e cercavano talenti in tutto il mondo.

Una mattina ricevo questa mail:

> "My name is Joey and I am the Talent Acquisition Partner here in honestbee.
> I came across your profile on Stack Overflow and noticed that you are currently
> looking for new opportunities. [...] We are currently hiring a Senior Data Engineer,
> will you be interested to have a chat with me to find out more?"

Dopo qualche giorno abbiamo fissato una prima call. Non era un colloquio tecnico:
era uno screening iniziale con una recruiter. Lei non era un tecnico, doveva solo
capire se c'era interesse reciproco, se la mia esperienza si sposava con i loro
bisogni e se l'idea di lavorare dall'altra parte del mondo avesse senso per me.

Il fatto che vivessi in Italia per loro non era un problema, erano abituati
a gestire trasferimenti, ma il fatto che conoscessi davvero Singapore era
considerato un vantaggio. Avevo parenti lì e mia moglie aveva lavorato
prima a Marina Bay Sands e poi in diversi negozi di abbigliamento
ed accessori ad Orchard Road e a Changi Airport.

Durante la discussione, forse per rompere il ghiaccio, mi ha chiesto
quale fosse il mio piatto preferito di Singapore. Avrei potuto dire "Chili Crab"
o "Hainanese Chicken Rice", ma ho risposto "Beef Pepper Rice"
di Pepper Lunch. Lei è scoppiata a ridere, dato che Pepper Lunch è una catena
giapponese e non è un piatto tipico di Singapore, ma in effetti è il cibo
che si mangia davvero quando lì ci si vive.

Comunque il primo screening lo avevo passato e dopo qualche giorno mi
ha inviato un take-home test.

Un take-home test non è un esame a crocette: è un problema di programmazione
vero, spesso basato su un caso reale dell'azienda, che ti viene dato da
risolvere a casa con calma, senza il timer di un colloquio. È un ottimo
segnale, perché capovolge la logica della selezione: non stai cercando
di indovinare la risposta che l'intervistatore si aspetta, stai dimostrando
come lavori davvero, con i tuoi tempi, i tuoi strumenti e il tuo modo
di documentare.

Il famoso Joel Spolsky, tra l'altro proprio uno dei fondatori della
piattaforma Stack Overflow, nel suo celebre "Joel Test", lo mette nero
su bianco con la domanda numero 11 "Do new candidates write code during
their interview?".

Se un'azienda ti assume senza farti scrivere codice, ti sta assumendo
sulla fiducia. Se ti fa un take-home test, invece, vuole vedere il tuo
codice reale. È la differenza tra un colloquio in cui reciti e uno in cui
lavori per davvero.

## Job Description e test

Poco dopo, Joey mi ha risposto allegandomi la Job Description e il test.

> "Hi Federico, Thank you for your CV and it was pleasure talking to you earlier.
> Please find attached the JD and the technical test for your review and attempt.
> You have 7 days to complete the test."

La Job Description era impressionante. Cercavano un Senior Data Engineer che
gestisse terabyte di dati, che conoscesse Kafka, Kubernetes, Redshift, GCP, che
costruisse pipeline ETL in tempo reale con Airflow. Era un tipo di annuncio che incuteva
timore, ma in fondo mi avevano cercato loro.

Il test, invece, era un problema di programmazione molto più concreto e pratico.
Chiedevano esplicitamente di scrivere delle web API in Python che sfruttassero
le API pubbliche di Dota2, per risolvere tre richieste specifiche:

- dato un elenco di giocatori (tramite ID o username), restituire una classifica
  basata sul win rate nel tempo (ultima settimana, mese, ecc.);
- confrontare due giocatori;
- suggerire un eroe a un giocatore in base ai suoi dati storici.

Tra le note c'era questo, che ho molto apprezzato:
"We are looking for elements of good style and creativity, so go for elegant code
as well as originality." Cercavano codice non solo funzionante,
non solo veloce: creativo, con un buon stile, pulito, originale. Cercavano
codice elegante.

Peccato che non avessi quasi mai scritto codice Python, e che non avessi la
minima idea di che cosa fosse Dota2.

## Il test (e l'arte di ammettere di non sapere)

Dota2, a quanto pare, era un famoso videogioco competitivo online in cui
due squadre di cinque giocatori si sfidano controllando dei personaggi
chiamati "eroi". Era un fenomeno di massa, con tornei da milioni di dollari.
Io non ne avevo la minima idea, e non ero intenzionato ad avercela,
ma fortunatamente le statistiche delle partite erano interrogabili via API web.

Non mi interessava capire le strategie di gioco, dovevo solo capire
la struttura dei dati: cosa fosse un "match", come si calcolava una vittoria,
e come incrociare l'ID del giocatore con i suoi eroi preferiti, studiando
la documentazione delle API di OpenDota durante la notte.

E per quanto riguarda Python, avrei potuto arrendermi, invece mi sono
rimboccato le maniche e ho cercato di capire la logica, partendo da ciò
che sapevo già - principalmente SQL e Perl.

Avevo sette giorni di tempo. Non c'erano vincoli specifici, ma dopo aver
ricercato e provato diverse librerie ho scelto la strada della semplicità:
Flask come framework Python minimalista, e PythonAnywhere per pubblicare
l'applicazione in pochi minuti, senza dover configurare server complessi.

Volevo che chi leggeva il codice capisse subito cosa faceva. Ho diviso
il codice in due file: `flask_dota2_app.py` per gestire le richieste web,
e `dota2utils.py` per la logica di business.

Mentre su Python improvvisavo, ho fatto leva su ciò che padroneggiavo da anni: SQL.
Per suggerire l'eroe ho scritto una query con un paio di CTE per calcolare
la media delle partite giocate dal profilo ed estrarre solo i personaggi usati
meno della media: l'idea non era proporre l'eroe preferito né uno sconosciuto,
ma consigliare un eroe già provato su cui fare pratica.

Naturalmente ho dovuto prendere delle scorciatoie, che però ho cercato
di documentare. Per passare l'ID del giocatore a quella query SQL eseguita
tramite l'API Explorer di OpenDota, nel codice ho scritto:

```python
# substitute the sql placeholder with the account_id - not usually the
# safest way to run a query but on this particular context it can be fine
query = queries['recommendations'].format(account_id)
```

Avevo ragione e non era il modo più sicuro per eseguire una query,
ma a 24 ore dalla scadenza, con sette giorni per imparare Flask e
l'API di Dota2, `.format()` andava benissimo. Ho pensato però che il
commento fosse apprezzato: faccio cose sbagliate, lo so, ma per lo
meno so di farle.

(Anni dopo, quasi a voler chiudere quel cerchio, ho scritto una libreria Python,
[filtersql](https://filtersql.org), proprio per prevenire SQL injection con
query dinamiche e risolvere quel problema).

Per consegnare il test, oltre al codice Python ed SQL, ho preparato
un report PDF ben impaginato usando StackEdit e Markdown. Nel documento ho spiegato
le mie scelte senza nascondere nulla: ho chiarito perché avevo ignorato
la ricerca per username (su Steam i nomi utente non sono univoci e avrebbero
introdotto ambiguità nei dati), ho ammesso candidamente che l'endpoint
di comparazione tra due giocatori era il meno elegante del progetto, e ho fatto
presente che i test sui filtri temporali li avevo fatti a mezzanotte senza poter
verificare il reale fuso orario dei server di OpenDota.

## Il direttore chiuso in magazzino

Qualche giorno dopo l'invio del test, ricevo questa mail:

> Joey: "Hi Federico, I am very pleased to inform you that we have received
> good feedback for your test and would like to move you to a video interview
> with our Head of Data Science/Data Engineering. He is available on 5th July
> at 8am or 10th July at 8am your time. Please let me know which works best for you."

Insomma, avevo superato il test!

Non era un codice perfetto (del resto, avevo imparato Python e Dota2 da sette giorni),
ma credo che abbiano apprezzato il metodo. Hanno visto che avevo consegnato in tempo,
che il report era ben impaginato, ordinato e leggibile, e che avevo documentato
onestamente le mie scelte. Ho ammesso le scorciatoie, ho evidenziato i possibili
problemi di sicurezza e per loro queste note valevano più di un'architettura
perfetta ma incomprensibile.

Fissiamo il colloquio per la mattina successiva, ore 8 del mattino (ora italiana).

Mi connetto alla stanza Zoom, aspetto, aspetto... ma non c'è nessuno. Che sia
la classica strategia di mettere alla prova i candidati da parte delle aziende
tossiche?

Scrivo a Joey chiedendo se è tutto a posto. Mi risponde che l'intervista per lei era
confermata, ma che verificava subito e mi faceva sapere.

Dopo 15 minuti mi scrive dicendomi di aver trovato il direttore. Era letteralmente
chiuso dentro al magazzino e non riusciva ad uscire. L'intervista doveva essere rimandata.

In pieno stile startup iper-caotica, l'Head of Data Science/Data Engineering era rimasto
intrappolato nel magazzino fisico perché dovevano lanciare un nuovo "grande progetto".

(si trattava di un colossale supermercato/magazzino automatizzato e senza casse
di 6.000 metri quadri, inaugurato l'ottobre successivo)

## L'intervista

Qualche giorno dopo, la call è finalmente avvenuta.

Non ricordo esattamente di cosa abbiamo parlato a livello tecnico, ma ricordo
benissimo la spiegazione dell'organizzazione aziendale. Il team era sterminato:
c'erano decine di data engineer dedicati esclusivamente ai transactional data.
Poi c'era un team di Business Intelligence, il cui compito era fornire report
interni ma anche agli investitori. Infine, c'era un altro gruppo che
raccoglieva i dati applicativi per monitorare in tempo reale il corretto
funzionamento della piattaforma.

Lui era tedesco ma in questo caso ho evitato battute sul cibo.

Mi ha parlato del fatto che cercavano qualcuno che sviluppasse e gestisse
le data pipelines orchestrate da DAG Airflow. Gli ho detto che di dati me
ne intendevo, che non conoscevo quella specifica tecnologia, ma che non
avevo paura di imparare. Ed era stato felice di sentirlo. Mi ha raccontato
che in quei giorni stavano intervistando per la stessa posizione
anche un DBA Oracle. Cercavano talenti ma non avevano le idee ben chiare,
ho pensato.

Gli uffici erano ricavati all'interno di un ex centro commerciale, e con un certo
orgoglio il direttore mi raccontava che non avevano nessun data center fisico,
nulla di nulla: tutto girava su nel cloud sfruttando i servizi AWS.

Era il 2018 e sembrava di stare nella Silicon Valley; guardando indietro
fa sorridere il contrasto tra un'architettura interamente scalata su cloud AWS
e l'Head of Data Science/Data Engineering letteralmente intrappolato in una stanza fisica.

## L'esito

Qualche giorno dopo, mi è arrivata la mail con l'esito. La call era andata tutto
sommato bene, ma hanno ritenuto di non procedere a causa di un "misfit in terms of
technology expectation", cosa che in effetti si sapeva fin da subito.

Col senno di poi, è stata una grande fortuna. Tra il 2019 e 2020 l'azienda
è crollata, licenziamenti di massa, CEO dimissionario, bancarotta e
mille scandali.

Ma anche senza il senno di poi, credo sia stata una bella esperienza perché
comunque, per una settimana, ho parlato con il capo di una delle aziende
più moderne, tecnologicamente all'avanguardia e promettenti del Sud-Est asiatico.
Mi hanno fatto scrivere codice, hanno letto ed analizzato il mio report,
hanno speso tempo per capire come ragionavo. Non mi hanno scelto, pazienza,
ma il fatto stesso di essere arrivato fin lì, partendo da Stack Overflow,
è stata una bella soddisfazione.