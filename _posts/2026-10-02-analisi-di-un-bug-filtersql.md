---
layout: post
title: "filtersql v1.2.6: il bug banale che ha rivelato un abisso architetturale"
date: 2026-10-02 22:00:00 +0200
categories: python, filtersql
---

# filtersql v1.2.6: il bug banale che ha rivelato un abisso architetturale

Quando ho pubblicato la versione 1.2.6 di `filtersql` ero convinto di aver raggiunto
un buon standard di sicurezza. Poi, puntuale come un ordigno svizzero, è
saltato fuori un nuovo bug. 

Era la classica svista minuscola, la riga di codice ignorante che guardi e pensi
di risolvere in pochi minuti.

Ma quando ho messo le mani nella correzione mi sono reso conto che la patch banale risolveva
in effetti il bug, ma non il problema che si è rivelato essere più grande.

Il bug non era nella logica: **era architetturale**.

---

### Premessa 1: La giungla dei nomi di colonna nei database reali

Vero che dipende dal DBMS, dalla versione e dalla configurazione, ma in linea generale i database
consentono di definire i nomi delle colonne con grande ed eroica anarchia.
Una colonna può contenere spazi, accenti, caratteri speciali e simboli Unicode.
Si può addirittura quotare il carattere di quoting.

Ed in produzione in effetti non c'è limite all'ineleganza: da colonne
come `"qtà totale"` o `"n° ordine"` che pur non essendo una idea grandiosa sono tutto sommato legittime,
fino a casi limite ma inaspettatamente validi come `"totale  "` (con due spazi in coda, o magari con tab, newlines, ecc.).

Per garantire una quotazione avanzata e sicura senza impazzire, ho scelto di applicare una
semplificazione: rimuovere eventuali spazi bianchi accidentali agli estremi (*trimming*).
La colonna `"totale  "` veniva ripulita in `"totale"`. Questo avrebbe ridotto la
copertura dei casi più assurdi, ma i casi più dignitosi sarebbero rimasti salvi.

---

### Premessa 2: L'incubo dei conflitti di `scope` (Multi-Tenancy)

`filtersql` gestisce il concetto di **`scope`**: una mappa di condizioni fisse (come `tenant_id = 42`)
applicata lato server a tutte le operazioni (`select`, `insert`, `update`, `delete`).
Non sostituisce davvero i permessi del database, ma offre comunque una sua utilità nel blindare le applicazioni web multi-tenant.

Ma cosa succede se un client invia un payload che prova a modificare proprio la colonna riservata allo `scope`?

A parte la rottura di un equilibrio estetico, nelle letture e nelle cancellazioni non si pone davvero un problema, dato che le condizioni finiscono tutte in `AND` e al massimo non esce e non si cancella nulla, ma in scrittura la faccenda si fa interessante:

#### Caso `insert`:
```sql
insert into users (tenant_id, tenant_id, name) values (400, 500, 'Mario');
```
A seconda del DBMS, questa istruzione può assegnare
il primo valore (`400`), prendere l'ultimo (`500`),
comportarsi in modo imprevedibile o fallire miseramente.

#### Caso `update` (molto peggio):
```sql
update users set tenant_id = 500 where tenant_id = 400;
```
Questa singola riga di codice trasferisce uno o più record dal proprio tenant a un altro.
Ed a parte perdere i propri dati che spariscono dal controllo, un utente malevolo potrebbe sfruttare questo comportamento per spostare un account admin dal proprio tenant al tenant
di un nemico per prenderne il controllo.

**La soluzione logica:** il problema sembrava risolvibile
con banale controllo di collisione che ho implementato
facilmente.

Se una colonna appartiene allo `scope`, deve essere vietata nei campi di `insert` o `update`. 

Facile come bere un bicchiere d'acqua passeggiando nel parco.

---

### Il Bug della v1.2.6

Rileggi le due premesse, inserisci un minuscolo errore logico,
ed ecco che l'ingranaggio salta.

Se lo `scope` protegge la colonna `"tenant_id"` e un client invia in scrittura la chiave `"tenant_id "` (con uno spazio in fondo), cosa succede?

1. purtroppo il controllo dei conflitti confrontava immediatamente le due stringhe grezze: `"tenant_id"` contro `"tenant_id "`;
2. le stringhe risultavano **diverse** ed il controllo dava l'avanti popolo;
3. solo successivamente, la funzione di quoting sanificava l'identificatore applicando il `trim()`.
4. risultato: la query veniva compilata ed eseguita modificando proprio `tenant_id`!

