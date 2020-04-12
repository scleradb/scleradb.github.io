In this document, we show how to build custom connectors to any data source. These connectors enable ingestion of data from arbitrary sources in a ScleraSQL query. You only need to format the data as rows of a table, and Sclera will take care of evaluating your SQL queries on the same -- these queries can include transforming, filtering and aggregating this data, as well as joining this data with data ingested from other connectors, or with data in tables stored in other data stores.

[Sclera - CSV Connector](../setup/components.md#sclera-csv) and [Sclera - Text Files Connector](../setup/components.md#sclera-textfiles) are built using this SDK. For examples of how these connectors are used in Sclera, please refer to the [SQL documentation for external data access](../sclerasql/sqlextdataaccess.md).

*To be updated, links to the code and javadocs might not work.*

## Building Data Access Connectors

To build a custom data access connector, you need to provide implementations of the following abstract classes in the SDK:

- <a class="anchor" name="externalsourceservice"></a> `ExternalSourceService` ([API Link](/api/sclera-core/com/scleradb/external/service/ExternalSourceService.html))
    - Provides the external data source as a service to Sclera.
    - Contains an `id` that identifies this service.
    - Contains a method `createSource` that is used to create a new [`ExternalSource` instance](#externalsource) for this service.
- <a class="anchor" name="externalsource"></a> `ExternalSource` ([API Link](/api/sclera-core/com/scleradb/external/objects/ExternalSource.html))
    - Represents the external data source.
    - Provides the schema and other metadata about the sourced data to Sclera at compile-time.
    - Provides the [sourced data](#tableresult) to Sclera at runtime.
- <a class="anchor" name="tableresult"></a> `TableResult` ([API Link](/api/sclera-core/com/scleradb/sql/result/TableResult.html))
    - Represents the data sourced from the external data source.
    - Provides an iterator over the rows containing the sourced data.
 
<a class="anchor" name="example"></a>
## Example: Building a CSV File Connector

This section shows how to implement a lightweight clone of the [CSV file connector](../setup/components.md#sclera-csv) using the Sclera Extensions SDK. The referenced code is available on [GitHub](https://github.com/scleradb/sclera-plugin-csv-lite).

This connector enables using data from a CSV file within a SQL query. The CSV file can be local or remote as long as it is accessible through an URL.

A simple SQL query that uses the connector is as follows:

    SELECT * FROM EXTERNAL CSVLITE("http://scleraviz.herokuapp.com/assets/data/tips.csv")

This query retrieves the CSV data in the file at the specified URL and incorporates it as a virtual table for further processing. A more complicated query could have filters, joins and aggregates on this virtual table.

While processing this query, Sclera comes across the `EXTERNAL` keyword, and accordingly identifies `"CSVLITE"` as an external datasource service. Consulting the [service provider specification file](https://github.com/scleradb/sclera-plugin-csv-lite/blob/master/src/main/resources/META-INF/services/com.scleradb.external.service.ExternalSourceService), it finds the service provider as an object of the class `CSVSourceService` ([source](https://github.com/scleradb/sclera-plugin-csv-lite/blob/master/src/main/scala/CSVSourceService.scala)), which implements the [abstract class `ExternalSourceService`](#externalsourceservice). (For details, see the [Java Service Provider Interface documentation](https://docs.oracle.com/javase/tutorial/sound/SPI-intro.html).)

The class `CSVSourceService` implements the method `createSource`, as required by the [abstract class `ExternalSourceService`](#externalsourceservice). This method takes a list of datasource parameters specified in the SQL query -- in the query above, the list consists of only one parameter, the URL, of type `CharConst`. The implementation of the method parses and validates the parameter, and uses them to create an object of the class `TickerSource`, which implements the [abstract class `ExternalSource`](#externalsource) mentioned above.

<a class="anchor" name="serviceid"></a>The service identifier is provided by the `id` attribute. For `CSVSourceService`, the identifier is `"CSVLITE"`.

For the query above, Sclera calls the method `createSource` of the `CSVSourceService` object, with the URL as the parameter, and gets back a `CSVSource` object.

The class `CSVSource` implements the following methods and attributes, as required by the [abstract class `ExternalSource`](#externalsource):

- The attribute `name` provides a user-interpretable name to this data source. In our implementation, this is taken to be the same as the service identifier.
- The attribute `columns` provides Sclera with the schema of the output that this data source will emit at runtime. The schema contains the name of each column, and its type. This must be consistent with that provided by `CSVResult` (see below). In our implementation, the `CSVResult` object is used to provide the `columns` in `CSVSource` -- but there may be cases where this coupling might not be possible (for instance, the `CSVResult` object might get computed later at runtime), hence the redundancy.
- The method `result` is called by Sclera at runtime, and returns an object of the class `CSVResult`, which implements the [abstract class `TableResult`](#tableresult) mentioned above.
- The method `toString` provides a printable string, which will be used for this datasource when [`EXPLAIN` is run on a query](../interface/shell.md#compile-time-explain) using this data source.

Sclera [plans](../intro/technical.md#query-processor) the query mentioned above taking into account the schema and result sort order provided by the `CSVSource` object. When the plan is [evaluated](../intro/technical.md#evaluation), the object's `result` method returns the object `CSVResult`, which retrieves the data from the URL, as discussed in a moment.

The class `CSVResult` implements the following methods, as required by the [abstract class `TableResult`](#tableresult):

- The attribute `columns` provides Sclera with the schema of the data that this data source will emit at runtime. The schema contains the name of each column, and its type. 
- The method `rows` returns an iterator over `TableRow` objects containing the data. A `TableRow` object can be constructed from a mapping of column names to column values. The column names and data types must be consistent with that provided the attribute `columns` above.
- The attribute `resultOrder` tells Sclera how the emitted data is sorted. This information is optional, but helps in eliminating redundant sorts on the emitted result. In our implementation, the order of rows in the source CSV data is not known, hence this attribute is an empty list.

For the query above, [Apache Commons CSV Library](http://commons.apache.org/proper/commons-csv/) is used to retrieve the CSV data from the specified URL. This data is converted into an iterator of `TableRow` instances which is returned as a part of `CSVResult`, as described above.

## Packaging and Deploying the Connector

The implementation uses [sbt](http://www.scala-sbt.org) for building the connector [(installation details)](http://www.scala-sbt.org/release/docs/Getting-Started/Setup.html#installing-sbt). This is not a requirement -- any other build tool can be used instead.

### Dependencies

The implementation has a dependency on:

- the [ Apache `commons-csv` library](http://commons.apache.org/proper/commons-csv/).
- the [`"sclera-core"`](../setup/components.md#sclera-core) and [`"sclera-config"`](../setup/components.md#sclera-config) core components. Note that these dependencies is annotated `"provided"` since these libraries will already be available in the `CLASSPATH` when this connector is run with Sclera.
- (optional) the test framework [`scalatest`](http://www.scalatest.org/) for running the tests.

see the included [sbt build file](https://github.com/scleradb/sclera-plugin-csv-lite/blob/master/build.sbt) for details.

### Deployment Steps

The connector can be deployed simply by having its jar and all its dependencies in the `CLASSPATH`.

Alternatively, for a managed deployment, follow the following steps:

- First, publish the implementation as a package, locally or in a public artifact repository. In sbt, you can publish locally by running `sbt publish-local`.
- Use [`scleradmin`](../setup/install.md#plugin-management) to install the component and its dependencies:

        > scleradmin --add <plugin> --root <sclera-root>

  where `<plugin>` is the artifact identifier for your published package, and [`<sclera-root>`](../setup/install.md#installing-sclera-core-packages-and-shell) is the directory where Sclera is installed.

For example, having published the `"CSVLITE"` plugin described above as `com.example:sclera-plugin-csv-lite:1.0-SNAPSHOT`, we can deploy it as:

    > scleradmin --add "com.example:sclera-plugin-csv-lite:1.0-SNAPSHOT" --root /path/to/sclera

The connector should now be visible to Sclera, and can be used in the queries.

**Note:** Please ensure that the identifier you assign to your connector is unique -- that is, it does not conflict with the identifier of any other available [`ExternalSourceService`](#externalsourceservice) instance.
