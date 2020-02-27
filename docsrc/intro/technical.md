At its core, Sclera contains a [SQL processor](#sql-processor), which works with a [metadata store](#metadata-store), and a [cache store](#cache-store) to process the SQL statements.

## SQL Processor
The SQL processor evaluates the SQL [queries](#query-processor) and [commands](#command-processor) submitted to Sclera. The evaluation is done by translating them to subcommands/subqueries that are executed in turn on the underlying database system where the data actually resides.

The challenge is to coordinate the multiple database systems in such a way that the end user is not aware of the heterogeneity, getting an experience no different from working on a single system with all the data and the analytics capabilities built in.

### Query Processor
The query processor is responsible for compiling and evaluating the input query, broadly following the following steps:

#### Parsing
The query is parsed into an operator plan. This plan contains the details of the base data and the relational operations to be performed on that data to get the final result.

The data source can be a [table in a linked datastore](../setup/dbms.md#importing-database-tables), or a [text](../sclerasql/sqlextdataaccess.md#sclera-textfile) or [CSV](../sclerasql/sqlextdataaccess.md#sclera-csv) file on the disk, or a web service. The operations can be standard *relational operators* (e.g. filters, project, join, group-by aggregation), or *extension operators* (e.g. classification, clustering, entity extraction).

#### Optimization
A query optimizer rewrites the plan by reordering the operations to make it more efficient.

For instance, the optimizer pushes down evaluation of relational operators to the underlying data sources the extent possible. Relational operators on base tables in a datastore are marked to be executed on that datastore as a part of fetching the data. This is more efficient than fetching just the base data and evaluating the operators externally.

#### Evaluation
Finally, the query processor evaluates the optimized plan. This is done in two steps.

- First, the operator plan is converted to an evaluation plan; this replaces the operators with (a) evaluation operators (to be executed by Sclera's streaming evaluation engine) (b) expression evaluators, which are converted into queries and evaluated on an underlying system, or (c) materializers, which materialize a stream or an expression into a (temporary) table on an underlying system.

- Second, the evaluation plan is evaluated in a pipeline and the result is passed to the consumer via the appropriate interface (JDBC library or command line shell).

##### Relational Operators
Let us consider single-input relational operators first. If the input to the relational operator is already present in a datastore, the operator is evaluated as a part of the query that fetches the data. Otherwise, the operator is evaluated by Sclera's streaming evaluation engine -- the only exception is the sort operator, in which case the input is materialized in the [cache store](#cache-store) and sorted as a part of the query that fetches the result.

Planning of multi-input operators is more involved. We take the example of `JOIN`.

[Recall](../sclerasql/sqlregular.md#from-table-join) that comma-separated `from_items` appear in the `FROM` clause are converted into a sequence of binary cross-products, which may later be converted into binary joins based on the predicates in the `WHERE` clause. In this section, we only consider planning of a join with two inputs.

The inputs to the join are planned recursively; after planning, each input is either an expression that is to be pushed down to an underlying system as a query for evaluation, or a data stream that is the either the result of prior in-memory computations, or ingested from an external source. Further, we assume if an input is a data stream, it is the left input -- if the left input is not a data stream, but the right input is, the join is rewritten to commute its inputs.

There are multiple cases:

*Case 1: Both inputs are data streams, the join is an inner or outer equi-join, and the input streams are sorted on the respective joining columns*

Sclera evaluates the join in its embedded engine using the merge-join algorithm.

*Case 2A: Left input is data stream, the join is an inner or outer equi-join, and the left input stream is sorted on its joining column*

The right input is evaluated with an `ORDER BY` on its joining column. This case then reduces to the *Case 1* above and is evaluated accordingly.

*Case 2B: Left input is data stream, the join is an inner or left-outer equi-join, and the left input stream is not sorted on its joining column*

The right input is materialized at source location (if not a base table), and indexed on its joining column. The left input is processed in batches, and each batch probes for the joining right-input rows using the index. This is an indexed nested loop join with the left input in the outer loop.

This evaluation strategy is chosen to avoid materializing the left input, which is assumed to be expensive (or impossible) to materialize. Note that a right-outer join cannot be evaluated using this strategy.

*Case 2C: Left input is data stream, but scenario not covered by the cases above*

The left input is materialized at the location of the right input (or the [cache store](#cache-store) if the right input is a stream). If right input is a stream, it is also materialized at the cache store. This reduces to the *Case 3A* below.

This the most inefficient scenario, but we think it is rare. Nevertheless, please be careful when joining streams without the appropriate sort order, especially when the join is a right-outer join.

*Case 3A: Neither input is a data stream, and both inputs are present at the same location*

In this case, the join expression is pushed down to the common location, and is computed by the underlying system.

*Case 3B: Neither input is a data stream, and the inputs are present at different locations*

This is a cross-system join. To evaluate a cross-system join, Sclera needs all the inputs to be present at a single location; let us call this the "target location" for the join. This target location is decided as follows:

- For each input, Sclera finds the location of the underlying data. These locations are the candidates for the target location, and are listed in the order of appearance of the corresponding `from_item` in the `FROM` clause. The list may contain duplicates.
- From this list, Sclera then removes the [cache store](#cache-store), if present, as well as the ["read-only" locations](../setup/dbms.md#read-write-versus-read-only-mode).
- If the list is empty, Sclera assigns the [cache store](#cache-store) as the target location. This has the effect that cross-system joins across multiple read-only locations are evaluated by moving all the data to the cache store; the join is then computed at the cache store.
- If the list is not empty, Sclera assigns the location of the left input the target location. This has the effect that all the data from locations other than the assigned target location is moved to the target location, where the join is then computed.

The ordering of the `from_item`s in a `FROM` clause thus matters when evaluating cross-system joins. While this enables you to control how data is moved while evaluating a query, you need to pay special attention to this ordering -- especially when significant amounts of data needs to be transfered.

In any case, when evaluating a query with a cross-system join, please take a close look at the query's evaluation plan (obtained using the [`EXPLAIN` shell command](../interface/shell.md#compile-time-explain)) before submitting the query.

In the current version, Sclera moves data from a "source" to a "target" database system by reading in the data from the source and inserting it into a temporary table in the target. This transfer is done in a streaming (pipelined) manner wherever possible, to avoid reading the entire result in memory. This could be a bottleneck when large amounts of data (millions of rows) are transferred. More efficient data transfer mechanisms will be in place in later versions of Sclera.

##### Extension Operators
These operators are evaluated using external libraries available through a component.

If the input is not already available in memory (entirely, or as a stream/iterator from a datastore), it is fetched using the datastore's interface (e.g. JDBC/SQL for a RDBMS). The component then passes the input to the associated library (after appropriate transformations, if needed); the operator's result is then computed using the library's API.

An operator could be evaluated using multiple alternative [components](../setup/components.md). For instance, the ["classification"](../sclerasql/sqlextml.md#classification) operator could be evaluated using [WEKA](http://www.cs.waikato.ac.nz/ml/weka/) ([component: `sclera-weka`](../setup/components.md#sclera-weka)), or any other machine learning plugin. The specific library/component used can be enforced by the query, or through defaults in the configuration. See the [SQL documentation](../sclerasql/sqlextml.md#extended-syntax-for-using-specific-libraries-and-algorithms) for details.

Note that the way the input is prepared and/or the result is obtained could be very different for different libraries. Without Sclera, moving from one library to an alternative library with similar functionality would be a messy "porting" job. With Sclera, all that complexity is taken care of under the hood.

### Command Processor
The command processor is responsible for executing the non-query commands, such as creating tables, inserting, updating and deleting data.

A command may or may not have an embedded query. If it does, Sclera makes use of the [query processor](#query-processor) discussed above to plan and execute the query, and translates the statement to work with the final result (details left out for brevity).

In either case, Sclera interfaces with the underlying systems' APIs to get the task done. For instance, to create a table when the underlying system is a NoSQL datastore, Sclera makes use of the appropriate API functions to create the structure. Similar translation happens when inserting, updating or deleting rows, and so on.

<a class="anchor" name="metadata-store"></a>
## Schema Store
The schema store contains the metadata that enables the [SQL processor](#sql-processor) to plan the SQL statements for execution on the underlying systems. This metadata includes:

- The connection parameters for every [database system that is connected to Sclera](../setup/dbms.md#connecting-to-database-systems).
- The schema of the [tables imported from the connected database systems](../setup/dbms.md#importing-database-tables)
- Specification of the [virtual tables](../sclerasql/sqlregular.md#creating-views)

By default, an embedded [H2 database](http://www.h2database.com) is used as a data store. This default can be changed by modifying the [configuration](../setup/configuration.md#sclera-location-schema-database).

## Cache Store
The cache store is used by the [query processor](#query-processor) for evaluationg relational operators on intermediate results. By default, an embedded [H2 main-memory database](http://www.h2database.com) is used as a cache data store. This default can be changed by modifying the [configuration](../setup/configuration.md#sclera-location-datacache).
