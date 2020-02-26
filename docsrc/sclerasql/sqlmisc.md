This section lists the supported [aggregate functions](#aggregate-functions), [data types](#data-types) and [reserved words](#reserved-words) in Sclera' SQL.

## Aggregate Functions
The aggregate functions supported in Sclera can be [order-insensitive](#order-insensitive-aggregate-functions), which apply to both [ordered](../sclerasql/sqlextordered.md#running-aggregates) and [unordered data](../sclerasql/sqlregular.md#aggregation), or [order-sensitive](#order-sensitive-aggregate-functions), which apply only to [ordered data](../sclerasql/sqlextordered.md#running-aggregates).

### Order-Insensitive Aggregate Functions
These functions can be used in standard [SQL aggregates](../sclerasql/sqlregular.md#aggregation) as well as in [running aggregates on ordered data](../sclerasql/sqlextordered.md#running-aggregates) (table adapted from [PostgreSQL documentation](http://www.postgresql.org/docs/current/static/functions-aggregate.html)):

| Aggregate | Argument Type | Return Type | Description |
| --------- | ------------- | ----------- | ----------- |
| `avg(expression)` | any numeric type | `FLOAT` | the average (arithmetic mean) of all input values |
| `bool_and(expression)` | `BOOLEAN` | `BOOLEAN` | true if all input values are true, otherwise false |
| `bool_or(expression)` | `BOOLEAN` | `BOOLEAN` | true if at least one input value is true, otherwise false |
| `count(*)` |   | `BIGINT` | number of input rows |
| `count(expression)` | any type | `BIGINT` | number of input rows for which the value of expression is not null |
| `every(expression)` | `BOOLEAN` | `BOOLEAN` | equivalent to bool\_and |
| `max(expression)` | any numeric, string, or date/time type | same as argument type | maximum value of expression across all input values |
| `min(expression)` | any numeric, string, or date/time type | same as argument type | minimum value of expression across all input values |
| `sum(expression)` | any numeric type | `BIGINT` for `SMALLINT` or `INTEGER` arguments, `FLOAT` for floating-point arguments | sum of expression across all input values |
| `corr(Y, X)` | any numeric type | `FLOAT` | correlation coefficient |
| `covar_pop(Y, X)` | any numeric type | `FLOAT` | population covariance |
| `covar_samp(Y, X)` | any numeric type | `FLOAT` | sample covariance |
| `regr_avgx(Y, X)` | any numeric type | `FLOAT` | average of the independent variable `(sum(X)/N)` |
| `regr_avgy(Y, X)` | any numeric type | `FLOAT` | average of the dependent variable `(sum(Y)/N)` |
| `regr_count(Y, X)` | any numeric type | `BIGINT` | number of input rows in which both expressions are nonnull |
| `regr_intercept(Y, X)` | any numeric type | `FLOAT` | y-intercept of the least-squares-fit linear equation determined by the `(X, Y)` pairs |
| `regr_r2(Y, X)` | any numeric type | `FLOAT` | the coefficient of determination (also called R-squared or goodness of fit) |
| `regr_slope(Y, X)` | any numeric type | `FLOAT` | slope of the least-squares-fit linear equation determined by the `(X, Y)` pairs |
| `regr_sxx(Y, X)` | any numeric type | `FLOAT` | `sum(X^2) - sum(X)^2/N` ("sum of squares" of the independent variable) |
| `regr_sxy(Y, X)` | any numeric type | `FLOAT` | `sum(X*Y) - sum(X)*sum(Y)/N` ("sum of products" of independent times dependent variable) |
| `regr_syy(Y, X)` | any numeric type | `FLOAT` | `sum(Y^2) - sum(Y)^2/N` ("sum of squares" of the dependent variable) |
| `stddev(expression)` | any numeric type | `FLOAT` | historical alias for stddev\_samp |
| `stddev_pop(expression)` | any numeric type | `FLOAT` | population standard deviation of the input values |
| `stddev_samp(expression)` | any numeric type | `FLOAT` | sample standard deviation of the input values |
| `variance(expression)` | any numeric type | `FLOAT` | historical alias for var\_samp |
| `var_pop(expression)` | any numeric type | `FLOAT` | population variance of the input values (square of the population standard deviation) |
| `var_samp(expression)` | any numeric type | `FLOAT` | sample variance of the input values (square of the sample standard deviation) |

### Order-Sensitive Aggregate Functions
These functions can only be used in [running aggregates on ordered data](../sclerasql/sqlextordered.md#running-aggregates) (table adapted from [PostgreSQL documentation](http://www.postgresql.org/docs/current/static/functions-window.html#FUNCTIONS-WINDOW-TABLE)):

| Function | Return Type | Description |
| -------- | ----------- | ----------- |
| `row_number()` | `BIGINT` | number of the current row within its partition, counting from 1 |
| `rank()` | `BIGINT` | rank of the current row with gaps; same as row\_number of its first peer |
| `dense_rank()` | `BIGINT` | rank of the current row without gaps; this function counts peer groups |
| `percent_rank()` | `FLOAT` | relative rank of the current row: `(rank - 1) / (total rows - 1)` |
| `cume_dist()` | `FLOAT` | relative rank of the current row: (number of rows preceding or peer with current row) / (total rows) |
| `ntile(num_buckets INTEGER)` | `INTEGER` | integer ranging from 1 to the argument value, dividing the partition as equally as possible |
| `lag(value ANY [, offset INTEGER [, default ANY ]])` | same type as value | returns value evaluated at the row that is offset rows before the current row within the partition; if there is no such row, instead return default. Both offset and default are evaluated with respect to the current row. If omitted, offset defaults to 1 and default to null |
| `first_value(value ANY)` | same type as value | returns value evaluated at the row that is the first row of the window frame |
| `last_value(value ANY)` | same type as value | returns value evaluated at the row that is the last row of the window frame |
| `nth_value(value ANY, nth INTEGER)` | same type as value | returns value evaluated at the row that is the nth row of the window frame (counting from 1); null if no such row |
| `string_agg(expression ANY, delimiter VARCHAR)` | `CHAR(n)` | input values concatenated into a string, separated by delimiter |

## Data Types
Sclera supports a subset of [PostgreSQL data types](http://www.postgresql.org/docs/current/interactive/datatype.html):

`BIGINT`, `BOOL` or `BOOLEAN`, `CHAR(n)`, `CHAR`, `DATE`, `DECIMAL(prec)`, `DECIMAL(prec, scale)`, `DECIMAL`, `FLOAT(prec)`, `FLOAT`, `INT` or `INTEGER`, `NUMERIC(prec)`, `NUMERIC(prec, scale)`, `NUMERIC`, `REAL`, `SMALLINT`, `TEXT`, `TIMESTAMP`, `TIME`, `VARCHAR(n)`, `VARCHAR`.

Note that `ARRAY` and other composite data types are not supported.

## Reserved Keywords
The following cannot be used as a name or alias of any object within a SQL command:

`ADD`, `ALL`, `ALTER`, `AND`, `ANY`, `ARG`, `AS`, `ASC`, `ASSOCIATOR`, `BETWEEN`, `BIGINT`, `BIT`, `BOOL`, `BOOLEAN`, `BPCHAR`, `BY`, `CASE`, `CAST`, `CHAR`, `CLASSIFIED`, `CLASSIFIER`, `CLUSTERED`, `CLUSTERER`, `COLUMN`, `CONNECTED`, `CREATE`, `CROSS`, `DATE`, `DAY`, `DECIMAL`, `DELETE`, `DESC`, `DISTINCT`, `DROP`, `ELSE`, `END`, `ESCAPE`, `EXCEPT`, `EXISTS`, `EXTERNAL`, `FALSE`, `FETCH`, `FIRST`, `FLAG`, `FLOAT`, `FOREIGN`, `FROM`, `FULL`, `GRAPH`, `GROUP`, `HAVING`, `HOUR`, `ILIKE`, `IMPUTED`, `IN`, `INNER`, `INSERT`, `INT`, `INTEGER`, `INTERSECT`, `INTERVAL`, `INTO`, `IS`, `ISNULL`, `JOIN`, `KEY`, `LABEL`, `LAST`, `LEFT`, `LIKE`, `LIMIT`, `LOCATION`, `MATCH`, `MINUTE`, `MONTH`, `NATURAL`, `NEXT`, `NOT`, `NOTNULL`, `NULL`, `NULLS`, `NUMERIC`, `OF`, `OFFSET`, `ON`, `ONLY`, `OR`, `ORDER`, `ORDERED`, `OUTER`, `OVER`, `PARTITION`, `PIVOT`, `PREDICTOR`, `PRIMARY`, `READONLY`, `REAL`, `REFERENCES`, `REMOVE`, `RIGHT`, `ROW`, `ROWS`, `SCHEMA`, `SECOND`, `SELECT`, `SET`, `SIMILAR`, `SMALLINT`, `SOME`, `SYMMETRIC`, `TABLE`, `TEMP`, `TEMPORARY`, `TEXT`, `THEN`, `TIME`, `TIMESTAMP`, `TO`, `TRUE`, `UNION`, `UNKNOWN`, `UPDATE`, `USING`, `VALUES`, `VARBIT`, `VARCHAR`, `VARCHAR2`, `VARYING`, `VIEW`, `WHEN`, `WHERE`, `WITH`, `WITHOUT`, `YEAR`, `ZONE`.
