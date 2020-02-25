While computing the optimal (`MAX` or `MIN`) of an expression is straightforward in SQL, retrieving the rows that contain the optimal value is remarkably tedious and inefficient.

As a simple example, consider the table table `vhclicks` containing (simplified) clickstream data for an e-commerce site. It contains a row for each page visited, containing the visitor identifier `visitorid`, time of visit `visittime`, page type `pagetype`, and the number of "hovers" by the visitor on the page `hcount`.

    > vhclicks;
    -----------+-----------+----------+--------
     VISITORID | VISITTIME | PAGETYPE | HCOUNT 
    -----------+-----------+----------+--------
     1         | 10:21:03  | login    | 1      
     1         | 10:24:39  | prodview | 10     
     1         | 10:27:14  | logout   | 1      
     2         | 10:21:04  | login    | 1      
     2         | 10:22:10  | search   | 5      
     2         | 10:27:15  | logout   | 1      
     2         | 11:01:22  | login    | 1      
     1         | 11:01:23  | login    | 1      
     2         | 11:02:33  | prodview | 7      
     2         | 11:04:10  | search   | 10     
     2         | 11:05:47  | prodview | 5      
     1         | 11:05:48  | prodview | 9      
     2         | 11:07:19  | checkout | 3      
     2         | 11:09:52  | prodview | 10     
     2         | 11:13:21  | logout   | 1      
     1         | 11:13:22  | logout   | 1      
    -----------+-----------+----------+--------
    (16 rows)

Now, suppose we want to retrieve the rows for pages with the greatest `hcount` visited after `11:00`. Here is how we perform this simple task in standard SQL.

    > SELECT *
      FROM vhclicks
      WHERE hcount = (
        SELECT MAX(hcount) FROM vhclicks WHERE visittime > '11:00:00'::time
      ) AND visittime > '11:00:00'::time;
    -----------+-----------+----------+--------
     VISITORID | VISITTIME | PAGETYPE | HCOUNT 
    -----------+-----------+----------+--------
     2         | 11:04:10  | search   | 10     
     2         | 11:09:52  | prodview | 10     
    -----------+-----------+----------+--------
    (2 rows)

Notice the redundancy -- we need to specify the condition twice, which could be painful and error-prone for a complex `WHERE` clause. Moreover, this takes two passes over the data, and the `WHERE` clause is evaluated twice. This is clearly undesirable for large datasets.

Sclera provides a construct, called `ARG`, that eliminates the redundancy and inefficiencies in syntax as well as execution of the above query. In Sclera, we can frame the same query as:

    > (vhclicks WHERE visittime > '11:00:00'::time) ARG MAX(hcount)
    -----------+-----------+----------+--------
     VISITORID | VISITTIME | PAGETYPE | HCOUNT 
    -----------+-----------+----------+--------
     2         | 11:04:10  | search   | 10     
     2         | 11:09:52  | prodview | 10     
    -----------+-----------+----------+--------
    (2 rows)

Unlike the earlier query, this query is evaluated in a single pass over the data.

