---
title: CREATE DOMAIN
linkTitle: CREATE DOMAIN
summary: Create a new domain in a database
description: CREATE DOMAIN
menu:
  latest:
    identifier: api-ysql-commands-create-domain
    parent: api-ysql-commands
aliases:
  - /latest/api/ysql/ddl_create_domain/
isTocNested: true
showAsideToc: true
---

## Synopsis

The `CREATE DOMAIN` command creates a user-defined data type with a range, optional DEFAULT, NOT NULL and CHECK constraint.

## Syntax

### Diagrams

#### create_domain

<svg class="rrdiagram" version="1.1" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns="http://www.w3.org/2000/svg" width="1082" height="65" viewbox="0 0 1082 65"><path class="connector" d="M0 22h5m67 0h10m70 0h10m54 0h30m36 0h20m-71 0q5 0 5 5v8q0 5 5 5h46q5 0 5-5v-8q0-5 5-5m5 0h10m80 0h30m74 0h10m69 0h20m-188 0q5 0 5 5v8q0 5 5 5h163q5 0 5-5v-8q0-5 5-5m5 0h30m75 0h10m83 0h20m-203 0q5 0 5 5v8q0 5 5 5h178q5 0 5-5v-8q0-5 5-5m5 0h30m132 0h30m32 0h20m-67 0q5 0 5 5v8q0 5 5 5h42q5 0 5-5v-8q0-5 5-5m5 0h20m-249 0q5 0 5 5v23q0 5 5 5h224q5 0 5-5v-23q0-5 5-5m5 0h5"/><rect class="literal" x="5" y="5" width="67" height="25" rx="7"/><text class="text" x="15" y="22">CREATE</text><rect class="literal" x="82" y="5" width="70" height="25" rx="7"/><text class="text" x="92" y="22">DOMAIN</text><a xlink:href="../grammar_diagrams#name"><rect class="rule" x="162" y="5" width="54" height="25"/><text class="text" x="172" y="22">name</text></a><rect class="literal" x="246" y="5" width="36" height="25" rx="7"/><text class="text" x="256" y="22">AS</text><a xlink:href="../grammar_diagrams#data-type"><rect class="rule" x="312" y="5" width="80" height="25"/><text class="text" x="322" y="22">data_type</text></a><rect class="literal" x="422" y="5" width="74" height="25" rx="7"/><text class="text" x="432" y="22">COLLATE</text><a xlink:href="../grammar_diagrams#collation"><rect class="rule" x="506" y="5" width="69" height="25"/><text class="text" x="516" y="22">collation</text></a><rect class="literal" x="625" y="5" width="75" height="25" rx="7"/><text class="text" x="635" y="22">DEFAULT</text><a xlink:href="../grammar_diagrams#expression"><rect class="rule" x="710" y="5" width="83" height="25"/><text class="text" x="720" y="22">expression</text></a><rect class="literal" x="843" y="5" width="132" height="25" rx="7"/><text class="text" x="853" y="22">domain_constraint</text><a xlink:href="../grammar_diagrams#..."><rect class="rule" x="1005" y="5" width="32" height="25"/><text class="text" x="1015" y="22">...</text></a></svg>

#### domain_constraint

<svg class="rrdiagram" version="1.1" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns="http://www.w3.org/2000/svg" width="634" height="125" viewbox="0 0 634 125"><path class="connector" d="M0 37h25m98 0h10m122 0h20m-265 0q5 0 5 5v8q0 5 5 5h240q5 0 5-5v-8q0-5 5-5m5 0h50m-5 0q-5 0-5-5v-17q0-5 5-5h274q5 0 5 5v17q0 5-5 5m-269 0h20m45 0h10m52 0h137m-254 25q0 5 5 5h5m52 0h177q5 0 5-5m-249-25q5 0 5 5v50q0 5 5 5h5m61 0h10m25 0h10m83 0h10m25 0h5q5 0 5-5v-50q0-5 5-5m5 0h40m-339 0q5 0 5 5v68q0 5 5 5h314q5 0 5-5v-68q0-5 5-5m5 0h5"/><rect class="literal" x="25" y="20" width="98" height="25" rx="7"/><text class="text" x="35" y="37">CONSTRAINT</text><a xlink:href="../grammar_diagrams#constraint-name"><rect class="rule" x="133" y="20" width="122" height="25"/><text class="text" x="143" y="37">constraint_name</text></a><rect class="literal" x="345" y="20" width="45" height="25" rx="7"/><text class="text" x="355" y="37">NOT</text><rect class="literal" x="400" y="20" width="52" height="25" rx="7"/><text class="text" x="410" y="37">NULL</text><rect class="literal" x="345" y="50" width="52" height="25" rx="7"/><text class="text" x="355" y="67">NULL</text><rect class="literal" x="345" y="80" width="61" height="25" rx="7"/><text class="text" x="355" y="97">CHECK</text><rect class="literal" x="416" y="80" width="25" height="25" rx="7"/><text class="text" x="426" y="97">(</text><a xlink:href="../grammar_diagrams#expression"><rect class="rule" x="451" y="80" width="83" height="25"/><text class="text" x="461" y="97">expression</text></a><rect class="literal" x="544" y="80" width="25" height="25" rx="7"/><text class="text" x="554" y="97">)</text></svg>

### Grammar
```
create_domain ::= CREATE DOMAIN name [ AS ] data_type
    [ COLLATE collation ]
    [ DEFAULT expression ]
    [ domain_constraint [ ... ] ];

domain_constraint ::= [ CONSTRAINT constraint_name ]
{ NOT NULL | NULL | CHECK (expression) };
```

Where

- `name` is the name of the domain.
- `data_type` is the underlying data type.
- `COLLATE collation` is an optional collation for the domain.
- `DEFAULT expression` sets default value for columns and must match data type of the domain.
- `CONSTRAINT constraint_name` is an optional name for constraint.
- `NOT NULL` does not allow null values.
- `NULL` allows null values (default).
- `CHECK (expression)` must produce a boolean result.

## Semantics

- An error is raised if `name` already exists in the specified database.

## Examples

```sql
postgres=# CREATE DOMAIN idx DEFAULT 5 CHECK (VALUE > 0);
```

```sql
postgres=# CREATE TABLE sample(k text primary key, v idx);
```

## See Also
[`DROP CONSTRAINT`](../ddl_drop_constraint)
[`ALTER CONSTRAINT`](../ddl_alter_constraint)
[Other YSQL Statements](..)
