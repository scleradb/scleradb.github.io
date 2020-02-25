The ScleraSQL extension `ALIGN` enables you to align rows in two sequences, say two stock tickers, in a way as to minimize the total "distance" between the rows. The distance is arbitrary and is specified as a part of the `ALIGN` clause.

We can think of `ALIGN` as a type of `JOIN` for ordered sequences. The difference from relational joins is that in the latter, any row can join with any row as long as the join condition is met; in `ALIGN` we also have the additional ordering constraint in that the rows need to be aligned in order.

<a class="anchor" name="margin"></a>We can also provide an additional constraint to `ALIGN` by specifying a margin - the maximum number of skipped rows between two aligning rows. A margin of `m` will force the row at position `i` in one input to be aligned only rows between positions `i-m` and `i+m` in the other input.

The alignment is done using a technique called [Dynamic Time Warping](https://en.wikipedia.org/wiki/Dynamic_time_warping).

The syntax is similar to the SQL [`JOIN`](/doc/ref/sqlregular#from-table-join) syntax, and is as follows:

        table_expression ALIGN table_expression [ ON distance [ MARGIN margin ] ]

where:

- `table_expression` is an arbitrary [table expression](/doc/ref/sqlregular#table-expression)
- `distance` is a numeric [scalar expression](/doc/ref/sqlregular#scalar-expressions) that gives the distance between a row in the left input and a row in the right input
- `margin` specifies the maximum number of skipped rows between two aligned rows, as [explained earlier](#margin)

If the distance is not specified or is a constant, or if the margin is `zero`, the operator simply aligns the rows at position `i` in one input with the row in position `i` in the right input.