We can think of `ARG` as a special "filter", similar to [`WHERE`](/doc/ref/sqlregular#where-clause) and [`HAVING`](/doc/ref/sqlregular#having-clause). However, while `WHERE` and `HAVING` compute their output by applying a condition one row at a time, the output rows of `ARG` is computed based on the entire input (or the [input partition](#arg-on-partitioned-input)).

We can get the rows containing optimal values for multiple aggregates as well -- the following query returns the row with the earliest `visittime` past `11:00`, in addition to the rows above.

    > (vhclicks WHERE visittime > '11:00:00'::time) ARG (MAX(hcount), MIN(visittime));
    -----------+-----------+----------+--------
     VISITORID | VISITTIME | PAGETYPE | HCOUNT 
    -----------+-----------+----------+--------
     2         | 11:04:10  | search   | 10     
     2         | 11:09:52  | prodview | 10     
     2         | 11:01:22  | login    | 1      
    -----------+-----------+----------+--------
    (3 rows)

It is easy to do a cascaded computation. For instance, to find the row with the earliest `visittime` among those with the greatest `hcount` across pages visited after `11:00`, we say:

    > (vhclicks WHERE visittime > '11:00:00'::time) ARG MAX(hcount) ARG MIN(visittime);
    -----------+-----------+----------+--------
     VISITORID | VISITTIME | PAGETYPE | HCOUNT 
    -----------+-----------+----------+--------
     2         | 11:04:10  | search   | 10     
    -----------+-----------+----------+--------
    (1 row)

We can use `ARG` with the [`MATCH` operator](/doc/ref/sqlextordered#pattern-matching-with-match) as well. For instance, the following query finds the product views on which visitor id `1` hovered the most in a session:

    > (vhclicks WHERE visitorid = 1 ORDER BY visittime) ARG prodview.MAX(hcount)
      OVER MATCH "login.(prodview | search | checkout)*.logout" ON pagetype;
    -----------+-----------+----------+--------
     VISITORID | VISITTIME | PAGETYPE | HCOUNT 
    -----------+-----------+----------+--------
     1         | 10:24:39  | prodview | 10     
     1         | 11:05:48  | prodview | 9      
    -----------+-----------+----------+--------
    (2 rows)

A set of rows will be output for *every* match of the [`MATCH` regular expression](/doc/ref/sqlextordered#match-syntax). Note that the order of the input is relevant for the above query.

## `ARG` on Partitioned Input
We can use `ARG` on partitioned input. The following query find the pages with the highest `hcount` for each visitor after `11:00`.

    > (vhclicks WHERE visittime > '11:00:00'::time) PARTITION BY visitorid ARG MAX(hcount);
    -----------+-----------+----------+--------
     VISITORID | VISITTIME | PAGETYPE | HCOUNT 
    -----------+-----------+----------+--------
     2         | 11:04:10  | search   | 10     
     2         | 11:09:52  | prodview | 10     
     1         | 11:05:48  | prodview | 9      
    -----------+-----------+----------+--------
    (3 rows)

Similarly, the following query finds, for each visitor, the product views on which the visitor hovered the most in a session:

    > (vhclicks ORDER BY visittime) PARTITION BY visitorid ARG prodview.MAX(hcount)
      OVER MATCH "login.(prodview | search | checkout)*.logout" ON pagetype;
    -----------+-----------+----------+--------
     VISITORID | VISITTIME | PAGETYPE | HCOUNT 
    -----------+-----------+----------+--------
     1         | 10:24:39  | prodview | 10     
     2         | 11:09:52  | prodview | 10     
     1         | 11:05:48  | prodview | 9      
    -----------+-----------+----------+--------
    (3 rows)

## `ARG` Syntax
This section introduced a new [table expression](/doc/ref/sqlregular#table-expression) with the following syntax:

    table_expression [ PARTITION BY ( partn_columns ) ]
    ARG ( [ label . ] aggr_func ( aggr_params ) [, ...] )
    [ [ OVER ] match_expression ]

where:

- `table_expression` is an arbitrary [table expression](/doc/ref/sqlregular#table-expression).
- `partn_columns` is an optional comma-separated list of columns in the result of `table_expression`. When specified:
    - The result of `table_expression` is partitioned on this set of columns; the aggregation happens independently on the rows within each partition.
- `aggr_func` is an [aggregate function](/doc/ref/sqlmisc#aggregate-functions)
- `aggr_params` is a comma-separated list of [scalar expressions](/doc/ref/sqlregular#scalar-expressions), all of whose column references are contained in the result of `table_alias`. These are the parameters of the aggregate function `aggr_func`.
- `match_expression` is an optional [`MATCH` expression](/doc/ref/sqlextordered#match-syntax)
- `label` is optional. When specified, it can be:
    - When `MATCH` is not present, it is the `table_alias` for the `table_expression`, or
    - When `MATCH` is present, it is a label identifying a subsequence in the [`MATCH` regular expression](/doc/ref/sqlextordered#regular-expression).
