In this section, we describe the extensions that enable access to data in [CSV (Comma-Separated Value) files](#sclera-csv), [free-form text files](#sclera-csv) and web services within a SQL query. This data can be used just like a relational base table within the SQL query.

These extensions are developed using the [Sclera Connector Development SDK](../sdk/sdkintro.md), the sources for each of these extensions is available on [GitHub](https://github.com/scleradb) for reference.

<a class="anchor" name="sclera-csv"></a>
## Accessing CSV Data
Sclera can dynamically load CSV files and present the data as a table to a SQL query. Recall that a [CSV file](https://en.wikipedia.org/wiki/Comma-separated_values) is a text file, structured into lines such that the first line contains the column names separated by commas, and the remaining lines contain the corresponding values separated by commas.

This feature introduces a new [table expression](#table-expression) with the following syntax:

    EXTERNAL CSV( path [ , format [ , header_flag [ , path_col ] ] ]  )
    [ ORDERED BY ( expression [ ASC | DESC ] [ NULLS { FIRST | LAST } ] [, ...] ) ]

- `path` is one of the following:
    - a URL that responds with CSV data (e.g. `http://download.finance.yahoo.com/d/quotes.csv?s=AAPL&f=sl1d1t1c1ohgv&e=.csv`)
    - the path to a CSV file
    - the path to a directory containing multiple CSV files; each of these files is expected to have the same format and identical headers (if present).
- <a class="anchor" name="sclera-csv-format"></a> `format` is the CSV format, as specified by the [Apache Commons CSV library](http://commons.apache.org/proper/commons-csv/). It can be one of (see the [Apache Commons CSV documentation](http://commons.apache.org/proper/commons-csv/archives/1.1/apidocs/org/apache/commons/csv/CSVFormat.html) for details):
    - [DEFAULT](http://commons.apache.org/proper/commons-csv/archives/1.1/apidocs/org/apache/commons/csv/CSVFormat.html#DEFAULT), which specifies `','` as the delimiter, `'"'` as the quote, `'\r\n'` as the record separator, and ignores empty lines in the input.
    - [RFC4180](http://commons.apache.org/proper/commons-csv/archives/1.1/apidocs/org/apache/commons/csv/CSVFormat.html#RFC4180), which specifies `','` as the delimiter, `'"'` as the quote, `'\r\n'` as the record separator, and does not ignore empty lines in the input.
    - [EXCEL](http://commons.apache.org/proper/commons-csv/archives/1.1/apidocs/org/apache/commons/csv/CSVFormat.html#EXCEL), which is the same as `RFC4180` except that it allows missing column names.
    - [TDF](http://commons.apache.org/proper/commons-csv/archives/1.1/apidocs/org/apache/commons/csv/CSVFormat.html#TDF), which specifies `'\t'` (tab) as the delimiter, `'"'` as the quote, `'\r\n'` as the record separator, and ignores spaces surrounding the values.
    - [MYSQL](http://commons.apache.org/proper/commons-csv/archives/1.1/apidocs/org/apache/commons/csv/CSVFormat.html#MYSQL), which specifies `'\t'` (tab) as the delimiter, does not quote the values (but escapes special characters with `'\'`), specifies `'\n'` as the record separator, and does not ignore empty lines in the input.
- `header_flag` can be `HEADER` or `NOHEADER` indicating whether or not a header is present in the input CSV. If not specified, it defaults to `HEADER`
- `path_col` the name of an optional output column. If specified, a new column of the specified name is added to each row, and populated with the URL or file name from which that row has been read. This is useful when the `path` is a directory -- in this case, the specified column will contain the path to the relevant file under that directory.
- The `ORDERED BY` clause *declares* the sort order of the rows in the CSV file; note that this is an unverified declaration, and Sclera blindly relies on the same while planning further evaluation on the rows read.

As an example, consider a CSV file `"/path/to/custinfo.csv"` containing columns `email`, `location` and `age` for each customer.

You can view the data as a table in Sclera by saying:

    > EXTERNAL CSV("/path/to/custinfo.csv");

This can be used in SQL queries just like a regular base table or view. The following query lists, for each distinct location, the number customers in that location.

    > SELECT location, COUNT(*)
      FROM EXTERNAL CSV("/path/to/custinfo.csv")
      GROUP BY location;

The following query joins this table with a [MySQL (connected as location `myloc`)](../setup/dbms.md#connecting-to-mysql) table `defaulters`, and computes the number of defaulters in that region:

    > SELECT location, COUNT(*)
      FROM EXTERNAL CSV("/path/to/custinfo.csv") JOIN myloc.defaulters USING (email)
      GROUP BY location;

Note that since the CSV format does not include the type of the columns, each column in the table returned by `CSV(_)` has type `VARCHAR`.

### Exporting Query Results as CSV Files
The `EXTERNAL CSV` can also be used to export the result of a query into a CSV file, using the following syntax:

    CREATE EXTERNAL CSV( file_path [ , format ] ) AS table_expression

- `file_path` is the path to the file to be created. If the file exists, it will be overwritten.
- `format` is as [described earlier](#sclera-csv-format).
- `table_expression` is a [table expression](#table-expression).

The syntax is similar to the standard [`CREATE TABLE ... AS` statement](../sclerasql/sqlregular.md#creating-tables-with-empty-results), except the `TABLE` is replaced by the `EXTERNAL CSV(...)`.

For example, the following statement creates a CSV file `custdefault.csv` containing the `JOIN` of the data in `custinfo.csv` and the table `defaulters` in the location `myloc`:

    > CREATE EXTERNAL CSV("/path/to/custdefault.csv") AS
      EXTERNAL CSV("/path/to/custinfo.csv") JOIN myloc.defaulters USING (email);

Similarly, the following statement creates a CSV file `locdefault.csv` containing the number of defaulters by location:

    > CREATE EXTERNAL CSV("/path/to/locdefault.csv") AS
      SELECT location, COUNT(*) as defaulter_count
      FROM EXTERNAL CSV("/path/to/custinfo.csv") JOIN myloc.defaulters USING (email)
      GROUP BY location;

<a class="anchor" name="sclera-textfiles"></a>
## Accessing Text Files
Sclera can dynamically load raw text data from text files and present the same as a table to a SQL query.

This feature introduces a new [table expression](#table-expression) with the following syntax:

    EXTERNAL TEXTFILES( filedir_path [, ...] )

where `filedir_path` is the path to file to be loaded, or to a directory containing the files to be loaded. Multiple paths can be specified in a comma-separated list.

The resulting virtual table contains a row for each file, with two columns of type `VARCHAR`. The first column, called `file` contains the [canonical path](http://docs.oracle.com/javase/7/docs/api/java/io/File.html#getCanonicalPath\(\)) of the file, and the second column `contents` contains the textual contents of the file.

The following query returns the path and contents of file `"/tmp/myfile"` and all files under the directory `"/tmp/mydir"`.

    > SELECT file, contents
      FROM TEXTFILES("/tmp/myfile", "/tmp/mydir");

The resulting table can also be aggregated over, joined with other base or virtual tables, and so on, just like a base table or a view.

<a class="anchor" name="sclera-stockticker"></a>
## Accessing Web Services
Sclera can provide access to data provided by web services from within SQL, using components built using the [Sclera Extensions SDK](../sdk/sdkintro.md).

As an illustrative example, Sclera provides access to (dummy) stock ticker streams.

This feature introduces a table expression with the following syntax:

    EXTERNAL STOCKTICKER( symbol [ , gap [ , duration ] ] )
    [ ORDERED BY ( expression [ ASC | DESC ] [ NULLS { FIRST | LAST } ] [, ...] ) ]

- The mandatory parameter `symbol` string contains the stock symbol of interest. A stock symbol can be qualified with the exchange symbol (e.g. `"NYSE:IBM"`), or it can be unqualified (e.g. `"IBM"`). The exchange for unqualified symbol is determined automatically by the service.
- The optional parameter `gap` is an integer specifying the duration between consecutive readings, in seconds. For a reading per minute specify 60, for one per hour specify 3600, and so on. When not specified, `gap` is taken to be 60 seconds.
- The optional parameter `duration` is an integer specifying the number of days for which the data is requested. When not specified, `duration` is taken to be 1 day.
- The `ORDERED BY` clause *declares* the sort order of the emitted rows; note that this is an unverified declaration, and Sclera blindly relies on the same while planning further evaluation on these rows.

The resulting table contains one row per reading for the stock specified by the parameter `symbol`, and the duration between readings and the number of readings is determined by parameters `gap` and `duration` mentioned above. Each output row contains the following columns:

- `symbol`, containing the ticker symbol (same as the `symbol` input parameter)
- `exchange`, containing the exchange for the `symbol`; this is taken from the `symbols` parameter if mentioned with the ticker symbol, or is identified automatically by the service
- `ts`, containing the timestamp of the reading
- `high` value of the stock since the previous reading
- `low` value of the stock since the previous reading
- `open` value after the previous reading
- `close` value before this reading

The output is ordered on `symbol`, and on `ts` within each symbol.

The resulting table can also be aggregated over, joined with other base or virtual tables, and so on, just like a base table or a view.

For instance, the following query gives the symbol, exchange, timestamp and the difference between open and close for each two-minute period for past one day. The stocks of interest is ORCL (Oracle).

    > SELECT symbol, exchange, ts, (close - open) as difference
      FROM EXTERNAL STOCKTICKER("ORCL", 120, 1);
