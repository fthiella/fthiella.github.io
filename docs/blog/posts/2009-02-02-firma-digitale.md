---
title: "Firma Digitale"
date: 2009-02-02
categories: [sicurezza, firma-digitale]
legacy: true
---

La firma digitale è un procedimento che, a partire da un documento
informatico e da alcune informazioni associate ad una persona, produce
un nuovo oggetto informatico firmato che attesta la volontà della
persona di sottoscrivere il documento originario.

Il procedimento che viene applicato al documento da firmare si basa
sulla crittografia a chiave asimmetrica, e per avere valore legale
equivalente alla firma autografa deve soddisfare dei particolari
requisiti.

## Crittografia

La crittografia è nata per trasmettere messaggi tra un mittente ed
un destinatario in modo sicuro, utilizzando però un canale di
trasmissione non sicuro. Un sistema di crittografia serve ad
offuscare i messaggi e renderli illeggibili da chiunque non disponga
della chiave per decifrarli.

Nella crittografia a chiave simmetrica la chiave che si utilizza per
cifrare i messaggi è la stessa chiave che serve a decifrarli.

Naturalmente mittente e destinatario hanno la necessità di scambiarsi
in qualche modo la chiave, e per farlo devono utilizzare un canale
di trasmissione sicuro; ma se dispongono di un canale sicuro, per
quale motivo non utilizzano esclusivamente questo canale anche per i
messaggi? Perché il canale sicuro potrebbe essere molto costoso,
oppure potrebbe essere disponibile solamente in intervalli di tempo
limitati (ad esempio a volte ci si deve trovare di persona per
scambiarsi le chiavi).

Quando mittenti e destinatari cominciano a diventare numerosi, oppure
quando non si conoscono di persona, la crittografia a chiave
simmetrica comincia a mostrare i suoi limiti.

## Crittografia Asimmetrica

La crittografia a chiave asimmetrica utilizza invece due chiavi diverse
per le operazioni di cifratura e decifratura, permettendo di
semplificare il compito di distribuzione delle chiavi. Queste due
chiavi sono associate tra di loro e dispongono di una proprietà che
le rende molto utili: tutti i messaggi cifrati con una chiave possono
essere decifrati soltanto con l'altra chiave, e viceversa. Dal punto
di vista logico, queste chiavi funzionano in modo simmetrico.
Asimmetrico è l'utilizzo che se ne fa: poiché da una chiave non è
possibile ricavare la sua chiave associata, è possibile rilasciare
pubblicamente una chiave, la quale prende il nome di chiave pubblica,
mentre l'altra chiave, che viene mantenuta segreta, prende il nome
di chiave privata.

Per inviare un messaggio sicuro ad un determinato destinatario, è
necessario ottenere la sua chiave pubblica e cifrare il messaggio con
tale chiave: solamente il destinatario legittimo è in grado di
decifrarlo, poiché è l'unico che possiede la sua chiave privata.

**Ma tutto questo che cosa c'entra con la firma?**

Applicando lo stesso ragionamento al contrario, se un messaggio è
decifrabile tramite una chiave pubblica abbiamo la garanzia che esso
sia stato cifrato dalla persona a cui è associata tale chiave, poiché
è l'unica che possiede la relativa chiave privata.

## Certificati

Tutto così semplice? Naturalmente no!

Per scrivere un messaggio sicuro ad un destinatario devo per prima
cosa ottenere la sua chiave pubblica; ma come faccio ad essere certo:
1. che il destinatario sia proprio chi mi dice di essere, e 2. che la
chiave pubblica sia proprio la sua?

Lo stesso discorso naturalmente vale per il procedimento inverso: per
essere certo che un messaggio sia stato cifrato da un utente
utilizzando la sua chiave privata, devo essere sicuro di disporre
esattamente della sua chiave pubblica.

Scopo dei certificati digitali è proprio di garantire che una chiave
pubblica sia associata alla vera identità del soggetto che la
rivendica come propria. Un certificato digitale è un documento,
firmato elettronicamente da un'autorità di certificazione fidata, che
associa una chiave pubblica con un'identità.

Ovviamente per verificare se il certificato digitale è valido, è
necessario disporre della chiave pubblica del certificatore: ma gli
enti certificatori fidati sono pochi, e le loro chiavi pubbliche
possono essere distribuite agevolmente tramite canali sicuri (ad
esempio generalmente vengono installate con il sistema operativo).

## Dispositivi sicuri

