---
layout: post
title: "String equality vs semantic equivalence: a bug in a SQL compiler"
date: 2026-10-15
categories:
  - python
  - filtersql
---
When I released version 1.2.6 of `filtersql` - a zero-dependency Python library designed specifically to compile structured JSON payloads into secure, parameterized SQL queries - I was confident in its security model. Then a little new edge case popped up.

It looked like the classic minor slip - the kind of one-line fix you expect to resolve in five minutes. But when I dove into the code, I quickly realized that the patch would actually fix the specific problem, but the security hole was much bigger.

The bug wasn't in the logic; **it was architectural**.

---

## Premise 1: The wild west of database column names

Real-world database schemas can be remarkably anarchic. Columns can contain spaces, accents, special characters, and Unicode symbols.

To handle clean quoting without overcomplicating things, I initially applied a simple normalization rule: trimming leading and trailing whitespace from identifiers. A column like `"total "` (with trailing spaces) would be cleaned up into `"total"`. Some edge cases would be cut off, but most standard cases would be fine.

---

## Premise 2: The nightmare of `scope` collisions

`filtersql` handles **`scope`**: a map of fixed server-side conditions (such as `tenant_id = 42`) injected automatically into `select`, `insert`, `update`, and `delete` operations. While it doesn't replace database-level row-level security, it acts as a strong guardrail for multi-tenant web apps.

At first, I didn't apply any blocking logic to prevent a client payload from trying to override those columns, but I quickly realized that something was missing.

For reads and deletes, the scope condition is added to the `where` clause as an `and`, so `select` and `delete` are safe by construction. Writes are a different story: that's where things get interesting.
### The `insert` case
```sql
insert into users (tenant_id, tenant_id, name) values (400, 500, 'Mario');
```
Depending on the DBMS, this might pick the first value, the last value, or fail outright.

### The `update` case (much worse)
```sql
update users set tenant_id = 500 where tenant_id = 400;
```
A single malformed or malicious payload could silently transfer records from one tenant to another. Aside from losing records, a malicious user could exploit this behavior to shift an admin account from their own tenant to a victim's tenant and take it over.

The solution seemed easy: implement a **collision check**.

---
## The v1.2.6 bug

Unfortunately, that collision check had a little bug.

If the scope protects the column `"tenant_id"`, and a client sends a write payload with the key `"tenant_id "` (with a trailing space), what happens?

1. The collision check compared raw strings: `"tenant_id"` vs `"tenant_id "`.
2. They were evaluated as **different**, so the check gave the green light.
3. Later in the pipeline, the quoting function sanitized the identifier by applying `.strip()`.
4. Result: the query was compiled and executed, successfully overriding the protected `tenant_id`!

This is a very classic **canonicalization bug**: the validation check and the final execution engine were looking at two different representations of the same data. Fixing this specific check by moving the `.strip()` earlier would have solved the immediate bug. Five minutes of work.

But that's when the real architectural problem emerged.

---
## The architectural problem: string equality vs. semantic equivalence

Two identical strings are equal, which is a comforting thought.

But **two completely different strings can mean the exact same thing to a database engine**.
### 1. The qualified identifier trap (`users.tenant_id`)

What happens if the server sets
```python
scope: {"tenant_id": 400}
```
 and the client sends
```python
values: {"users.tenant_id": 500}
```
 ?

A string-based collision check sees no match (`"tenant_id"` != `"users.tenant_id"`). The library happily generated:

```sql
update users
set
  users.tenant_id = 500
where
  tenant_id = 400;
```

While Postgres or SQLite rejected this syntax with quiet dignity, _MySQL_ accepted it enthusiastically, executing the unauthorized tenant transfer.

![Adminer executing unauthorized tenant transfer on MariaDB](/images/20261015_mariadb_query.png)

### The deeper semantic problem: case, aliases, and Unicode

But table-qualification was only the tip of the iceberg. There was actually _much_ more to it.

Consider case-insensitivity, database aliases, and Unicode normalization forms (`NFC` vs `NFD`), where the letter `à` can be represented by a single codepoint or two separate ones. The raw byte strings differ entirely, but the database treats them as the exact same column.

Trying to guess every possible semantic variation an attacker or a database could invent was a losing battle. `filtersql` only generates strings and is intentionally decoupled from a live database connection.

---
## Asymmetric validation: strict on writes, flexible on reads

To solve this, I initially realized I couldn't make string comparisons "smarter" - I had to **change the contract at the boundary**.

I introduced an asymmetric architecture:

1. **In reads (`select`, `where`):** The system remains flexible, accepting qualified identifiers (`users.tenant_id`) or JSONB paths because they are safely quoted and parsed separately.
2. **In writes (`insert`, `update`, `delete`):** A strict rule was established: all keys in `values` or `id` must pass **`_validate_bare_identifier`**.

A valid write identifier must be a pure bare identifier: no leading or trailing whitespace (trimming is gone; if a space is present, it's rejected with an `InvalidIdentifierError`), no dots, no JSONB operators, and in **mandatory Unicode NFC canonical form**. By normalizing both the scope and the write keys to NFC and applying `.casefold()`, the collision space collapsed.
#### Looking back

In hindsight, explicit allowlists should have been part of the design from day one.

The identifier validation rules and NFC normalization with `casefold()` addressed the immediate vulnerabilities, but they also exposed a deeper limitation: trying to anticipate every way a database might interpret an identifier is a losing game.

That's why `filtersql` 1.3.0 introduced `allowed_columns` and `allowed_sources`. Rather than trying to recognize every potentially dangerous identifier, applications can explicitly define which columns and data sources a given endpoint is allowed to expose. Anything outside that boundary is rejected.

This also clarifies the separation of responsibilities:

- The application defines the allowed data surface for each endpoint and supplies trusted server-side constraints, such as tenant scope.
- `filtersql` acts as a deterministic SQL compiler, validating the input against those constraints and generating parameterized SQL.

The goal isn't to make every possible identifier safe by guessing how a database will interpret it. It's to constrain what the compiler is allowed to express in the first place.

_You can find the code, docs, and examples on [filtersql.org](https://filtersql.org/) or [GitHub](https://github.com/fthiella/filtersql)._