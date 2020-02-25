In this section, we describe the standard SQL constructs and query clauses supported by Sclera. The extensions are covered in [later sections](/doc/ref/sqlintro).

The SQL constructs discussed below enable you to create, populate, query, update and delete base tables; these lie at the core of Sclera, and are supported across all editions.

Sclera supports a significant subset of standard SQL. The specific SQL dialect is largely compatible with [PostgreSQL](http://www.postgresql.org/docs/current/static/sql.html). In addition, Sclera supports a few [proprietary short-cuts](#querying-data-using-generalized-table-expressions) that, in our opinion, make the scripts more compact and readable.

This section is neither a tutorial, nor a formal specification of the supported SQL, and only serves as a guide illustrating the differences between Sclera's SQL and the [PostgreSQL's SQL commands](http://www.postgresql.org/docs/current/static/sql.html). If you need help with the basics, or want a more formal description, we suggest that you read this section alongside [PostgreSQL's excellent documentation](http://www.postgresql.org/docs/current/static/sql.html).

**Syntax Notation:** `[ .. ]` means optional, `|` separates the alternatives, `[ .. | .. ]` lists alternatives at most one of which should be included, `{ .. | .. }` lists alternatives exactly one of which should be included, and `[, ...]` stands for comma-separated repeats of the immediately preceding expression. In the following, the keywords appear in upper case to distinguish them from the other terms; however, Sclera is case-insensitive and keywords in actual commands and queries can be in upper or lower case.

We start with describing the syntax of a [scalar expression](#scalar-expressions), followed with that of a [select expression (or, query)](#querying-data-using-select-expressions). Next, we describe the syntax of SQL commands for [creating base tables](#creating-base-tables), [inserting](#inserting-rows-into-a-base-table), [updating](#updating-rows-in-a-base-table), [deleting](#deleting-rows-in-a-base-table) data in base tables, and finally [dropping](#dropping-base-tables) base tables. This is followed with commands for [creating](#creating-views) and [dropping](#dropping-views) views. Finally, we describe the [data types](#data-types) supported by ScleraSQL, and the [reserved words](#reserved-words) that cannot be used except as a part of the SQL syntax.

## Scalar Expressions
A scalar expression (equivalently, a `scalar_expression`) can be any of the following:

- A `NULL` or [a constant](http://www.postgresql.org/docs/current/static/sql-syntax-lexical.html#SQL-SYNTAX-CONSTANTS)
- <a class="anchor" name="column-reference"></a>A [column reference](http://www.postgresql.org/docs/current/static/sql-expressions.html#SQL-EXPRESSIONS-COLUMN-REFS) to a column in any of the `from_item` elements in the [`FROM` clause](#from-clause). The column reference can be qualified by the associated table name or alias.
- An expression with a [type cast](http://www.postgresql.org/docs/current/static/sql-expressions.html#SQL-SYNTAX-TYPE-CASTS). The type cast can be used to convert the type of an expression to a supported [data type](#data-types).
- A [scalar subquery](http://www.postgresql.org/docs/current/static/sql-expressions.html#SQL-SYNTAX-SCALAR-SUBQUERIES), which is a [SQL query](#querying-data-using-select-expressions), enclosed in parenthesis, whose result can have exactly one column and at most one row. If the result is empty, the value of the subquery is `NULL`, otherwise it gets the value of the column in the returned row.
    - <a class="anchor" name="correlated-subquery"></a>Sclera does not support correlated subqueries; in other words, you should be able to execute the scalar subquery as a stand-alone SQL query. The SQL standard is more general as it allows subqueries with expressions containing parameters coupling it to the enclosing query (see [PostgreSQL's documentation](http://www.postgresql.org/docs/current/static/sql-expressions.html#SQL-SYNTAX-SCALAR-SUBQUERIES)).
- <a class="anchor" name="aggregate-expression"></a>An [aggregate expression](http://www.postgresql.org/docs/current/static/sql-expressions.html#SYNTAX-AGGREGATES). Aggregate expressions can only appear in the [`SELECT` clause](#select-clause), [`ORDER BY` clause](#order-by-clause), or [`HAVING` clause](#having-clause).
- A complex scalar expression constructed by applying supported [functions and operators](http://www.postgresql.org/docs/current/static/functions.html) on other [scalar expressions](#scalar-expressions).

The expressions in Sclera are mostly compliant with the supported subset of the SQL standard and [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-expressions.html). The difference is in lack of support for [correlated subqueries](#correlated-subquery), extended [data types](#data-types) and associated [functions and operators](http://www.postgresql.org/docs/current/static/functions.html).

## Querying Data using Select Expressions

<a class="anchor" name="select-expression"></a>A `SELECT` statement (equivalently, a `select_expression`) has the following syntax:

    SELECT [ ALL | DISTINCT [ ON ( expression [, ...] ) ] ]
           { expression [ [ AS ] output_name ] | abbreviation } [, ...]
    [ FROM from_item [, ...] ]
    [ WHERE condition ]
    [ GROUP BY expression [, ...] ]
    [ HAVING condition [, ...] ]
    [ { UNION | INTERSECT | EXCEPT } [ ALL | DISTINCT ] select_expression ]
    [ ORDER BY expression [ ASC | DESC ] [ NULLS { FIRST | LAST } ] [, ...] ]
    [ LIMIT count [ ROW | ROWS ] ]
    [ OFFSET start [ ROW | ROWS ] ]
    [ FETCH [ FIRST | NEXT ] [ count ] [ ROW | ROWS ] ONLY ]

This is a restriction of [PostgreSQL's select statement syntax](http://www.postgresql.org/docs/current/static/sql-select.html).

The semantics of a `select_expression` (that is, SQL engine's interpretation of what is to be evaluated), and description of the various clauses appears below. The clauses are explained roughly in the order in which they are executed by Sclera's execution engine.

### `FROM` Clause
The `FROM` clause specifies the input to the query as a list of `from_item` elements.

<a class="anchor" name="from-item"></a>Each `from_item` in the `FROM` clause can be one of:

- <a class="anchor" name="from-table-name"></a>`[location_name.]table_name [ [ AS ] alias [ ( column_alias [, ...] ) ] ]`
    - `table_name` is a base table at the data [location](/doc/ref/dbms#location) `location_name`.
        - If the `table_name` is unique across the locations, you can omit the `location_name`.
    - `alias`, when specified, becomes the substitute name for `table_name` within the scope of the query. If `alias` is specified, `table_name` can no longer be used in any [expression](#scalar-expressions).
    - <a class="anchor" name="column-alias"></a>`column_alias`, when specified, becomes the substitute name for the corresponding column in the table `table_name`. If `column_alias` is specified for a column, the original column name can no longer be used in any [expression](#scalar-expressions).
        - If the number of column aliases is less than the number of columns in `table_name`, the remaining columns of `table_name` retain their original names.
        - If the number of column aliases is more than the number of columns in `table_name`, the remaining columns aliases are ignored.
- <a class="anchor" name="from-select-expression"></a>`( select_expression ) [ AS ] alias [ ( column_alias [, ...] ) ]`
    - Defines a virtual table computed using the nested `select_expression` and referenced using the name `alias`.
    - By default, the output columns of the nested `select_expression` become the columns of the virtual table. However, if a `column_alias` list is specified, it overrides these default names, [as earlier](#column-alias).
- <a class="anchor" name="from-values"></a>`VALUES ( constant [, ...] ) [, ...] [ AS ] alias [ ( column_alias [, ...] ) ]`
    - Defines a virtual table containing the rows enumerated explicitly as a comma-separated list of tuples, and referenced using the name `alias`. Note that all the rows must confirm to the same schema -- that is, each row must have the same number of elements, and the element at a given position in each row must have the same data type.
    - By default, the Sclera assigns unique names to the columns in the rows. However, if a `column_alias` list is specified, it overrides these default names, [as earlier](#column-alias).
- <a class="anchor" name="from-table-join"></a>`from_item [ NATURAL ] join_type from_item [ ON join_condition | USING ( join_column [, ...] ) ]`
    - Defines a composite `from_item` formed by joining two `from_item` elements, hereafter referred to as *inputs*.
    - `join_type` can be one of `[ INNER ] JOIN`, `LEFT [ OUTER ] JOIN`, `RIGHT [ OUTER ] JOIN`, `FULL [ OUTER ] JOIN` or `CROSS JOIN` (recall that words in `[ .. ]` are optional -- here, the words `INNER` and `OUTER` are just syntactic sugar).
    - If the `join_type` is not a `CROSS JOIN`, you can specify at most one of the `ON join_condition`, `USING ( join_column [, ...] ) ]` or `NATURAL`.
        - The `join_condition` in `ON` is a boolean-valued [expression](#scalar-expressions) on the consolidated output columns of the two inputs.
        - The `join_column` list in `USING` is shorthand for a join condition which is a conjunct of equalities among columns with the same name. Also, the output contains only one copy of the common columns in the `join_column` list.
            - Example: `X JOIN Y USING (A, B)` is shorthand for `X JOIN Y ON (X.A = Y.A AND X.B = Y.B)`. The output contains only one copy of columns `A` and `B`.
        - The `NATURAL` is a shorthand for a `USING` with the `join_column` list containing *all* columns with the same name across the two inputs; these column names need not be specified explicitly when `NATURAL` is used.
            - Example: Given table `X` with columns `A`, `B`, `C` and table `Y` with columns `B`, `C`, `D`, the `from_item` `X NATURAL JOIN Y` is shorthand for `X JOIN Y USING (B, C)`.
        - If none of the `ON`, `USING` or `NATURAL` are specified, the join is considered a `CROSS JOIN`.
    - If the `join_type` is a `CROSS JOIN`, none of the `ON`, `USING` or `NATURAL` should be specified.

A comma-separated list of more than one `from_item` elements translates to a cross-product of these elements, right to left; this cross-product is rewritten to an `INNER JOIN` by the optimizer if the `WHERE` clause includes cross-input predicates.

The `FROM` clause is optional. If the clause is absent, the input is implicit as a single row with a single `NULL`-valued column.

This clause is compliant with the supported subset of the SQL standard and [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-select.html#SQL-FROM).

#### Cross-System Joins
When a join contains inputs from more than one database system, it is a cross-system join.

While planning the evaluation of cross-system joins, Sclera considers the left input of a join larger than the right input, and prefers moving the right input to the left input's location. The ordering of the `from_item`s in a `FROM` clause thus matters when evaluating cross-system joins (see the [technical documentation](/doc/ref/technical#relational-operators) for details). While this enables you to control how data is moved while evaluating a query, you need to pay special attention to this ordering -- especially when significant amounts of data needs to be transfered.

Specifically, when specifying a join between a relational database and HBase, if large amount of data in HBase is expected to be involved in the join, then you should place the HBase source leftmost in the `FROM` list; this will ensure that data from the relational database is moved to HBase for the join, not vice-versa.

In any case, when evaluating a query with a cross-system join, please take a close look at the query's evaluation plan (obtained using the [`EXPLAIN` shell command](/doc/ref/shell#compile-time-explain)) before submitting the query.

### `WHERE` Clause
The `WHERE` clause specifies a boolean [expression](#scalar-expressions) on the column names in the output of the [`FROM` clause](#from-clause). This boolean expression is used to filter the rows in the input.

The boolean expression can only contain column names in the output of the [`FROM` clause](#from-clause), and cannot contain an [aggregate expression](#aggregate-expression).

The `WHERE` clause is optional. If the clause is absent, the input is not filtered.

This clause is compliant with the SQL standard and [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-select.html#SQL-WHERE).

### `GROUP BY` Clause
The `GROUP BY` clause groups the input rows, creating a group for each distinct combination of the `GROUP BY` [expressions](#scalar-expressions).

The clause works with aggregate functions in the [`SELECT` clause](#select-clause), [`ORDER BY` clause](#order-by-clause), and/or [`HAVING` clause](#having-clause). These clauses generate a row for each group, which can then be [output](#select-clause), or used for [ordering](#order-by-clause) or [filtering](#having-clause) the result.

Each expression in [`SELECT`](#select-clause), [`ORDER BY`](#order-by-clause) or [`HAVING`](#having-clause) clauses not appearing within an aggregate function must build on the `GROUP BY` expressions or constants. The output is a row for each group.

The `GROUP BY` clause in Sclera is mostly compliant with the SQL standard and [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-select.html#SQL-GROUPBY). In SQL standard and [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-select.html#SQL-GROUPBY), you can include an expression in the `SELECT` clause in the `GROUP BY` list by specifying its position number. This is not allowed in Sclera; instead, you can associate the `SELECT` expression with an alias (`output_name` in the [syntax](#select-expression) stated above) and use that in the `GROUP BY` list.

### `SELECT` Clause
The mandatory `SELECT` clause specifies the output of the query. The specification consists of a list of [scalar expressions](#scalar-expressions), optionally associated with alias names.

Each scalar expression in the clause corresponds to a column in the output; the name of the column is the expression's alias name, or an automatically generated name.

The `SELECT` expressions can only contain column names in the output of the [`FROM` clause](#from-clause), and may include [aggregate expressions](#aggregate-expression).

#### Abbreviations
SQL provides certain abbreviations when specifying the `SELECT` expression list. An abbreviation is a special expression that expands to an expression list on compilation. In Sclera, an abbreviation can be one of the following:

- `* [EXCEPT column_list]`: The `*` expands to an expression list consists of all the columns in the output of the [`FROM` clause](#from-clause). If `EXCEPT column_list` is specified, then the columns in the `column_list` are not included in the expansion.
- `alias.* [EXCEPT column_list]`: The `alias.*` expands to an expression list consists of all the columns in the `from_item` with alias `alias` in the [`FROM` clause](#from-clause). If `EXCEPT column_list` is specified, then the columns in the `column_list` are not included in the expansion.

In the above, `column_list` is either a `column_alias`, or a comma-separated `column_alias` list in parenthesis.

The `EXCEPT` modifier is a Sclera addition to the syntax, it is not supported in the SQL standard or in [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-select.html).

Abbreviations cannot be used in the `SELECT` clause if a `GROUP BY` clause is present in the query.

#### Distinct
The select clause also has an optional `DISTINCT` clause, included between `SELECT` and the expression list, which comes in two variants.

- The first variant, specified by the modifier `DISTINCT`, eliminates duplicate rows in the output. This clause is part of the SQL standard.
- The second variant, specified by `DISTINCT ON (expression [, ...])`, is a generalization of the first variant. The rows are grouped based on the distinct values of the `DISTINCT ON` expressions, and one row from each row is output. If more than one rows exist in a group, this choice is arbitrary. However, if an [`ORDER BY` clause](#order-by-clause) is present in the query, the rows in each group are first ordered, and the first row (in the resulting sorted order of rows) is chosen for output. This variant is also supported by [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-select.html#SQL-DISTINCT), but is not a part of the SQL standard.

#### Aggregation
If the `SELECT` expressions include [aggregate expressions](#aggregate-expression), then the input rows are aggregated.

If a [`GROUP BY` clause](#group-by-clause) is present, there is a group for each distinct combination of the `GROUP BY` expression values; otherwise all the input rows form a single group. The rows in each group are aggregated based on the aggregate expressions, resulting in one output row for each group.

For the above computation to be valid, all column names that do not appear as expressions in the `GROUP BY` clause (and are hence not constant within a group), must appear within an [aggregation function](/doc/ref/sqlmisc#aggregate-functions).

The `SELECT` clause in Sclera is compliant with the SQL standard and [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-select.html#SQL-SELECT-LIST).

### `HAVING` Clause
The `HAVING` clause specifies a boolean-valued aggregate expression that is evaluated in the same way as the aggregate expressions in [`SELECT` clause](#aggregation).

However, unlike the [`SELECT` clause](#select-clause), the resulting value is not output. Instead, it is used to filter the aggregation result.

This clause is compliant with the SQL standard and [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-select.html#SQL-HAVING).

### `UNION`/`INTERSECT`/`EXCEPT` Clause
The `UNION`, `INTERSECT` and `EXCEPT` clauses are used to compute the set union, intersection or difference, respectively, on the result of two queries.

The modifier `DISTINCT` removes duplicate rows in the computed result; this is also the default when no modifier is specified. To disable the removal of duplicate rows, you can specify `ALL` instead.

These clauses in Sclera are compliant with the SQL standard and [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-select.html#SQL-UNION).

#### Cross-System Set Operations
When the clause contains queries evaluated at more than one system, it specifies a cross-system set operation.

To evaluate a cross-system set operation, Sclera needs both the query results to be present at a single location; let us call this the "target location" for the set operation. This target location is decided as follows:

- For each query, Sclera finds the location of the query result after evaluation. These locations are the candidates for the target location, and are listed in the order of appearance of the corresponding queries.
- From this list, Sclera then removes the [Cache Store](/doc/ref/technical#cache-store), if present, as well as the ["read-only" locations](/doc/ref/dbms#read-write-versus-read-only-mode).
- If the list is empty, Sclera assigns the [Cache Store](/doc/ref/technical#cache-store) as the target location. This has the effect that cross-system set operations across read-only locations are evaluated by evaluating both the queries, and moving the query results to the cache store; the set operation is then computed at the cache store.
- If the list is not empty, Sclera assigns the location on the left in the list as the target location. This has the effect that the query in the other location is evaluated, and its result is moved to the target location. The set operation is then computed along with the evaluation of the query in the target location.

The ordering of the queries thus matters when evaluating cross-system set operations. While this enables you to control how data is moved while evaluating the queries containing set operations, you need to pay special attention to this ordering -- especially when significant amounts of data needs to be transfered.

Specifically, when specifying a set operation between queries evaluated in a relational database and HBase respectively, if the HBase query's result is expected to be large, then you should place the HBase query before (that is, to the left of) the other query; this will ensure that HBase is picked as the target location in the set operation.

In any case, when evaluating a query with a cross-system set operation, please take a close look at the query's evaluation plan (obtained using the [`EXPLAIN` shell command](/doc/ref/shell#compile-time-explain)) before submitting the query.

In the current version, Sclera moves data from a "source" to a "target" database system by reading in the data from the source and inserting it into a temporary table in the target. This transfer is done in a streaming (pipelined) manner wherever possible, to avoid reading the entire result in memory. This could be a bottleneck when large amounts of data (millions of rows) are transferred. More efficient data transfer mechanisms will be in place in later versions of Sclera.

### `ORDER BY` Clause
The `ORDER BY` clause specifies a list of [scalar expressions](#scalar-expressions) that determines the sort order of the computed result.

The optional modifiers `ASC` (for *ascending*) or `DESC` (for *descending*) associated with each expression in the `ORDER BY` list determine how that particular expression should be considered while determining the overall sort order.
When `ASC` is specified, the larger values are ordered after the smaller values, and the `NULLS` are ordered after all values. When `DESC` is specified, the smaller values are ordered after the larger values, and the `NULLS` are ordered before all values. The default is `ASC`.

The optional `NULLS FIRST` and `NULLS LAST` modifiers force the NULLS to be ordered before or after all values.

The `ORDER BY` clause in Sclera is mostly compliant with the SQL standard and [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-select.html#SQL-ORDERBY). In SQL standard and [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-select.html#SQL-ORDERBY), you can include an expression in the `SELECT` clause in the `ORDER BY` list by specifying its position number. This is not allowed in Sclera; instead, you can associate the `SELECT` expression with an alias (`output_name` in the [syntax](#select-expression) stated above) and use that in the `ORDER BY` list.

### `LIMIT`, `FETCH` and `OFFSET` Clauses
The `LIMIT` clause retains the specified number of initial rows in the result. If the result has less than the specified number of rows, this clause has no effect.

The `FETCH` clause is an alternative to `LIMIT`, with a relatively verbose syntax. Here, again, you can specify the number of initial rows to retain in the result; but you can also say `FETCH FIRST ROW ONLY` to retain only the first row, if present. In general, the words `FIRST`, `NEXT`, `ROW` and `ROWS` are optional syntactic sugar; the number of initial rows, if not specified, defaults to `1`.

A query can have either the `LIMIT` clause or the `FETCH` clause.

The `OFFSET` clause discards the specified number of initial rows in the result. If the result has less than the specified number of rows, all rows are discarded.

When both the `OFFSET` and the `LIMIT` (or `FETCH`) clauses are present in a query, the `OFFSET` clause applies before the `LIMIT` (or `FETCH`) clause.

In absence of an [`ORDER BY` clause](#order-by-clause), the query result is in arbitrary order and consequently, the retained/discarded row set is not deterministic (i.e. a different set of rows may be retained/discarded on each invocation). As such, the recommended use of these clauses is in conjunction with the [`ORDER BY` clause](#order-by-clause).

The `FETCH` and `OFFSET` clauses in Sclera are compliant with the SQL standard and [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-select.html#SQL-LIMIT). The `LIMIT` clause is supported in [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-select.html#SQL-LIMIT), but not in the SQL standard.

## Querying Data Using Generalized Table Expressions
In standard SQL, as well as in [PostgreSQL](http://www.postgresql.org/docs/current/static/sql.html) and other relational databases, a query is restricted to the [`SELECT` statement](#select-expression) described [above](#querying-data-using-select-expressions).

<a class="anchor" name="table-expression"></a>Sclera generalizes the query syntax such that [anything](#from-item) that can appear in the [`FROM` clause](#from-clause), possibly without the alias, is accepted as a query. We call this a `table_expression`. To recap, this includes:

- A standard [`SELECT` statement](#select-expression).
    - As a result, any query in the [earlier query syntax](#select-expression) remains admissible; this is indeed a generalization of the ["standard" select expression / query discussed above](#querying-data-using-select-expressions).
- A base table name, possibly qualified with a location. You can rename the table and the columns the same way you could in the [`FROM` clause](#from-table-name).
    - **Syntax:** `[location_name.]table_name [ [ AS ] alias [ ( column_alias [, ...] ) ] ]` (see [above](#from-table-name) for explanation)
- A values expression, which explicitly lists out the rows.
    - **Syntax:** `VALUES ( constant [, ...] ) [, ...] [ AS ] alias [ ( column_alias [, ...] ) ]` (see [above](#from-values) for explanation)
- A join expression involving nested `table_expression` elements.
    - **Syntax:** `table_expression [ NATURAL ] join_type table_expression [ ON join_condition | USING ( join_column [, ...] ) ]` (see [above](#from-table-join) for explanation)
- A table expression in parenthesis, with optional alias.
    - **Syntax:** `( table_expression ) [ [ AS ] alias [ ( column_alias [, ...] ) ] ]`

In addition, a `table_expression` can have additional `WHERE`, `ORDER BY` and `LIMIT`/`OFFSET`/`FETCH` clauses. This is not supported in the standard SQL `FROM` clause.

- A table expression with a [`WHERE` clause](#where-clause).
    - **Syntax:** `( table_expression ) WHERE condition`
- A table expression with an [`ORDER BY` clause](#order-by-clause).
    - **Syntax:** `( table_expression ) ORDER BY expression [ ASC | DESC ] [ NULLS { FIRST | LAST } ] [, ...] ]`
- A table expression with [`LIMIT`, `FETCH` and/or `OFFSET` clauses](#limit-fetch-and-offset-clauses)
    - **Syntax:** `( table_expression ) [ LIMIT count [ ROW | ROWS ] ] [ OFFSET start [ ROW | ROWS ] ] [ FETCH [ FIRST | NEXT ] [ count ] [ ROW | ROWS ] ONLY ]`

Using this generalized syntax, to get the contents of the table `ORDERS`, you just need to say 

    > ORDERS;

instead of the unnecessarily verbose

    > SELECT * FROM ORDERS;

To filter and limit, you can say

    > ORDERS WHERE CUSTID = 10 LIMIT 3

instead of

    > SELECT * FROM ORDERS WHERE CUSTID = 10 LIMIT 3

Similarly, to get the join of the tables `ORDERS` and `CUSTOMERS`, you just need to say

    > ORDERS JOIN CUSTOMERS USING (CUSTID);

instead of the verbose

    > SELECT * FROM ORDERS JOIN CUSTOMERS USING (CUSTID);

Further, we can union the above with another table `PREV` by saying

    > ORDERS JOIN CUSTOMERS USING (CUSTID) UNION PREV;

instead of the standard

    > SELECT * FROM ORDERS JOIN CUSTOMERS USING (CUSTID) UNION SELECT * FROM PREV;

The queries in the verbose [standard `SELECT` syntax](#select-expression) are still valid, but the equivalent queries in the extended syntax are more compact. As we can see, the difference is more pronouced in more complex queries.

## Base Tables
In this section we describe Sclera's SQL commands for creating, updating and deleting base tables.

### Creating Base Tables

Creating tables in Sclera is different from [adding tables to Sclera using the `ADD TABLE` command](/doc/ref/dbms#importing-database-tables). The latter merely imports the metadata of an *existing* table into Sclera. The `create table` commands, described below, create a new table.

The net effect of these commands is (a) creating the table on the underlying database system, and (b) adding the table metadata to Sclera's metadata store.

If the database system understands SQL (e.g. [MySQL](http://www.mysql.com)), Sclera creates the table by generating the appropriate `CREATE TABLE` command, taking care of dialect and datatype differences; if the database system does not understand SQL (e.g. HBase), Sclera uses the system's API (or any other available mechanism) to create an underlying structure (e.g. HBase tables) that is compatible with the metadata specified in the command.

The `CREATE TABLE` command has two variants. The [first variant](#creating-empty-tables) creates an empty table. The [second variant](#creating-tables-with-empty-results) creates a table, and also populates it with the result of a query.

#### Creating Empty Tables
The syntax of the `CREATE TABLE` command is as follows:

    CREATE [ { TEMPORARY | TEMP } ] TABLE [location_name.]table_name (
      column_name data_type [ column_constraint [ ... ] ] [, ...]
      [ , table_constraint [, ...] ]
    )

This creates a new, empty table (persistent or [temporary](#temporary-modifier)) with the name `table_name` at location `location_name` (or the [default location](/doc/ref/configuration#sclera-location-default), if `location_name` is omitted), consisting of columns with names and types given by the associated `column_name` and `column_type`, and with column and table constraints specified by `column_constraint` and `table_constraint`. These terms are described in turn below.

##### Temporary Modifier
The optional `TEMPORARY` (shortened form: `TEMP`) modifier in `CREATE TEMPORARY TABLE` creates a table that exists for the duration of the Sclera session only. When the session ends, the table is deleted from the underlying database system and its metadata is removed from Sclera's metadata store.

##### Table Name
The mandatory `table_name` is the name of the table to be created.

You can optionally qualify the name with `location_name`, the name of the [location](/doc/ref/dbms#location) where the table is to be created. If the `location_name` is omitted, the table is created at the [default location](/doc/ref/configuration#sclera-location-default).

##### Columns and Their Types
A table needs to have at least one column. You can specify the list of columns and their types as a comma-separated list of `column_name` and `data_type` pairs.

The `data_type` must be one of the [data types supported by Sclera](#data-types).

##### Column Constraints
With each column in the column list, you can optionally specify a set of column constraints. Each `column_constraint` can be one of:

- `NOT NULL`: This column can never be `NULL`. The constraint is checked while inserting rows into the underlying table.
- `PRIMARY KEY`: This column is the primary key of the table -- as such, it must never be `NULL`, and a value of the column must be unique across all the rows in the table. This constraint can be specified for at most one column, and cannot be specified along with a [`PRIMARY KEY` table constraint](#table-constraints).
- `REFERENCES [location_name.]table_name [ (column_name) ]`: This column references the table `table_name`. The table with the name specified by `table_name` must be present in an underlying database system (if `table_name` is not unique across locations, the `location_name` must be specified). Further:
    - If the `column_name` is specified, the table `table_name` must contain a column with the name specified by `column_name` of the same type as this column and with a `PRIMARY KEY` constraint. Also, for each row in this table, either this column is `NULL`, or there exists a row in table `table_name` with a matching value in the column `column_name`.
    - If the `column_name` is not specified, the table `table_name` must have a primary key, which must be of the same type as this column. Also, for each row in this table, either this column is `NULL`, or there exists a row in table `table_name` with a matching primary key value.

    When the `column_name` is not specified, or equivalently, when it is the primary key of the table `table_name`, this column is called a *foreign key* (to table `table_name`).

##### Table Constraints
A table constraint is a constraint on an entire row of the table, rather than a single column. Table constraints are more general than column constraints; each `column_constraint` mentioned above can be equivalently stated as a `table_constraint`. However, table constraint also allows you to specify multi-column primary keys and references/foreign keys.

The optional `table_constraint` list appears as a continuation of the [column list](#columns-and-their-types). Each `table_constraint` can be one of:

- `PRIMARY KEY ( column_name [, ...] )`: The set of columns within the parenthesis the primary key of the table. As such, each of these columns can never be `NULL`, and the combined set of values of these columns must be unique across all the rows in the table. This constraint cannot be specified along with a [`PRIMARY KEY` column constraint](#column-constraints).
- `FOREIGN KEY ( column_name [, ...] ) REFERENCES [location_name.]table_name [ ( column_name [, ...] ) ]`: The list of columns references the table `table_name`. The table with the name specified by `table_name` must be present in an underlying database system (if `table_name` is not unique across locations, the `location_name` must be specified). Further:
    - If the reference table's `column_name` list is specified, (a) it must have the same number of columns as the foreign key list, (b) the table `table_name` must contain each column in the reference list and these columns must collectively have a `PRIMARY KEY` constraint, and (c) each column of the reference list must have the same type as the corresponding column (i.e. at the same position) in the foreign key list. Also, for each row in this table, either some column in the foreign-key list is `NULL`, or there exists a row in table `table_name` with a matching set of values of the respective columns in the reference list.
    - If the reference table's `column_name` list is not specified, the table `table_name` must have a primary key, which (a) must have the same number of columns as the foreign key list, and (b) each column of this primary key must have the same type as the corresponding column (i.e. at the same position) in the foreign key list. Also, for each row in this table, either some column in the foreign-key list is `NULL`, or there exists a row in table `table_name` with a matching set of values of the respective columns in the primary key.

Note that Sclera does not enforce the column or table constraints described above. Sclera passes on these constraints to the underlying database system, if possible, and it is the responsibility of the underlying database system to check for these constraints. Nevertheless, Sclera assumes that these constraints hold while querying the data in the tables. While creating tables on database systems (such as HBase) which do not enforce such constraints, it is the responsibility of the *user* to make sure that these constraints hold while inserting rows into the table.

This statement is restricted in comparison, but compliant with the SQL standard and [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-createtable.html).

#### Creating Tables with Query Results
This variant of the [`CREATE TABLE` statement](#creating-empty-tables) enables you to create a table, and populates it with the result of the given query. This is more efficient than performing the two tasks one after another, especially in transactional systems such as [MySQL](http://www.mysql.com) and [PostgreSQL](http://www.postgresql.org). Also, this is sometimes more convenient because you need not specify the schema explicitly; if not specified, the schema is derived from the output schema of the given query.

The standard syntax of the `CREATE TABLE AS` command is as follows:

    CREATE [ { TEMPORARY | TEMP } ] TABLE [location_name.]table_name [ (
      column_name data_type [ column_constraint [ ... ] ] [, ...]
      [ , table_constraint [, ...] ]
    ) ] AS select_expression

where `select_expression` is as [described earlier](#select-expression). This is almost identical to the [`CREATE TABLE` syntax](#creating-empty-tables), except the new `select_expression`, and that the schema description (column definitions, column and table constraint specifications) is now optional.

Sclera generalizes this expression to accept a [`table_expression`](#table-expression) instead of a `select_expression`. The generalized syntax is:

    CREATE [ { TEMPORARY | TEMP } ] TABLE [location_name.]table_name [ (
      column_name data_type [ column_constraint [ ... ] ] [, ...]
      [ , table_constraint [, ...] ]
    ) ] AS table_expression

Note that since `select_expression` is a `table_expression`, statements in the standard syntax continues to be valid in this updated syntax.

As in the case of [queries](#querying-data-using-generalized-table-expressions), this generalization simplifies the `CREATE TABLE AS` statements. 

For instance, you can say:

    > CREATE TABLE CUSTORDERS AS ORDERS JOIN CUSTOMERS USING (CUSTID);

instead of the standard:

    > CREATE TABLE CUSTORDERS AS SELECT * FROM ORDERS JOIN CUSTOMERS USING (CUSTID);

This statement is restricted in comparison, but compliant with the SQL standard and [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-createtableas.html).

### Inserting Rows into a Base Table
The `INSERT` statement inserts rows into an existing base table.

The standard syntax of the `INSERT` statement is as follows:

    INSERT INTO [location_name.]table_name [ ( column_name [, ...] ) ] (select_expression | values_expression)

Here, `table_name` is a base table at location `location_name`, into which the rows in the result of `select_expression` or `value_expression` will be inserted. The `select_expression` stands for the [`SELECT` statement described earlier](#select-expression), and the `value_expression` stands for the [`VALUES` clause discussed earlier](#from-values), possibly without the alias.

In the above, the `location_name` can be omitted if `table_name` is unique across locations.

Sclera generalizes the syntax to:

    INSERT INTO [location_name.]table_name [ ( column_name [, ...] ) ] table_expression

wherein the `select_expression` or `value_expression` have been replaced with [`table_expression`](#table-expression). [Recall](#table-expression) that a `select_expression` or a `value_expression` is a `table_expression` as well; so the standard syntax continues to be valid. 

The optional `column_name` explicitly lists the table columns; each column in the list must be present in the table schema. If this list is not specified, it is taken to be the list of columns in the schema of the table `table_name`, in the order in which they were listed while creating the table. Given this explicit/implicit list, the following should hold:

* It must have the same number of columns as the result of the `table_expression`.
* For each column in the list, the type of a column in the table `table_name` must be compatible with the type of the corresponding (i.e. at the same position) column in the result.

If the `column_name` list is explicitly specified, any column in the table `table_name` which does not appear in the list gets a `NULL` for every row inserted.

This statement is restricted in comparison, but compliant with the SQL standard and [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-insert.html).

### Updating Rows in a Base Table
The `UPDATE` statement updates the value of one or more columns in rows in a base table.

The syntax supported in Sclera is as follows:

    UPDATE [location_name.]table_name
    SET { column_name = scalar_expression } [, ...]
    [ WHERE condition ]

Here, `table_name` is the name of the table to be updated. The table must exist at the location `location_name`.  The `location_name` can be omitted if `table_name` is unique across locations.

The optional `WHERE` clause specifies the `condition` (a boolean-valued [`scalar_expression`](#scalar-expressions)) that a row in the table must satisfy to be eligible for the update; rows that do not satisfy this condition are left unchanged. If the `WHERE` clause is not present, the update applies to all rows in the table.

The update to a row is specified by the `SET` clause, which contains a comma-separated list of "assignments". Each assignment specifies that given an eligible row, the column with the name specified by `column_name` should get the value obtained by evaluation of the `scalar_expression` on that row. The update proceeds by executing the specified list of assignments on each eligible row.

This syntax is compatible with the SQL standard, but is a restricted version of the syntax supported in [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-update.html).

### Deleting Rows in a Base Table
The `DELETE` statement deletes rows in a base table.

The syntax supported in Sclera is as follows:

    DELETE FROM [location_name.]table_name
    [ WHERE condition ]

Here, `table_name` is the name of the table from which rows are to be deleted. The table must exist at the location `location_name`.  The `location_name` can be omitted if `table_name` is unique across locations.

The optional `WHERE` clause specifies the `condition` (a boolean-valued [`scalar_expression`](#scalar-expressions)) that a row in the table must satisfy to be eligible for deletion; rows that do not satisfy this condition are left unchanged. If the `WHERE` clause is not present, all rows in the table are deleted.

This syntax is a restricted version of that supported in the SQL standard and [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-delete.html).

### Dropping Base Tables
The `DROP TABLE` statement drops a base table from Sclera as well as the underlying database system.

This statement is similar to the [`REMOVE TABLE` statement](/doc/ref/dbms#removing-database-tables). However, `REMOVE TABLE` only removes the metadata of the named table from Sclera; it does not drop the table from the underlying data store. `DROP TABLE` remove the table's metadata from Sclera and also drops the table from the underlying datastore.

The syntax supported in Sclera is as follows:

        DROP TABLE [location_name.]table_name

Here, `table_name` is the name of the table to be dropped. The table must exist at the location `location_name`.  The `location_name` can be omitted if `table_name` is unique across locations.

This syntax is a restricted version of that supported in the SQL standard and [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-droptable.html).

## Views
A view is a SQL query with the appearance of a *read-only* table. A view can be used as a base table in any [query](#table-expression). But, unlike a base table, a view does not contain any real data; it is just a placeholder for the associated query.

### Creating Views
The standard `CREATE VIEW` statement has the following syntax:

    CREATE [ TEMP | TEMPORARY ] VIEW view_name AS select_expression

Sclera generalizes the [`select_expression`](#select-expression) to a [`table_expression`](#table-expression), resulting in:

    CREATE [ TEMP | TEMPORARY ] VIEW view_name AS table_expression

This statement creates a view with the specified `view_name` and associates it with the query given by `table_expression`. Notice the similarity with the [`CREATE TABLE AS` statement](#creating-tables-with-query-results).

The optional `TEMPORARY` or `TEMP` modifier creates a view that exists for the duration of the Sclera session only. When the session ends, the view is deleted and its metadata is removed from Sclera's metadata store.

Unlike a base table, a view is not attached to a [location](/doc/ref/dbms#location); this is because the underlying query (specified by the `table_expression` above) can span multiple locations.

This statement is restricted in comparison, but compliant with the SQL standard and [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-createview.html).

### Dropping Views
The `DROP VIEW` statement drops a view from Sclera.

The syntax supported in Sclera is as follows:

    DROP VIEW view_name

Here, `view_name` is the name of the view to be dropped.

This syntax is compatible with the SQL standard and is a restricted version of that supported in [PostgreSQL](http://www.postgresql.org/docs/current/static/sql-dropview.html).
