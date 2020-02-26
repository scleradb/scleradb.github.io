Ordered data is data that has an underlying sort order -- examples include website clickstreams, tweets, stock tickers, system logs, and so on. The sort order is typically time (in which case, we also call it a [time series](http://en.wikipedia.org/wiki/Time_series)), but other orders are also possible. [Such data is hard to analyze in standard SQL.](http://kx.com/news/in-the-news/sql-timeseries.php)

Sclera provides expressive constructs to efficiently and flexibly analyze ordered data. As we shall see, these constructs are far more powerful than [SQL window functions](http://www.postgresql.org/docs/current/static/functions-window.html), and are also much simpler and intuitive to use; they not only obviate the use of expensive self-joins and nested queries in a majority of queries, but also enable queries that so far were really hard to express in standard SQL.

Since evaluation of the constructs discussed in this section takes a single pass over the data and [uses a bounded amount of memory](#scalability) irrespective of the number of rows processed (which means that no intermediate results are materialized), these constructs can be readily applied over streaming data as well.

We describe these constructs in the sections below, and also illustrate their utility using a number of use cases.

- We first present [the constructs that enable queries to access the prior rows of a base table or intermediate result, and define running aggregates over them](#accessing-history-without-self-joins). This generalizes the [window functions](http://www.postgresql.org/docs/current/static/functions-window.html) of standard SQL.
- We then generalize the above, and show [how to use regular expressions to form aggregation groups within a base table or an intermediate result](#pattern-matching-with-match), based on column values along with positional constraints. This generalizes the [window functions](http://www.postgresql.org/docs/current/static/functions-window.html) as well as the [`GROUP BY` clause](../sclerasql/sqlregular.md#group-by-clause) of SQL.

## Accessing History Without Self-Joins

Most operations on ordered data involve current as well as prior rows in a stream. SQL is good at accessing the current row, not so much in accessing the prior rows.

Consider ticker data for a stock, one row per day, and say we need to find the opening prices for the days where the closing price in the previous session was  5% or more above the average closing price so far.

In Sclera, this query can be specified as:

    SELECT T.ts, T.openprice
    FROM (ticker ORDER BY ts) T
    WHERE T[-1].closeprice >= T.avg(closeprice) * 1.05

The same query can be specified in standard SQL (not supported in Sclera) as:

    SELECT T.ts, T.openprice
    FROM ticker T WINDOW w AS (ORDER BY ts)
    WHERE LAG(closeprice) OVER w >= AVG(closeprice) OVER w * 1.05

In standard SQL, we only get "first class" access to `T`'s current row -- the earlier rows, and running aggregates across them, are available only through specialized functions. Also, the sort order of the rows in `T` is specified in a WINDOW clause that is separate from `T`.

In Sclera, we get first class access to `T`'s current row, as well as all prior rows, through an optional index notation that is baked into the SQL syntax. Also, the running aggregates are associated with `T` which represents both the sequence of rows and their sort order. This leads to simpler and more readable queries, as is apparent from the example above. These constructs are described in further detail below.

<a class="anchor" name="indexed-alias"></a>
### Table Alias as an Array of Rows

Recall that a table alias in a standard SQL query identifies a [table expression](../sclerasql/sqlregular.md#table-expression) in the [`FROM` clause](../sclerasql/sqlregular.md#from-clause).

In standard SQL, each column reference is explicitly or [implicitly](../sclerasql/sqlregular.md#column-reference) associated with a table alias. Operationally, the table alias stands for the *current* row of the result of the [table expression](../sclerasql/sqlregular.md#table-expression) identified by the alias (hereafter called the intermediate result associated with the table alias).

Sclera generalizes the use of table alias by introducing an optional index to retrieve prior rows for the associated intermediate result. In other words, the history is considered a list, starting at the first emitted row, and ending at the previous emitted row.

For instance, the following query returns, for each input row, the difference of the a day's openprice and the previous closeprice.

    SELECT T.closeprice - T[-1].openprice
    FROM (ticker ORDER BY ts) T

The use of negative indexes as offsets from the end of the list is borrowed from [Python](http://www.openbookproject.net/thinkcs/python/english2e/ch09.html#accessing-elements).

For the "previous row" of `T` to be well defined in the above query, the intermediate result associated with `T` needs to be ordered -- since the data here is coming from a base table on disk, an `ORDER BY ts` is needed; this is not needed if the [input is already sorted](../sclerasql/sqlextdataaccess.md).

<a class="anchor" name="indexed-alias-syntax"></a> The index is optional, and can be an arbitrary integer. Specifically, given the table alias `T`:

- `T` represents the current row (this is consistent with [standard SQL](../sclerasql/sqlregular.md#column-reference))
- `T[0]` is the first row, `T[1]` is the second row, and so on.
- `T[-1]` is the previous row, `T[-2]` is one previous to that, and so on.

If the index is out of range, the corresponding values are taken as `NULL`.

<a class="anchor" name="indexed-alias-example"></a> As an example, consider the table `FOO`:

    > FOO;
    ---+--- 
     A | B  
    ---+--- 
     1 | X  
     2 | Y  
     3 | Z  
    ---+--- 
    (3 rows)

Then, the query

    > SELECT T.A AS A,
             T[-2].B AS PPREVB, T[-1].B AS PREVB, T.B AS B,
             T[0].B AS FIRSTB, T[1].B AS SECONDB, T[2].B AS THIRDB
      FROM (FOO ORDER BY A) T;

gives the following result:

    ---+--------+-------+---+--------+---------+--------
     A | PPREVB | PREVB | B | FIRSTB | SECONDB | THIRDB
    ---+--------+-------+---+--------+---------+--------
     1 | NULL   | NULL  | X | X      | NULL    | NULL  
     2 | NULL   | X     | Y | X      | Y       | NULL  
     3 | X      | Y     | Z | X      | Y       | Z     
    ---+--------+-------+---+--------+---------+--------
    (3 rows)

In the above:

- The column `B` is the input column `B`.
- The column `PREVB` is the input column `B` with a lag of one row.
- The column `PPREVB` is the input column `B` with a lag of two rows.
- The column `FIRSTB` contains the value of `B` in the first input row.
- The column `SECONDB` contains the value of `B` in the second input row. It is `NULL` when the input so far has less than two rows.
- The column `THIRDB` contains the value of `B` in the third input row. It is `NULL` when the input so far has less than three rows.

The indexed table aliases can be used in any expression in the scope of the alias -- in `SELECT`, `WHERE`, `GROUP BY`, `HAVING` or `ORDER BY` clauses -- they are accepted wherever the regular table alias is in standard SQL.

### Running Aggregates

The table alias can also be used to retrieve the running aggregates over the result of the table or subquery associated with the alias.

As an example, again consider the table `FOO`:

    > FOO;
    ---+--- 
     A | B  
    ---+--- 
     1 | X  
     2 | Y  
     3 | Z  
    ---+--- 
    (3 rows)

The query

    > SELECT T.A AS A, T.SUM(A) AS SUMA
      FROM (FOO ORDER BY A) T;

gives the following result:

    ---+------
     A | SUMA
    ---+------
     1 | 1   
     2 | 3 
     3 | 6
    ---+------
    (3 rows)

Formally, the syntax of the running aggregate is:

    table_alias.aggregate_function ( aggr_params )

where:

- `table_alias` identifies a [table expression](../sclerasql/sqlregular.md#table-expression) in the [`FROM` clause](../sclerasql/sqlregular.md#from-clause)
- `aggregate_function` is an [aggregate function](../sclerasql/sqlmisc.md#aggregate-functions)
- `aggr_params` is a comma-separated list of [scalar expressions](../sclerasql/sqlregular.md#scalar-expressions), all of whose column references are contained in the result of `table_alias`.

### Indexed Table Alias and Running Aggregates on Partitioned Input

The input rows can be partitioned by specifying a set of columns in a `PARTITION BY` clause. The history is then maintained independently for each partition.

Given the table alias `T`:

- `T` represents the current row, as [earlier](#indexed-alias-syntax) and consistent with [standard SQL](../sclerasql/sqlregular.md#column-reference).
- `T[0]` is the first row with the same values of the partition columns as the current row, `T[1]` is the second such row, and so on.
- `T[-1]` is the previous row with the same values of the partition columns as the current row, `T[-2]` is such a row previous to that, and so on.

As an example, consider the table `BAR`:

    > BAR;
    ---+---+---
     A | B | C 
    ---+---+---
     1 | X | 0
     2 | Z | 1
     3 | Y | 0
     5 | Z | 0
     4 | Y | 1
     6 | X | 1
    ---+---+---
    (6 rows)

Then, the query

    > SELECT T.A AS A, T.C AS C,
             T[-2].B AS PPREVB, T[-1].B AS PREVB, T.B AS B,
             T[0].B AS FIRSTB, T[1].B AS SECONDB, T[2].B AS THIRDB
      FROM (BAR ORDER BY A) T PARTITION BY C;

gives the following result:

    ---+---+--------+-------+---+--------+---------+--------
     A | C | PPREVB | PREVB | B | FIRSTB | SECONDB | THIRDB
    ---+---+--------+-------+---+--------+---------+--------
     1 | 0 | NULL   | NULL  | X | X      | NULL    | NULL  
     2 | 1 | NULL   | NULL  | Z | Z      | NULL    | NULL  
     3 | 0 | NULL   | X     | Y | X      | Y       | NULL  
     4 | 0 | X      | Y     | Z | X      | Y       | Z     
     5 | 1 | NULL   | Z     | Y | Z      | Y       | NULL  
     6 | 1 | Z      | Y     | X | Z      | Y       | X     
    ---+---+--------+-------+---+--------+---------+--------
    (6 rows)

This result is computed using the same logic as [earlier](#indexed-alias-example), but separately for the two partitions identified by `C = 0` and `C = 1` respectively.

Similarly, running aggregates on a partitioned input are computed independently for each partition.

The query

    > SELECT T.A AS A, T.C AS C, T.SUM(A) AS SUMA
      FROM (BAR ORDER BY A) T PARTITION BY C;

gives the following result:

    ---+---+------
     A | C | SUMA
    ---+---+------
     1 | 0 | 1
     2 | 1 | 2
     3 | 0 | 4
     4 | 0 | 8
     5 | 1 | 7
     6 | 1 | 13
    ---+---+------
    (6 rows)

Partitioning is useful, for instance, when we need to analyze rows for a particular visitor in a multi-visitor clickstream.

## Pattern Matching with `MATCH`
Regular expressions are commonplace mechanism for text search, supported almost all text editors. Standard SQL uses regular expressions in the [SIMILAR TO](http://www.postgresql.org/docs/9.0/static/functions-matching.html#FUNCTIONS-SIMILARTO-REGEXP) operator, used to match patterns over string values.

In Sclera, regular expressions can also be used to match, query and aggregate a sequence of rows. The idea is to see a column of the input as a sequence of symbols -- just like text -- and match regular expression over this sequence. This allows us to "parse" the input sequence of rows and work with them at a granularity that is extremely tough to emulate in standard SQL.

The regular expression is matched progressively with the incoming sequence of tuples. Specifically, a match occurs at a row when a segment of rows upto that row match the regular expression. Multiple segments upto the row may match the regular expression -- in this case, the longest matching segment is taken as the match and the others are ignored.

As soon as a match occurs, a row is emitted to the output. The output rows are constructed by evaluating [aggregates](../sclerasql/sqlmisc.md#aggregate-functions) on the subsequence of rows matching the respective labels in the regular expression.

We illustrate with a number of examples.

### Pattern Matching Examples
<a class="anchor" name="clicks"></a> Consider the clickstream data for an e-commerce site. For simplicity, we assume that the data is for a single visitor (we will remove this constraint [soon](#match-on-partitioned-input)). The data, simplified to focus on the ideas below, is as follows:

    > clicks;
    -----------+----------
     VISITTIME | PAGETYPE 
    -----------+----------
     10:21:03  | login    
     10:22:09  | search   
     10:24:39  | prodview 
     10:27:14  | logout   
     11:01:22  | login    
     11:02:33  | prodview 
     11:04:09  | search   
     11:05:47  | prodview 
     11:07:19  | checkout 
     11:09:51  | prodview 
     11:13:21  | logout   
    -----------+----------
    (11 rows)

For this data, let us define a "session" as the segment starting at a "login" upto the following "logout". This is an extremely simplified clickstream -- but this simplification helps us to focus on the features being discussed in the example queries below.

<a class="anchor" name="example-1"></a> **Example 1**
Suppose we want to add a new column `sessionstart` to each row, containing the start time of the session to which that row belongs.

This is a straightforward task -- but in standard SQL, will need a complex query with nested subqueries. With Sclera, the query is as simple as:

    > SELECT visittime, pagetype, login.visittime AS sessionstart
      FROM (clicks ORDER BY visittime) MATCH "login.(prodview | search | checkout | logout)*" ON pagetype

The query progressively matches the regular expression `login.(prodview | search | checkout | logout)*` against the sequence of values in the column `pagetype`, and emits a row for each match. Note that this regular expression matches every prefix of a session starting at its "login" row. Every row in a session thus gets associated with the first row (i.e. the "login" row) of the session. The unqualified column `visittime` and `pageType` get values from the last row of the match, while `login.visittime` gets the value from the row that matches `login` in the match.

Notice how the symbols in the regular expression also assume the role of a `table_alias` representing the subsequence of rows that match that symbol.

The result of the query is, as expected:

    -----------+----------+--------------
     VISITTIME | PAGETYPE | SESSIONSTART 
    -----------+----------+--------------
     10:21:03  | login    | 10:21:03  
     10:22:09  | search   | 10:21:03  
     10:24:39  | prodview | 10:21:03  
     10:27:14  | logout   | 10:21:03  
     11:01:22  | login    | 11:01:22  
     11:02:33  | prodview | 11:01:22  
     11:04:09  | search   | 11:01:22  
     11:05:47  | prodview | 11:01:22  
     11:07:19  | checkout | 11:01:22  
     11:09:51  | prodview | 11:01:22  
     11:13:21  | logout   | 11:01:22  
    -----------+----------+--------------
    (11 rows)

The `ON pagetype` qualifier in the query above specifies that the values in the column `pagetype` are to be used used to label the input rows. This means that if we want a (sub)expression specifying "all labels expect `login`", as in the above queries, we need to know all the values the column `pagetype` can possibly take -- in a real scenario, this might be a large set of values, might not be available, or might be hard to compute.

One workaround is to use a [`CASE` expression](http://www.postgresql.org/docs/current/static/functions-conditional.html#FUNCTIONS-CASE) to generate the labels and apply `MATCH` on the generated labels. The query can be rewritten as:

    > SELECT visittime, pagetype, login.visittime AS sessionstart
      FROM (
        SELECT visittime, pagetype,
               CASE pagetype WHEN "login" THEN "login" ELSE "other" END AS pagelabel
        FROM clicks
        ORDER BY visittime
      ) MATCH "login.other*" ON pagelabel;

Alternatively, we can use the extended `ON` clause, as follows:

    > SELECT visittime, pagetype, login.visittime AS sessionstart
      FROM (clicks ORDER BY visittime)
           MATCH "login.other*"
           ON pagetype WHEN "login" THEN "login" ELSE "other";

The extended `ON` clause performs the same function as the `CASE` statement above for this query; [later](#example-4), we will see that it is a bit more general.

The same idea can be applied to match over numeric fields -- generate discrete-valued labels based on a condition and match over the generated labels.

<a class="anchor" name="example-2"></a> **Example 2**
In this example, we want to associate the sequence number of the session with each row, with the first session as `1`, the second session as `2`, and so on. In Sclera, we can easily compute this by matching all the sessions in a single regular expression, and assigning the session identifier as the number of logins seen so far.

The query is as follows:

    > SELECT visittime, pagetype, login.count() AS sessionseq
      FROM (clicks ORDER BY visittime)
           MATCH "(login.other*)+"
           ON pagetype WHEN "login" THEN "login" ELSE "other";

and the result is:

    -----------+----------+------------
     VISITTIME | PAGETYPE | SESSIONSEQ 
    -----------+----------+------------
     10:21:03  | login    | 1          
     10:22:09  | search   | 1          
     10:24:39  | prodview | 1          
     10:27:14  | logout   | 1          
     11:01:22  | login    | 2          
     11:02:33  | prodview | 2          
     11:04:09  | search   | 2          
     11:05:47  | prodview | 2          
     11:07:19  | checkout | 2          
     11:09:51  | prodview | 2          
     11:13:21  | logout   | 2          
    -----------+----------+------------
    (11 rows)

<a class="anchor" name="example-3"></a> **Example 3**
In this example, we want to find, for each session in which the visitor does a "checkout", the number of product views by a visitor between the "login" and the "checkout". Again, the standard SQL will be extremely complex. In Sclera, the query works out to far simpler:

    > SELECT login.visittime as sessionstart, prodview.count() AS prodviews
      FROM (clicks ORDER BY visittime)
      MATCH "login.(prodview | search)*.checkout" ON pagetype;

As in the previous query, this query progressively matches the regular expression `login.(prodview | search)*.checkout` against the sequence of values in the column `pagetype`. This regular expression, however, only matches segments starting at a "login" row and ending at a "checkout" row. Each match associates each symbol in the regular expression with the subsequence of rows matching the symbol; the symbol becomes the table alias for the associated rows, and can be [used to aggregate the rows as with regular table aliases](#running-aggregates). The expression `prodview.count()` thus counts the "prodview" rows that lie between the "login" row and the "checkout" row.

The result is:

    --------------+-----------
     SESSIONSTART | PRODVIEWS 
    --------------+-----------
     11:01:22     | 2         
    --------------+-----------
    (1 row)

Since we ignore the "search" rows, we can eliminate them in the input, getting the equivalent query:

    > SELECT login.visittime as sessionstart, prodview.count() AS prodviews
      FROM (clicks WHERE pagetype <> "search" ORDER BY visittime)
      MATCH "login.prodview*.checkout" ON pagetype;

<a class="anchor" name="example-4"></a> **Example 4**
Sometimes, we may need the same value in the column being matched to be represented by multiple symbols in the regular expression -- this is needed to differentiate rows with the same value of the column based on their positional context.

In this example, we want to know the number of product views before and after a checkout within a session.

Since the query differentiates rows with the same value of the matched column based on the positional context, we need a mechanism to associate multiple symbols with rows. This is achieved by the extended `ON` clause used earlier in [Example 1](#example-1), and is illustrated in the query below:

    > SELECT login.visittime as sessionstart,
             before.count() AS viewsbefore, after.count() as viewsafter
      FROM (clicks ORDER BY visittime)
           MATCH "login.(before | search)*.checkout.(after | search)*.logout"
           ON pagetype WHEN "prodview" THEN ("before", "after");

The `WHEN` clause specifies "before" and "after" as labels for "prodview" rows (the label "prodview" is not recognized anymore). The other rows retain their default label.

In a match, the "prodview" rows appearing before a checkout row get assigned the label "before", and "prodview" rows appearing after a checkout row get assigned the label "after". As earlier, these labels can be used to aggregate upon the associated rows -- thus, `before.count()` is the number of product views between "login" and "checkout", while `after.count()` is the number of product views between "checkout" and "logout".

The result of the query is:

    --------------+-------------+------------
     SESSIONSTART | VIEWSBEFORE | VIEWSAFTER 
    --------------+-------------+------------
     11:01:22     | 2           | 1          
    --------------+-------------+------------
    (1 row)

Again, since "search" rows are ignored in the above query, we can safely eliminate them in the input, getting the equivalent query:

    > SELECT login.visittime as sessionstart,
             before.count() AS viewsbefore, after.count() as viewsafter
      FROM (clicks WHERE pagetype <> "search" ORDER BY visittime)
           MATCH "login.before*.checkout.after*.logout"
           ON pagetype WHEN "prodview" THEN ("before", "after");

<a class="anchor" name="example-5"></a> **Example 5**
Sclera also supports start and end anchors in the regular expression. The start-anchor restricts the match to a prefix of the input row sequence, while the end-anchor restricts the match to a suffix of the input row sequence.

The following query returns the number of searches, across all sessions, before the first checkout by a visitor:

    > SELECT search.count() AS searches
      FROM (clicks WHERE pagetype in ("search", "checkout") ORDER BY visittime)
           MATCH "^search*.checkout" ON pagetype;
    ----------
     SEARCHES 
    ----------
     2        
    ----------
    (1 row)

Similarly, the following query returns the number of searches, across all sessions, after the last checkout by a visitor:

    > SELECT search.count() AS searches
      FROM (clicks WHERE pagetype in ("search", "checkout") ORDER BY visittime)
           MATCH "checkout.search*$" ON pagetype;
    ----------
     SEARCHES 
    ----------
     0        
    ----------
    (1 row)

Note that a query with an end-anchor can only emit the output after consuming all the input.

The two anchors can be used together. The following query counts the non-search rows before and after the last search.

    > SELECT before.count() AS viewsbefore, after.count() as viewsafter
      FROM (clicks ORDER BY visittime)
           MATCH "^(before | search)*.search.after*$"
           ON pagetype WHEN "search" THEN "search" ELSE ("before", "after");
    -------------+------------
     VIEWSBEFORE | VIEWSAFTER 
    -------------+------------
     5           | 4          
    -------------+------------
    (1 row)

If we want to include the counts for "search" rows as well in `before`, we can use a generalized qualifier:

    > SELECT LABEL(before, search).count() AS viewsbefore, after.count() as viewsafter
      FROM (clicks ORDER BY visittime)
           MATCH "^(before | search)*.search.after*$"
           ON pagetype WHEN "search" THEN "search" ELSE ("before", "after");
    -------------+------------
     VIEWSBEFORE | VIEWSAFTER 
    -------------+------------
     7           | 4          
    -------------+------------
    (1 row)

Of course, since the labels "search" and "before" are mutually exclusive, we could also have added `before.count()` and `search.count()` to get the same result.

    > SELECT before.count() + search.count() AS viewsbefore, after.count() as viewsafter
      FROM (clicks ORDER BY visittime)
           MATCH "^(before | search)*.search.after*$"
           ON pagetype WHEN "search" THEN "search" ELSE ("before", "after");
    -------------+------------
     VIEWSBEFORE | VIEWSAFTER 
    -------------+------------
     7           | 4          
    -------------+------------
    (1 row)

If we want the symbol before to include the searches before the last, we can say:

    > SELECT before.count() AS viewsbefore, after.count() as viewsafter
      FROM (clicks ORDER BY visittime)
           MATCH "^before*.search.after*$"
           ON pagetype WHEN "search" THEN ("before", "search") ELSE ("before", "after");
    -------------+------------
     VIEWSBEFORE | VIEWSAFTER 
    -------------+------------
     6           | 4          
    -------------+------------
    (1 row)

The difference with the previous query is that the last search is not included in the `VIEWSBEFORE` above.

<a class="anchor" name="example-6"></a> **Example 6**
Sclera also supports wild cards using the `ALL` clause in the extended `ON` syntax.

The following query outputs, for each "search" row, the pagetype of previous and next rows, if any.

    > SELECT search.visittime, prevpg.pagetype AS prevtype, nextpg.pagetype AS nexttype
      FROM (clicks ORDER BY visittime)
           MATCH "prevpg.search.nextpg" ON pagetype ALL(prevpg, nextpg)
    -----------+----------+----------
     VISITTIME | PREVTYPE | NEXTTYPE 
    -----------+----------+----------
     10:22:09  | login    | prodview 
     11:04:09  | prodview | prodview 
    -----------+----------+----------
    (2 rows)

The `ALL` labels apply to all rows, in addition to the labels assigned by the `ON`/`WHEN`/`THEN`/`ELSE` clauses.

For instance, in the last query in the [previous example](#example-5), notice that the symbol "before" was assigned to all rows. Rather than include "before" in all `THEN` and `ELSE` lists, we can say:

    > SELECT before.count() AS viewsbefore, after.count() as viewsafter
      FROM (clicks ORDER BY visittime)
           MATCH "^before*.search.after*$"
           ON pagetype WHEN "search" THEN "search" ELSE "after" ALL "before";
    -------------+------------
     VIEWSBEFORE | VIEWSAFTER 
    -------------+------------
     6           | 4          
    -------------+------------
    (1 row)

### `MATCH` on Partitioned Input
The input can be partitioned on one or more columns before applying `MATCH`, using a `PARTITION BY` clause. The regular expression is then matched independently on the rows within each partition.

<a class="anchor" name="vclicks"></a> The examples in the [previous section](#pattern-matching-with-match) were based on clickstream data of a single visitor. Let us now assume that the data has multiple visitors. In the following table a new column `visitorid` contains the identifier of the visitor for each row.

    > vclicks;
    -----------+-----------+----------
     VISITORID | VISITTIME | PAGETYPE 
    -----------+-----------+----------
     1         | 10:21:03  | login    
     1         | 10:24:39  | prodview 
     1         | 10:27:14  | logout   
     2         | 10:21:04  | login    
     2         | 10:22:10  | search   
     2         | 10:27:15  | logout   
     2         | 11:01:22  | login    
     1         | 11:01:23  | login    
     2         | 11:02:33  | prodview 
     2         | 11:04:10  | search   
     2         | 11:05:47  | prodview 
     1         | 11:05:48  | prodview 
     2         | 11:07:19  | checkout 
     2         | 11:09:52  | prodview 
     2         | 11:13:21  | logout   
     1         | 11:13:22  | logout   
    -----------+-----------+----------
    (16 rows)

For instance, the query in [Example 1](#example-1) can be rewritten as:

    > SELECT visitorid, visittime, pagetype, login.visittime AS sessionstart
      FROM (vclicks ORDER BY visittime) PARTITION BY visitorid
           MATCH "login.other*"
           ON pagetype WHEN "login" THEN "login" ELSE "other";
    -----------+-----------+----------+--------------
     VISITORID | VISITTIME | PAGETYPE | SESSIONSTART 
    -----------+-----------+----------+--------------
     2         | 10:21:04  | login    | 10:21:04     
     1         | 10:21:03  | login    | 10:21:03     
     1         | 10:24:39  | prodview | 10:21:03     
     2         | 10:22:10  | search   | 10:21:04     
     2         | 10:27:15  | logout   | 10:21:04     
     1         | 10:27:14  | logout   | 10:21:03     
     2         | 11:01:22  | login    | 11:01:22     
     2         | 11:02:33  | prodview | 11:01:22     
     2         | 11:04:10  | search   | 11:01:22     
     1         | 11:01:23  | login    | 11:01:23     
     2         | 11:05:47  | prodview | 11:01:22     
     2         | 11:07:19  | checkout | 11:01:22     
     2         | 11:09:52  | prodview | 11:01:22     
     1         | 11:05:48  | prodview | 11:01:23     
     2         | 11:13:21  | logout   | 11:01:22     
     1         | 11:13:22  | logout   | 11:01:23     
    -----------+-----------+----------+--------------
    (16 rows)

The evaluation now builds a `MATCH` processor for each unique `visitorid` -- each processor only sees the rows that belong to the same `visitorid`. The memory overhead thus depends upon the number of unique visitors.

If the data contains a large number of visitors, it may be more efficient to sort the input on (`visitorid`, `visittime`) -- this will ensure that only one visitor's data is processed at a time, making the memory overhead of `MATCH` independent of the number of input rows.

    > SELECT visitorid, visittime, pagetype, login.visittime AS sessionstart
      FROM (vclicks ORDER BY visitorid, visittime) PARTITION BY visitorid
           MATCH "login.other*"
           ON pagetype WHEN "login" THEN "login" ELSE "other";
    -----------+-----------+----------+--------------
     VISITORID | VISITTIME | PAGETYPE | SESSIONSTART 
    -----------+-----------+----------+--------------
     1         | 10:21:03  | login    | 10:21:03     
     1         | 10:24:39  | prodview | 10:21:03     
     1         | 10:27:14  | logout   | 10:21:03     
     1         | 11:01:23  | login    | 11:01:23     
     1         | 11:05:48  | prodview | 11:01:23     
     1         | 11:13:22  | logout   | 11:01:23     
     2         | 10:21:04  | login    | 10:21:04     
     2         | 10:22:10  | search   | 10:21:04     
     2         | 10:27:15  | logout   | 10:21:04     
     2         | 11:01:22  | login    | 11:01:22     
     2         | 11:02:33  | prodview | 11:01:22     
     2         | 11:04:10  | search   | 11:01:22     
     2         | 11:05:47  | prodview | 11:01:22     
     2         | 11:07:19  | checkout | 11:01:22     
     2         | 11:09:52  | prodview | 11:01:22     
     2         | 11:13:21  | logout   | 11:01:22     
    -----------+-----------+----------+--------------
    (16 rows)

### `MATCH` Syntax
This section introduced a new [table expression](../sclerasql/sqlregular.md#table-expression) with the following syntax:

    table_expression [ PARTITION BY ( partn_columns ) ] MATCH regular_expression [ON labeler]

where:

- <a class="anchor" name="table-expression"></a> `table_expression` is an arbitrary [table expression](../sclerasql/sqlregular.md#table-expression)
- `partn_columns` is an optional comma-separated list of columns in the result of `table_expression`.
    - When specified, the result of `table_expression` is partitioned on this set of columns; the matching happens independently on the rows within each partition.
- <a class="anchor" name="regular-expression"></a> `regular_expression`, with:
    - **alphabet:** the labels used by the [labeler](#labeler-syntax) to tag the input rows
    - **operators:** "`.`" (concatenation), "`|`" (disjunction), "`*`" (kleene star), "`+`" (kleene plus), and "`?`" (option).
        - Used to compose more complex regular expressions from the alphabet.
    - **anchors:** a start anchor "`^`" or an end anchor "`$`".
        - Required when we need only a prefix or a suffix match, respectively, and not all the matches as is the default.

Note that: 

- In the syntax above, the parenthesis on `partn_colums` can be omitted when the list is a singleton.
- <a class="anchor" name="default-labeler"></a> When the [`regular_expression`](#regular-expression) contains only one symbol, the labeler specification is optional; when the labeler is not specified, each input row will be labeled with the symbol in the regular expresion.

#### Labeler Syntax
The labeler tags each input row with a set of labels. The labeler is specified with a `label_col`, optional multiple `WHEN`/`THEN` clauses with an optional `ELSE` clause, and an optional `ALL` clause.

    label_col [ WHEN ( when_values ) THEN ( then_labels ) [ WHEN ... ] [ ELSE ( else_labels ) ] ] [ ALL ( all_labels ) ]

where:

- `label_col` is a column in the result of [`table_expression`](#table-expression) mentioned in the `MATCH` syntax above.
- `when_values` is a comma-separated list of values of the same SQL type as `label_col` (to be extra sure in case of numeric types, we can use explicit casts)
- `then_labels`, `else_labels` and `all_labels` are comma-separated lists of labels

In the syntax above, the parenthesis on lists can be omitted when the list is a singleton.

Consider a row, and let the value of the column `label_col` in the row be `X`. The set of labels that tag the row are determined as follows:

<a class="anchor" name="case1"></a> **Case 1: `ALL` clause is not present**

- When `WHEN`/`THEN`/`ELSE` clauses are not present, the row is labeled with `X`.
- If the `WHEN`/`THEN` clauses are present, and `X` is present in at least one `when_values` list, then the row is labeled by the union of the `then_labels` associated with the `when_values` that contain `X`.
- If the `WHEN`/`THEN` clauses are present, but `X` is not present in any `when_values`, then:
    - If `ELSE` clause is present, the row is labeled with the `else_labels`.
    - If `ELSE` clause is not present, the row is labeled with an empty set.

**Case 2: `ALL` clause is present**

- The row is labeled with the union of `all_labels` and the label set computed as in [Case 1](#case1) above.

## Accessing Matched Subsequences as an Array of Rows

Recall our [earlier discussion on how Sclera enables a table alias can be interpreted as an array of rows](#indexed-alias). The `MATCH` construct contains a generalization of the same -- we can interpret any symbol appearing in a regular expression (which represents a subsequence) as an array as well.

As a simple example, the query on the [table `clicks`](#clicks):

    > SELECT T.*, (T.visittime - T[0].visittime)::INT AS timediff
      FROM (clicks ORDER BY visittime) T;

can also be written as:

    > SELECT T.*, (T.visittime - T[0].visittime)::INT AS timediff
      FROM (clicks ORDER BY visittime) MATCH "T+";

In the first query, the table alias `T` is interpreted as an array over the entire prior sequence of input rows. In the second query, since the regular expression "`T+`" contains only one symbol `T`, the [default labeler](#default-labeler) labels each input row with `T`. The regular expression "`T+`" matches each non-empty prefix of the input sequence -- in other words, "`T+`" matches the entire history at any point. The two queries above are therefore equivalent, and have the output:

    -----------+----------+----------
     VISITTIME | PAGETYPE | TIMEDIFF 
    -----------+----------+----------
     10:21:03  | login    | 0        
     10:22:09  | search   | 66000    
     10:24:39  | prodview | 216000   
     10:27:14  | logout   | 371000   
     11:01:22  | login    | 2419000  
     11:02:33  | prodview | 2490000  
     11:04:09  | search   | 2586000  
     11:05:47  | prodview | 2684000  
     11:07:19  | checkout | 2776000  
     11:09:51  | prodview | 2928000  
     11:13:21  | logout   | 3138000  
    -----------+----------+----------
    (11 rows)

The array construct with `MATCH` becomes more useful when used along with more complex regular expressions. For instance, the following query retrieves the second visit (i.e. the row at index `1`) in each visitor in the [table `vclicks`](#vclicks).

    > SELECT T[1].*
      FROM (vclicks ORDER BY visittime) PARTITION BY visitorid MATCH "T+$";
    -----------+-----------+----------
     VISITORID | VISITTIME | PAGETYPE 
    -----------+-----------+----------
     2         | 10:22:10  | search   
     1         | 10:24:39  | prodview 
    -----------+-----------+----------
    (2 rows) 

Similarly, the following query retrieves the second-last visit (i.e. the row at index `-1`) in each visitor in the [table `vclicks`](#vclicks).

    > SELECT T[-1].*
      FROM (vclicks ORDER BY visittime) PARTITION BY visitorid MATCH "T+$";
    -----------+-----------+----------
     VISITORID | VISITTIME | PAGETYPE 
    -----------+-----------+----------
     2         | 11:09:52  | prodview 
     1         | 11:05:48  | prodview 
    -----------+-----------+----------
    (2 rows)

We can also have rows at multiple indexes juxtaposed in the same row.

    > SELECT T[1].*, T[-1].*
      FROM (vclicks ORDER BY visittime) PARTITION BY visitorid MATCH "T+$";
    -----------+-----------+----------+-------------+-------------+------------
     VISITORID | VISITTIME | PAGETYPE | VISITORID_1 | VISITTIME_1 | PAGETYPE_1 
    -----------+-----------+----------+-------------+-------------+------------
     2         | 10:22:10  | search   | 2           | 11:09:52    | prodview   
     1         | 10:24:39  | prodview | 1           | 11:05:48    | prodview   
    -----------+-----------+----------+-------------+-------------+------------
    (2 rows)

We can use arbitrary regular expressions on multiple symbols. Each symbol determines a subsequence that can be accessed as an array of rows.

    > SELECT visitorid, prodview[0].visittime AS firstprod,
             prodview.visittime AS lastprod,
             others[0].visittime AS firstothers,
             others.visittime AS lastothers
      FROM (vclicks ORDER BY visittime) PARTITION BY visitorid
      MATCH "(prodview | others)*$"
      ON pagetype WHEN "prodview" THEN "prodview" ELSE "others";
    -----------+-----------+----------+-------------+------------
     VISITORID | FIRSTPROD | LASTPROD | FIRSTOTHERS | LASTOTHERS 
    -----------+-----------+----------+-------------+------------
     2         | 11:02:33  | 11:09:52 | 10:21:04    | 11:13:21   
     1         | 10:24:39  | 11:05:48 | 10:21:03    | 11:13:22   
    -----------+-----------+----------+-------------+------------
    (2 rows)

<a class="anchor" name="scalability"></a>
## Scalability of Ordered Data Processing in Sclera

At a first look, the computations in the constructs described in this section seem expensive, raising issues of scalability. In this section, we briefly mention the overheads, and assert that the evaluation is highly scalable -- in fact, the evaluation happens in a single pass on the input, and in bounded memory that is independent of the number of rows in the input without any external I/O.

Use of a positive index for a column requires storing the value in the row at the specific offset for the duration of the query; the overhead of a positive index is thus the amount of memory needed to store the single value.

Use of a negative index for a column, on the other hand, requires storing the previous values of the column upto the index. The overhead of a negative index thus depends upon the absolute value of the index. Since the query only allows constants as index, this overhead is bounded for a given query.

The memory overhead of a running aggregate depends on the state being maintained incrementally by the aggregate -- this is a constant (i.e. independent of the number of rows in the input), with the singular exception of [`string_agg`](../sclerasql/sqlmisc.md#order-sensitive-aggregate-functions), in which case the overhead is linear in the number of input rows.

When there is no `PARTITION BY`, or when the input is coming sorted on the partition columns, all rows of the same partition are processed together, before the rows of the next partition -- the overheads therefore are independent of the number of partitions as well.

When the input is not sorted on the partition columns specified in the `PARTITION BY` clause, Sclera needs to process all the partitions simultaneously -- this implies a memory overhead poportional to the number of partitions. When the input is sorted on a subset of the partition columns, the number of partitions that need to be maintained concurrently reduce, and the memory overhead reduces proportionately.
