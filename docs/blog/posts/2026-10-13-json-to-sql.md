---
title: "Dall'eleganza del JSON a SQL: generare query sicure e parametrizzate in Python"
date: 2026-10-13
categories:
  - sql
  - python
  - filtersql
  - json
  - sicurezza
---
Capita spesso di dover trasformare dei filtri selezionati dall'utente in query SQL. Il caso tipico è una interfaccia web dove l'utente può selezionare prodotti per prezzo, selezionare i clienti dal nome, o visualizzare le fatture dal loro stato.

L'approccio più semplice è creare stringhe SQL dinamiche, a partire dai dati inseriti dall'utente. Tuttavia mescolare la struttura di una query con i valori inseriti dall'utente diventa via via più complesso e difficile da manutenere. Inoltre, la probabilità di introdurre problemi di sicurezza cresce.

Anni fa ho provato a costruire un framework web stile *Microsoft Access* in *Perl* e *HTML::Mason* con form, dropdown dipendenti, tabelle e sottotabelle master-detail, tutto generato da configurazioni JSON dichiarative.

L'idea era senza dubbio buona e i risultati incoraggianti, ma il progetto era troppo esteso per un singolo sviluppatore. Ho deciso quindi di mettere da parte il frontend e salvare la componente architetturale più solida: il compilatore JSON-to-SQL del backend. L'ho convertito in Python e ho continuato a utilizzarlo, migliorandolo per anni. Da qui è nato `filtersql`.

Recentemente, mentre costruivo un RAG aziendale custom, mi sono reso conto che gli LLM, pur essendo in grado di generare direttamente codice SQL, lo fanno in modo impreciso ed insicuro. Al contrario, sono perfetti per generare JSON dichiarativo. Il mio vecchio motore era esattamente l'anello mancante per permettere all'AI di interrogare i database relazionali in modo elegante e sicuro.

## Descrivi la query come JSON

Supponiamo di voler estrarre i prodotti che costano almeno 100, che fanno parte della categoria "libri", ordinati dal prezzo crescente.

Invece di creare il codice SQL a mano, possiamo esprimere la richiesta come un oggetto JSON:

```json
{
  "action": "select",
  "source": "products",
  "columns": ["id", "name", "price"],
  "filters": [
    {
      "field": "price",
      "operator": ">=",
      "value": 100
    },
    {
      "field": "category",
      "operator": "=",
      "value": "books"
    }
  ],
  "order": [
    {"field": "price", "order": "asc"}
  ],
  "limit": {
    "start": 0,
    "length": 10
  }
}
```
Questo JSON descrive esattamente *ciò che vogliamo*, senza necessità di includere alcun codice SQL tra i valori.

## Genera il codice SQL parametrizzato con Python

Con `filtersql` tale payload può essere convertito in SQL e in una lista separata di parametri:
```python
from filtersql import filtersql

payload = {
    "action": "select",
    "source": "products",
    "columns": ["id", "name", "price"],
    "filters": [
        {"field": "price", "operator": ">=", "value": 100},
        {"field": "category", "operator": "=", "value": "books"},
    ],
    "order": [{"field": "price", "order": "asc"}],
    "limit": {"start": 0, "length": 10},
}

sql, params = filtersql(payload, dbms="sqlite")

print(sql)
print(params)
```
il codice SQL generato sarà quindi:
```sql
select
  "id",
  "name",
  "price"
from
  "products"
where
  ("price" >= ? and "category" = ?)
order by
  "price" asc
limit 0, 10
```
I parametri sono restituiti separatamente:
```python
[100, "books"]
```
Codice SQL e parametri possono quindi essere trasmessi al driver del database:
```python
cursor.execute(sql, params)
```
L'aspetto fondamentale è che i valori dei filtri non vengono mai concatenati al testo della query, ma associati in modo nativo per prevenire le SQL injection. La sintassi generata per i placeholders (`?`, `%s`, ecc.) dipende dal motore del database selezionato.

## Perché separare l'intento dalla query SQL?

La separazione netta tra l'intento di business e l'istruzione SQL vera e propria è un pattern solido quando i filtri provengono da input non fidati: un frontend web, un'API REST esposta al pubblico, o l'output strutturato di un LLM.

Delegare la generazione a un layer intermedio garantisce che il payload possa contenere solo ciò che hai esplicitamente autorizzato, permettendoti persino di validare a priori le colonne interrogabili tramite allowlist rigide. Lo stesso identico payload JSON può inoltre essere riciclato su database diversi, perché sarà filtersql a preoccuparsi di tradurlo nel dialetto corretto.

Questa libreria non vuole sostituire un ORM o un query builder completo: fa una cosa sola, convertire input strutturati in filtri SQL inattaccabili, e cerca di farla bene.

È un approccio pulito ed elegante. Trovate il codice, la documentazione e gli esempi su [filtersql.org](https://filtersql.org).

Il passo successivo, cioè usare `filtersql` come compilatore per far interrogare un database a un LLM in modo sicuro, verrà trattato in un altro post.