Risolvere questo specifico passaggio non era poi complicato, bastava anticipare il `trim()` al momento giusto, prima del controllo di collisione. Il bug sarebbe stato risolto senza tanta fatica. Ma è stato lì che mi si è aperto l'abisso.

---

## Il problema architetturale: Uguaglianza Stringa vs Equivalenza Semantica

Due stringhe identiche sono per forza uguali, il che non è particolarmente sorprendente.

Ma **due stringhe diverse possono avere lo stesso identico significato logico per il database**.

Cosa succede se il server imposta `scope: {"tenant_id": 400}` e il client invia un payload con `values: {"users.tenant_id": 500}`?

Un controllo basato su stringhe non rileva alcun conflitto (`"tenant_id"` != `"users.tenant_id"`). Il sistema genera allegramente questo codice SQL:

```sql
update users
set
  users.tenant_id = 500
where
  tenant_id = 400;
```

Mentre diversi motori, quali PostgreSQL o SQLite, trovavano
questa sintassi indigesta e la rifiutavano giustamente
con un errore, *MySQL* la accettava con grande entusiasmo, eseguendo lo spostamento temuto.

Ma non solo!

Cosa succede se le colonne sono case-insensitive?
Cosa succede se a livello di database è definito qualche alias?
Cosa sarebbe successo con le decomposizioni Unicode (NFC vs NFD), dove la lettera `à` può essere rappresentata da un singolo codice o da due codici separati (a + accento)?

Le stringhe in byte erano diverse, ma per il database puntavano alla stessa identica colonna.

Riconoscere tutte le possibili varianti semantiche che un utente o un database potevano inventarsi per identificare la stessa colonna era una corsa alle armi persa in partenza. Chi sa davvero se due nomi sono equivalenti è il motore del database; ma `filtersql` genera solo stringhe e non è, per filosofia, collegato al database.

---

### La Svolta: L'Asimmetria Consapevole

Per uscirne vittorioso ho capito che non dovevo rendere più "intelligente" il confronto tra stringhe, ma dovevo **cambiare il contratto all'ingresso**.

Ho introdotto un'architettura asimmetrica:

1. **in lettura (`select`, `where`):** Il sistema rimane flessibile.
   Può accettare identificatori qualificati (`users.tenant_id`) o percorsi JSONB (`data->>key`), perché quotati e sanificati separatamente.
2. **in scrittura (`insert`, `update`, `delete`):** nuova regola: tutte le chiavi in `values` o `id` devono superare
   la validazione **`_validate_bare_identifier`**.

Un identificatore di scrittura valido dev'essere un **bare identifier puro**:

* nessuno spazio iniziale o finale (il `trim()` non pulisce più il dato: se c'è uno spazio, la query viene rifiutata con `InvalidIdentifierError`).
* nessun punto (vietata la qualificazione di tabella come `users.tenant_id`).
* nessuna sintassi JSONB o carattere di controllo.
* forma canonica **Unicode NFC obbligatoria**.

Normalizzando sia lo `scope` che i campi di scrittura in NFC e applicando `casefold()`,
l'area di confronto per le collisioni si è ridotta a uno spazio unidimensionale in cui **nessuna ambiguità semantica può più nascondersi**.

> Nota: nella 1.2.8 ho poi esteso la
> stessa regola anche a `_quote()`, così che
> l'incoerenza non possa ripresentarsi in nessun altro punto del codice.

---

### E la Allowed_Columns?

Ci pensavo da tempo, ma avrei dovuto implementarla prima.

Il collision check risolve il caso in cui il client prova a scrivere su una colonna di scope. Ma non risolve il caso in cui il client prova a scrivere o leggere su colonne
pericolose. Per quello serve una whitelist.

In un'applicazione reale (e a maggior ragione in una pipeline con gli LLM), lasciare che un client o un'IA interroghino o filtrino su qualsiasi colonna esista a DB senza filtro è un rischio enorme: rischi di esporre `password_hash`, `credit_card_token` o filtrare su `stipendio_netto`.

Il prossimo passo sarà quindi introdurre la possibilità
di limitare le colonne coinvolte a quanto previsto
nella `allowed_columns`.

In questo modo i ruoli rimangono ben distinti e puliti:

- il developer definisce a mano la whitelist applicativa dei campi esposti per quello specifico endpoint o utente.

- `filtersql` fa il lavoro sporco da compilatore: sanifica l'input, quota gli identificatori, applica lo scope multi-tenant senza possibilità di bypass e difende il database dai DoS.

Tutto questo sarà pubblicato in una delle prossime versioni.

Tanti saluti ed al prossimo bug!

[filtersql.org](https://filtersql.org) — [github.com/fthiella/filtersql](https://github.com/fthiella/filtersql)