Affinché il sistema della cifratura o della firma funzioni
correttamente, è indispensabile garantire che la chiave privata sia
mantenuta davvero privata. Se la chiave è memorizzata su di un file
su disco o su di una memoria di massa, c'è sempre il rischio che
venga in mano a terzi non autorizzati al suo utilizzo. Per evitare
questo rischio è necessario utilizzare dei dispositivi di firma
sicuri, dei dispositivi cioè che dispongono di una chiave privata la
quale, una volta generata, non può essere esportata all'esterno.

Una smart card è un dispositivo di questo tipo: al suo interno
contiene infatti una chiave privata che non può essere esportata, e
dispone di un chip che effettua le operazioni crittografiche
utilizzando la chiave memorizzata all'interno.

## Requisiti per la firma digitale

In base alla normativa vigente, la firma forte è l'unica firma che
attribuisce al documento informatico una valenza probatoria, e prevede
la presenza di una Certification Authority iscritta all'albo
dell'AIPA (oggi AgID) e l'utilizzo di un dispositivo di firma
sicuro. La firma digitale è un tipo di firma forte.

>(Nota: la terminologia "firma forte" / "firma debole" è stata
>superata dal regolamento eIDAS e dal CAD. Oggi si parla di firma
>elettronica qualificata, firma digitale, firma elettronica avanzata,
>firma elettronica semplice. I principi tecnici restano gli stessi.)

Una firma che non possiede i requisiti precedenti è detta firma
debole: la valenza probatoria del documento firmato tramite firma
debole è soggetta alla discrezionalità del giudice.

## Impronta del documento

L'impronta è un processo tramite il quale è possibile ottenere da un
oggetto informatico di dimensione qualsiasi una sequenza di bit a
lunghezza fissa (ad esempio 128 o 160 bit nel 2009, oggi 256 bit e
oltre). Visto che l'impronta ha lunghezza fissa mentre l'oggetto di
origine può avere qualsiasi lunghezza, è naturalmente possibile
ottenere delle collisioni, ovvero più oggetti diversi possono avere
la stessa impronta. Un algoritmo che calcola le impronte dei documenti è
sicuro se non permette di ottenere facilmente, data un'impronta, un
oggetto che possa generarla.

Ridurre documenti di qualsiasi dimensione ad una stringa di bit molto
corta permette di effettuare calcoli molto più veloci rispetto a
quelli necessari nel caso in cui l'operazione dovesse effettuarsi sul
documento intero.

## Procedura di generazione di un documento firmato

Il titolare di una coppia di chiavi può firmare digitalmente un
documento adottando la seguente procedura:

- calcola l'impronta del documento
- esegue la cifratura dell'impronta utilizzando la sua chiave privata
- allega il documento in chiaro
- allega l'impronta cifrata del documento
- allega il certificato rilasciato dall'autorità di certificazione

Tutti gli allegati vengono inseriti in una busta formato standard
PKCS#7 (oggi CAdES per i file .p7m, PAdES per i PDF, XAdES per
l'XML) che viene spedita al destinatario.

Il destinatario, per essere certo dell'autenticità e dell'integrità
del messaggio arrivato, effettua la seguente procedura di verifica:

- accede al certificato del mittente, contenuto nella busta
- verifica tramite l'autorità di certificazione che il certificato
  non sia stato revocato o sospeso
- decifra l'impronta del documento con la chiave pubblica del mittente
- calcola l'impronta del documento ricevuto

Se l'impronta calcolata equivale all'impronta decifrata, il documento
non è stato manomesso ed è corrispondente a quello che il mittente ha
firmato.

## Vulnerabilità

Un dispositivo di firma quale la smart card è un dispositivo sicuro.
Il PC che calcola l'impronta del documento invece è potenzialmente
insicuro: come si può essere certi che l'impronta firmata dalla smart
card sia esattamente l'impronta del documento visualizzato ed
effettivamente scelto dall'utente?

>(Nota: oggi il problema si è spostato sulla firma remota e sui
>servizi cloud, dove la chiave privata è su un HSM del provider. Il
>principio resta lo stesso: il punto debole non è il dispositivo di
>firma, ma tutto ciò che sta intorno.)

Altri problemi potrebbero riguardare documenti non statici che
contengono istruzioni o codici eseguibili, ma tali documenti sono
esclusi dalla validità della firma secondo la normativa vigente.

Infine è possibile, almeno teoricamente, creare documenti ambigui che
sono validi in più formati. Si suppone però che anche questi documenti
siano esclusi dalla validità della firma.