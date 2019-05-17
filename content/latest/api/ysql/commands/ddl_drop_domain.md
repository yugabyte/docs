---
title: DROP DOMAIN
summary: Remove a domain
description: DROP DOMAIN
menu:
  latest:
    identifier: api-ysql-commands-drop-domain
    parent: api-ysql-commands
aliases:
  - /latest/api/ysql/ddl_drop_domain/
isTocNested: true
showAsideToc: true
---

## Synopsis
The `DROP DOMAIN` command removes a domain from the database.

## Syntax

### Diagrams

<svg class="rrdiagram" version="1.1" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns="http://www.w3.org/2000/svg" width="385" height="49" viewbox="0 0 385 49"><path class="connector" d="M0 21h5m54 0h10m57 0h30m30 0h10m61 0h20m-136 0q5 0 5 5v8q0 5 5 5h111q5 0 5-5v-8q0-5 5-5m5 0h10m93 0h5"/><rect class="literal" x="5" y="5" width="54" height="24" rx="7"/><text class="text" x="15" y="21">DROP</text><rect class="literal" x="69" y="5" width="57" height="24" rx="7"/><text class="text" x="79" y="21">TABLE</text><rect class="literal" x="156" y="5" width="30" height="24" rx="7"/><text class="text" x="166" y="21">IF</text><rect class="literal" x="196" y="5" width="61" height="24" rx="7"/><text class="text" x="206" y="21">EXISTS</text><a xlink:href="../grammar_diagrams#table-name"><rect class="rule" x="287" y="5" width="93" height="24"/><text class="text" x="297" y="21">table_name</text></a></svg>

### Grammar
```
drop_domain ::= DROP DOMAIN [IF EXISTS ] name [, ...]  [ CASCADE | RESTRICT ]
```

Where

- `IF EXISTS` does not throw an error if domain does not exist.
- `name` is the name of the existing domain.
- `CASCADE` automatically drops objects that depend on the domain.
- `RESTRICT` refuses to drop the domain if objects depend on it (default).

## Semantics

- An error is raised if the specified name does not exist (unless `IF EXISTS` is set).
- An error is raised if any objects depend on this domain (unless `CASCADE` is set).

## Examples
Example 1

```sql
postgres=# CREATE DOMAIN idx DEFAULT 5 CHECK (VALUE > 0);
```

```sql
postgres=# DROP DOMAIN idx;
```

Example 2

```sql
postgres=# CREATE DOMAIN idx DEFAULT 5 CHECK (VALUE > 0);
```

```sql
postgres=# CREATE TABLE t (k idx primary key);
```

```sql
postgres=# DROP DOMAIN idx CASCADE;
```

## See Also

[`CREATE DOMAIN`](../ddl_create_domain)
[`ALTER DOMAIN`](../dml_alter_domain)
[Other YSQL Statements](..)
