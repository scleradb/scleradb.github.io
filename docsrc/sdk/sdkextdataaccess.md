In this document, we show how to build custom connectors to any data source. These connectors enable ingestion of data from arbitrary sources in a ScleraSQL query. You only need to format the data as rows of a table, and Sclera will take care of evaluating your SQL queries on the same -- these queries can include transforming, filtering and aggregating this data, as well as joining this data with data ingested from other connectors, or with data in tables stored in other data stores.

[Sclera - Stock Ticker Connector](/doc/ref/components#sclera-stockticker), [Sclera - CSV Connector](/doc/ref/components#sclera-csv), and [Sclera - Text Files Connector](/doc/ref/components#sclera-textfiles) are built using this SDK. For examples of how these connectors are used in Sclera, please refer to the [SQL documentation for external data access](/doc/ref/sqlextdataaccess).

## Building Data Access Connectors

To build a custom datasource connector, you need to provide implementations of the following abstract classes in the SDK:

- <a class="anchor" name="dataservice"></a> `DataService` ([Scala](http://scleradb.github.io/sclera-core-sdk/index.html#com.scleradb.external.service.DataService), [Java](http://scleradb.github.io/sclera-extensions-java-sdk/index.html#com.scleradb.java.external.service.DataService))
    - Provides the datasource as a service to Sclera.
    - Contains an `id` that identifies this service.
    - Contains a method `createSource` that is used to create a new [datasource instance](#datasource) for this service.
- <a class="anchor" name="datasource"></a> `DataSource` ([Scala](http://scleradb.github.io/sclera-core-sdk/index.html#com.scleradb.external.datasource.DataSource), [Java](http://scleradb.github.io/sclera-extensions-java-sdk/index.html#com.scleradb.java.external.datasource.DataSource))
    - Represents the datasource.
    - Provides the schema and other metadata about the output of the datasource to Sclera at compile-time.
    - Provides the output of the datasource to Sclera at runtime.
- <a class="anchor" name="tableresult"></a> `TableResult` ([Scala](http://scleradb.github.io/sclera-core-sdk/index.html#com.scleradb.sql.datatypes.TableResult), [Java](http://scleradb.github.io/sclera-extensions-java-sdk/index.html#com.scleradb.java.sql.datatypes.TableResult))
    - Represents the output of the datasource.
    - Provides an iterator over the rows containing the output data.
 
<a class="anchor" name="example"></a>
## Example: Building a Stock Ticker Connector

This section shows how to implement the [stock ticker connector](/doc/ref/components#sclera-stockticker) using the Sclera Extensions SDK. This connector enables using data from a stock ticker web service within a SQL query, and is included in all Sclera subscription packages; see the [component documentation](/doc/ref/sqlextdataaccess#sclera-stockticker) for the details.

The referenced code is available in the `sclera-stockticker` subdirectory of the example repository ([Scala](https://github.com/scleradb/sclera-extensions-scala), [Java](https://github.com/scleradb/sclera-extensions-java)).

Consider a simple SQL query that uses the connector ([full syntax](/doc/ref/sqlextdataaccess#sclera-stockticker)):

    SELECT * FROM EXTERNAL STOCKTICKER("NYSE:IBM", 120, 1);

This query retrieves the stock ticker for `IBM` in exchange `NYSE`, reading at gaps of `120` seconds for past `1` day.

While processing this query, Sclera comes across the `EXTERNAL` keyword, and accordingly identifies `"STOCKTICKER"` as an external datasource service. It finds the service provider as an object of the class `TickerService` ([Scala](https://github.com/scleradb/sclera-extensions-scala/blob/master/sclera-stockticker/src/main/scala/service/TickerService.scala), [Java](https://github.com/scleradb/sclera-extensions-java/blob/master/sclera-stockticker/src/main/java/com/example/scleradb/java/stockticker/service/TickerService.java)), which implements the [abstract class `DataService`](#dataservice) mentioned above.

The class `TickerService` implements the following method, as required by the [abstract class `DataService`](#dataservice):

- The method `createSource`, which takes a list (array in Java) of datasource parameters specified in the SQL query -- in the query above, these parameters are the string `"NYSE:IBM"`, integer `120` and integer `1`. The implementation of the method parses and validates these parameters, and uses them to create an object of the class `TickerSource`, which implements the [abstract class `DataSource`](#datasource) mentioned above.

<a class="anchor" name="serviceid"></a>The service identifier is provided by the `id` attribute (method in Java). For `TickerService`, the identifier is `"STOCKTICKER"`.

For the query above, Sclera calls the method `createSource` of the `TickerService` object, with parameters as the string `"NYSE:IBM"`, integer `120` and integer `1`, and gets back a `TickerSource` object.

The class `TickerSource` implements the following methods, as required by the [abstract class `DataSource`](#datasource):

- The method `toString` provides a printable string, which will be used for this datasource when [`EXPLAIN` is run on a query](/doc/ref/shell#compile-time-explain) using this datasource.
- The method `columns` (`columnsList` in Java) provides Sclera with the schema of the output that this datasource will emit at runtime. The schema contains the name of each column, and its type (in Java, the helper methods of the utility [JavaUtil object](http://scleradb.github.io/sclera-extensions-java-sdk/index.html#com.scleradb.java.util.JavaUtil$) are used to construct the column and type specifications).
- The method `resultOrderOpt` (`sortExprs` in Java) tells Sclera how the output is sorted. This information is optional, but helps in eliminating redundant sorts on the emitted result.
- The method `result` is called by Sclera at runtime, and returns an object of the class `TickerResult`, which implements the [abstract class `TableResult`](#tableresult) mentioned above.

Sclera [plans](/doc/ref/technical#query-processor) the query mentioned above taking into account the schema and result sort order provided by the `TickerSource` object. When the plan is [evaluated](/doc/ref/technical#evaluation), the object's `result` method returns the object `TickerResult`, which retrieves the data from the stock ticker web service, as discussed in a moment.

The class `TickerResult` implements the following method, as required by the [abstract class `TableResult`](#tableresult):

- The method `rows` (`resultRows` in Java) returns an iterator over `TableRow` objects containing the data. A `TableRow` object can be constructed from a mapping of column names to column values. The column names must be the same as provided by `TickerSource`, as described earlier, and the associated values also contain the type information for the value. Both Scala and Java have helper functions that simplify building these data rows.

In addition, `TickerResult` also specifies the metadata attributes `columns` (method `columnsList` in Java) and `resultOrderOpt` (method `sortExprs` in Java), as required by the [abstract class `TableResult`](#tableresult) -- this metadata must be consistent with that provided earlier by `TickerSource`.

For the query above, the web service is invoked using HTTP, and the response is parsed to get the ticker reads, which are returned as an iterator over `TableRow` objects. In the Scala implementation, this logic is embedded in the `rows` method of the class `TickerResult`. In the Java implementation, [class `LineIterator`](https://github.com/scleradb/sclera-extensions-java/blob/master/sclera-stockticker/src/main/java/com/example/scleradb/java/stockticker/source/LineIterator.java) and [class `TickerIterator`](https://github.com/scleradb/sclera-extensions-java/blob/master/sclera-stockticker/src/main/java/com/example/scleradb/java/stockticker/source/TickerIterator.java) perform the job (note that these classes simplify the implementation of the class `TickerResult`, and are not required by the SDK).

## Packaging and Deploying the Connector

The implementation uses [sbt](http://www.scala-sbt.org) for building the connector [(installation details)](http://www.scala-sbt.org/release/docs/Getting-Started/Setup.html#installing-sbt). This is not a requirement -- any other build tool can be used instead.

### Dependencies

For Scala:

- The Scala implementation has a dependency on the [`"sclera-core"` library](/doc/sdk/sdkintro#scalasdk). This library is available from the [Sclera repository](http://scleradb.releases.s3.amazonaws.com); see the included [sbt build file](https://github.com/scleradb/sclera-extensions-scala/blob/master/sclera-stockticker/build.sbt) for the details. Note that the dependency is annotated `"provided"` since the `jar` for `"sclera-core"` will be available in the `CLASSPATH` when this connector is run with Sclera.

For Java:

- The Java implementation has a dependency on the [`"sclera-core"` library](/doc/sdk/sdkintro#scalasdk), as wells as on the [`"sclera-extensions-java-sdk"` library](#javasdk). These libraries are available from the [Sclera repository](http://scleradb.releases.s3.amazonaws.com); see the included [sbt build file](https://github.com/scleradb/sclera-extensions-java/blob/master/sclera-stockticker/build.sbt) for the details. Note that the dependency on `"sclera-core"` is annotated `"provided"` since the `jar` for `"sclera-core"` will be available in the `CLASSPATH` when this connector is run with Sclera.

### Deployment Steps

The connector can be deployed using the following steps:

#### Alternative 1 (Recommended)
- First, publish the implementation as a local package. In sbt, this is done by running `sbt publish-local`.
- Run the following to install the component and its dependencies:

    <pre><code>> $SCLERA_HOME/bin/install.sh package_name package_version package_org</code></pre>

    - `$SCLERA_HOME` is the directory where Sclera is installed
    - `package_name` is the name of the package being installed
    - `package_version` is the version of the package being installed
    - `package_org` is the org of the package being installed
- Run the following to include the path to the installed component package jar and the dependencies in the `CLASSPATH`:

    <pre><code>> . $SCLERA_HOME/assets/install/classpath/package_name-classpath.sh</code></pre>

    - The bash script `package_name-classpath.sh` was automatically created in the previous step, while installing the package. The `package_name` part of the file name is the name of the package installed.

The connector should now be visible to Sclera, and can be used in the queries.

#### Alternative 2
- First, compile and package the implementation into a `jar` file. In sbt, this is done by running `sbt package`. If you have external dependencies, you may need to assemble everything into a single `jar` -- in `sbt`, this can be done using the [`sbt-assembly` plugin](https://github.com/sbt/sbt-assembly).
- Next, add the path to the generated `jar` file to the `CLASSPATH` environment variable.

The connector should now be visible to Sclera, and can be used in the queries.

**Note:** Please ensure that the identifier you assign to the connector is unique, that is - does not conflict with the identifier of any other available [`DataService`](#dataservice) instance.

While deploying or experimenting with the [example discussed above](#example), please change the [service identifier](#serviceid) to something other than `"STOCKTICKER"` before deployment, since the [`"STOCKTICKER"` service is already available as a part of your Sclera package](/doc/ref/sqlextdataaccess#sclera-stockticker).
