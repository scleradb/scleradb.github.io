These examples are meant to give a taste of ScleraSQL features. Please click on the "Read More" buttons for additional examples and extensive documentation.

## Data Access / Virtualization

Working across systems is easy. The following links Sclera to the customer data in PostgreSQL and the order data in HBase:

    > add location pgdb as postgresql('localhost/custdb');
    > add table pgdb.customers;
    > add location hbasedb as hbase('mapreduce');
    > add table hbasedb.orders;

Regular SQL over these tables executes across PostgreSQL and HBase:

    > select location, sum(orders.total)
      from orders join customers on (orders.custid = customers.id)
      group by customers.location;

<a href="/doc/ref/sqlextdataaccess" class="text-uppercase btn btn-primary btn-block">Read More</a>

## Data Cleaning

Data sourced from web-services typically does not include the data types -- everything from numbers to dates are available as strings. These string values need to be cast to appropriate data types before they can be used in further computations. ScleraSQL provides extensions for [automatic type inference](/doc/ref/sqlcleaning#automatic-type-inference) and [text parsing](/doc/ref/sqlcleaning#text-parsing) for the purpose.

As an example, the following statement infers the types of the columns in the input file `input.csv` by looking at the first ten data rows, and then casts all values to the inferred types on the fly as they are read from the file. Any occurrence of "N/A" is taken as `NULL`.

    > EXTERNAL CSV("input.csv") TYPEINFER(NULLS("N/A") LIMIT 10)

As another example, the following statement parses the string (e.g. '08/12') in column `month` in each row of the table `input_table`, separating the numbers before and after the / and places them in result columns `m` and `y` respectively.

    > input_table TEXT parse "(\d+)/(\d+)" IN month TO (m, y)

Sclera also provides extensions for filling in missing values in the input dataset (called "imputation"); this can be [done through regular SQL](/doc/ref/sqlcleaning#data-imputation-with-regular-sql) or [with the help of a classifier](/doc/ref/sqlcleaning#data-imputation-using-machine-learning).

<a href="/doc/ref/sqlcleaning" class="text-uppercase btn btn-primary btn-block">Read More</a>

## Data Wrangling

ScleraSQL supports a large subset of standard SQL. This means that you can explore and transform (e.g. filter, join, aggregate, pivot/unpivot) your data using familiar SQL.

Several examples of data wrangling appear in the [Sclera visualization examples](/scleraviz), where it is used to transform the input external data into a table with one data point per row, as expected by the visualization component.

For a specific illustration, [click here](http://scleraviz.herokuapp.com/demo/barchart-stacked-sorted). The code below is adapted from that example.

In the following, the input data is fetched from a CSV file, and the column datatypes are determined using the `TYPEINFER` operator. The data, containing one row per state, is first sorted to get the states in decreasing order of the total population, and then transformed using SQL `UNPIVOT` to get one row per bar -- with one new column containing the population, and another containing the age group.

    > (EXTERNAL CSV("population.csv") TYPEINFER(LIMIT 1)
       ORDER BY yunder5 + y5to13 + y14to17 +
                y18to24 + y25to44 + y45to64 + y65over DESC)
      UNPIVOT population FOR age IN (
        yunder5 AS "Under 5 Years",
        y5to13 AS "5 to 13 Years",
        y14to17 AS "14 to 17 Years",
        y18to24 AS "18 to 24 Years",
        y25to44 AS "25 to 44 Years",
        y45to64 AS "45 to 64 Years",
        y65over AS "65 Years and Over"
      )

Compare the above with doing the same in [D3/Javascript](http://bl.ocks.org/mbostock/3886208).

For further examples, please see the [documentation for the supported SQL subset](/doc/ref/sqlregular) and the [`PIVOT` / `UNPIVOT` extensions](/doc/ref/sqlcrosstab).

<a href="/doc/ref/sqlintro" class="text-uppercase btn btn-primary btn-block">Read More</a>

## Machine Learning

Machine learning computations are baked into ScleraSQL. The following statement trains a classifier for identifying prospects, using a survey on customers:

    > add table pgdb.survey;
    > create classifier myclassifier(isinterested) using
      select survey.isinterested, customers.*
      from survey join customers on (survey.custid = customers.id);

Using the classifier is equally straightforward. The following query identifies prospects among target customers:

    > add table hbasedb.targets;
    > select email, name, isprospect
      from (targets classified with myclassifier(isprospect));

<a href="/doc/ref/sqlextml" class="text-uppercase btn btn-primary btn-block">Read More</a>

## Pattern Matching

ScleraSQL also supports pattern matches over streaming data. The following query labels each clickstream log with the session's login time, and the log's position in the session, all in real-time:

    > select *, login.visittime as logintime, other.count() as pos
      from clicks partition by visitorid
      match "login.other*" on pagetype when "login" then "login" else "other";

<a href="/doc/ref/sqlextordered#pattern-matching-with-match" class="text-uppercase btn btn-primary btn-block">Read More</a>

## Data Visualization

Sclera's visualization component, ScleraViz, enables quick and easy visualization of your ScleraSQL query results. ScleraViz is integrated with [ScleraSQL](/doc/ref/sqlintro); this means a few lines of ScleraSQL can fetch, clean, analyze and visualize your data in a single sweep.

ScleraViz is inspired by [Grammar of Graphics](http://vita.had.co.nz/papers/layered-grammar.html), specifically [R's ggplot2](http://ggplot2.org/) -- but is implemented as an extension to [ScleraSQL](/doc/ref/sqlintro) and uses [D3](http://d3js.org) as the rendering engine. Moreover, unlike ggplot2, ScleraViz can clean, analyze and plot *streaming* data.

An online preview with a number of examples and their live demos is available at [http://www.scleradb.com/scleraviz](/scleraviz).

<a href="/doc/ref/visualization" class="text-uppercase btn btn-primary btn-block">Read More</a>
