Sclera supports standard SQL cross-tabulation function `PIVOT` and its inverse `UNPIVOT`. The semantics of these functions is the same as in [Oracle 11g](http://www.oracle.com/technetwork/articles/sql/11g-pivot-097235.html) and [MS SQL Server 2008](http://goo.gl/gzzBgK), but with a slightly modified (in our opinion, simplified) syntax.

## `PIVOT`
The `PIVOT` operator creates [a contingency table](http://en.wikipedia.org/wiki/Cross_tabulation) from raw input data.

### Examples

For instance, consider the table `vclicks` containing (rather simplified) visitor clicks data.

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

Lets say we want to see the counts of the visits for page types "search", "checkout" and "prodview" for each visitor. One way to compute these aggregates would be using `GROUP BY` aggregation:

    > SELECT visitorid, pagetype, COUNT(*)
      FROM vclicks
      WHERE pagetype IN ("search", "checkout", "prodview")
      GROUP BY visitorid, pagetype
      ORDER BY visitorid;
    -----------+----------+-------
     VISITORID | PAGETYPE | COUNT 
    -----------+----------+-------
     1         | prodview | 2     
     2         | prodview | 3     
     2         | checkout | 1     
     2         | search   | 2     
    -----------+----------+-------
    (4 rows)

This is good, but for a better view, we might want all the aggregates for a visitor to appear in the same row. This can be achieved using embedded `CASE` statements.

    > SELECT visitorid,
             COUNT(CASE pagetype WHEN "search" THEN 1 END) AS search,
             COUNT(CASE pagetype WHEN "checkout" THEN 1 END) AS checkout,
             COUNT(CASE pagetype WHEN "prodview" THEN 1 END) AS prodview
      FROM vclicks
      GROUP BY visitorid
      ORDER BY visitorid;
    -----------+--------+----------+----------
     VISITORID | SEARCH | CHECKOUT | PRODVIEW 
    -----------+--------+----------+----------
     1         | 0      | 0        | 2        
     2         | 2      | 1        | 3        
    -----------+--------+----------+----------
    (2 rows)

With `PIVOT`, we can say, instead:

    > vclicks PARTITION BY visitorid
      PIVOT COUNT(*) FOR pagetype IN ("search", "checkout", "prodview")
      ORDER BY visitorid; 
    -----------+--------+----------+----------
     VISITORID | SEARCH | CHECKOUT | PRODVIEW 
    -----------+--------+----------+----------
     1         | 0      | 0        | 2        
     2         | 2      | 1        | 3        
    -----------+--------+----------+----------
    (2 rows)

If we want counts across all the visitors, we do not need the `PARTITION BY` clause.

    > vclicks PIVOT COUNT(*) FOR pagetype IN ("search", "checkout", "prodview");
    --------+----------+----------
     SEARCH | CHECKOUT | PRODVIEW 
    --------+----------+----------
     2      | 1        | 5        
    --------+----------+----------
    (1 row)

We can use any aggregate in place of `COUNT`. To get the last visit time for the visit to the pages instead, we can say:

    > vclicks PARTITION BY visitorid
      PIVOT MAX(visittime) FOR pagetype IN ("search", "checkout", "prodview")
      ORDER BY visitorid; 
    -----------+----------+----------+----------
     VISITORID | SEARCH   | CHECKOUT | PRODVIEW 
    -----------+----------+----------+----------
     1         |          |          | 11:05:48 
     2         | 11:04:10 | 11:07:19 | 11:09:52 
    -----------+----------+----------+----------
    (2 rows)

### Syntax

The syntax of the operator is:

    table_expression [ PARTITION BY ( partn_columns ) ]
    PIVOT aggr_func ( aggr_params ) FOR target_column IN ( target_value [ AS alias ] [, ...] )

where:

- `table_expression` is an arbitrary [table expression](/doc/ref/sqlregular#table-expression)
- `partn_columns` is an optional comma-separated list of columns in the result of `table_expression`. When specified:
    - The result of `table_expression` is partitioned on this set of columns; the aggregation happens independently on the rows within each partition.
    - These columns will be included in each output row, alongside the aggregates for the corresponding partition.
- `aggr_func` is an [aggregate function](/doc/ref/sqlmisc#aggregate-functions)
- `aggr_params` is a comma-separated list of [scalar expressions](/doc/ref/sqlregular#scalar-expressions), all of whose column references are contained in the result of `table_alias`. These are the parameters of the aggregate function `aggr_func`
- `target_column` is the `GROUP BY` column, but the values are restricted to the `target_value` list specified next
- `target_value` are values of `target_column`, these are the values on which the grouping of rows happens (within a partition).
    - The respective aggregates will be included in the output rows as separate columns.
    - The name of the column for `target_value` will be the associated `alias`, if present, or a string representation of `target_value`.

## `UNPIVOT`

The `UNPIVOT` operator converts columns into rows.

### Examples

Consider the result of the example above. We assume that the result is in a table `pagecounts`.

    > pagecounts;
    -----------+--------+----------+----------
     VISITORID | SEARCH | CHECKOUT | PRODVIEW 
    -----------+--------+----------+----------
     1         | 0      | 0        | 2        
     2         | 2      | 1        | 3        
    -----------+--------+----------+----------

This table has all the counts for a visitor accumulated in a single row for the visitor.

This might not always be convenient, and lets say we need a separate row for SEARCH, CHECKOUT and PRODVIEW counts for each visitor. Each row in the result needs to have three columns: a column `visitorid` containing the the visitor id, a column `pagetype` containg one of the values "Search", "Checkout" or "ProdView" indicating what count the row is for, and a column `pagecount` containing the correspoding count.

The `UNPIVOT` operator does this very simply:

    > pagecounts UNPIVOT pagecount FOR pagetype IN (
        search AS "Search",
        checkout AS "Checkout",
        prodview AS "ProdView"
      );

The result is:

    -----------+----------+----------
     VISITORID | PAGETYPE | PAGECOUNT
    -----------+----------+----------
     1         | Search   | 0
     1         | Checkout | 0
     1         | ProdView | 2
     1         | Search   | 2
     1         | Checkout | 1
     1         | ProdView | 3
    -----------+----------+----------

Note that `pagetype` and `pagecount` are new columns. The values in `pagetype` correspond to the columns names in the input table, as specified in the `IN` clause.

### Syntax

The syntax of the operator is:

    table_expression
    UNPIVOT value_column FOR label_column IN ( label_value_column [ AS label ] [, ...] )

where:

- `table_expression` is an arbitrary [table expression](/doc/ref/sqlregular#table-expression).
- `label_value_column` is a column in the output of `table_expression`
- `label` is the string-valued label associated with the column `label_value_column`; if unspecified, the name of `label_value_column` is taken as the label
- `value_column` is the column in the result that will contain the unpivoted values
- `label_column` is the column in the result that will contain the label of the column `label_value_column` whose value is placed in `value_column`

Each of the columns `label_value_column` are assumed to be of the same type, and this common type is the type of the column `value_column` in the result.

For each row in the output of the input `table_expression`, the operator will generate a row for each specified column `label_value_column`, consisting of:

- the value of `label_value_column` in the input row, placed in the result column `value_column`
- the label of `label_value_column`, placed in the result column `label_column`
- a copy of all columns in the input row except any of the columns `label_value_column` specified in the `IN` clause

## Overhead
The `PIVOT` and `UNPIVOT` operators in Sclera are evaluated in a single pass over the input. The memory consumption is independent of the number of input rows.
