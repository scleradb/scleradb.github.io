In this document, we show how to build custom connectors to any relational/non-relational database management system. These connectors interface Sclera with an arbitrary database system, relational or non-relational, providing access to the underlying data, and also enable Sclera to push down computations in relevant parts user queries and commands on the interfaced database system.

[Sclera - Oracle Connector](../setup/components.md#sclera-oracle), [Sclera - MySQL Connector](../setup/components.md#sclera-mysql), and [Sclera - PostgreSQL Connector](../setup/components.md#sclera-postgresql) are built using this SDK.  For examples of how these connectors are used in Sclera, please refer to the [documentation on connecting Sclera to database systems](../setup/dbms.md).

## Building Database System Connectors

To build a custom datasource connector, you need to provide implementations of the following abstract classes in the SDK:

- <a class="anchor" name="dbservice"></a> `DBService` ([Scala](http://scleradb.github.io/sclera-core-sdk/index.html#com.scleradb.dbms.service.DataService), [Java](http://scleradb.github.io/sclera-extensions-java-sdk/index.html#com.scleradb.java.dbms.service.DBService))
    - Provides the database system as a service to Sclera.
    - Contains an `id` that identifies this service.
    - Contains a method `createLocation` that is used to create a new [location instance](#location) for this service.
- <a class="anchor" name="location"></a> `Location` ([Scala](http://scleradb.github.io/sclera-core-sdk/index.html#com.scleradb.dbms.location.Location), [Java](http://scleradb.github.io/sclera-extensions-java-sdk/index.html#com.scleradb.java.dbms.location.Location))
    - Represents the underlying database system.
    - Provides the configuration parameters, and properties (e.g. temporary or persistent, read-only or read-write, etc.).
    - Provides the [driver](#statementdriver) for interfacing with the underlying system.
- <a class="anchor" name="statementdriver"></a> `StatementDriver` ([Scala](http://scleradb.github.io/sclera-core-sdk/index.html#com.scleradb.dbms.driver.StatementDriver), [Java](http://scleradb.github.io/sclera-extensions-java-sdk/index.html#com.scleradb.java.dbms.driver.StatementDriver))
    - Executes the statements provided by Sclera on the underlying database system, and passes back the results.
    - Provides the [metadata driver](#statementmetadatadriver) for accessing the metadata for the data stored in the underlying database system.
- <a class="anchor" name="statementmetadatadriver"></a> `StatementMetadataDriver` ([Scala](http://scleradb.github.io/sclera-core-sdk/index.html#com.scleradb.dbms.driver.StatementMetadataDriver), [Java](http://scleradb.github.io/sclera-extensions-java-sdk/index.html#com.scleradb.java.dbms.driver.StatementMetadataDriver))
    - Provides the metadata for the data stored in the underlying database system.

### Special case: Relational Databases
If the underlying system is a relational database system, which talks SQL and uses JDBC as the interface, Sclera does most of the work. For such systems, you need to provide implementations of the following:

- <a class="anchor" name="dbservice"></a> `DBService` ([Scala](http://scleradb.github.io/sclera-core-sdk/index.html#com.scleradb.dbms.service.DataService), [Java](http://scleradb.github.io/sclera-extensions-java-sdk/index.html#com.scleradb.java.dbms.service.DBService))
    - Provides the relational database system as a service to Sclera.
- <a class="anchor" name="rdbmslocation"></a> `Location` ([Scala, Java](http://scleradb.github.io/sclera-core-sdk/index.html#com.scleradb.dbms.rdbms.location.RdbmsLocation))
    - Represents the underlying relational database system.
    - Provides a standard implementation of [`StatementDriver`](#statementdriver) and [`StatementMetadataDriver`](#statementmetadatadriver), based on JDBC. You only need to configure the JDBC configuration parameters (e.g. the JDBC URL) for the underlying system, and additional location properties (e.g. temporary or persistent, read-only or read-write, etc.).
    - Provides the [SQL mapper](#sqlmapper) for translating Sclera's internal SQL representation to the SQL for the underlying system.
- <a class="anchor" name="sqlmapper"></a> `SqlMapper` ([Scala](http://scleradb.github.io/sclera-core-sdk/index.html#com.scleradb.sql.mapper.SqlMapper), [Java](http://scleradb.github.io/sclera-extensions-java-sdk/index.html#com.scleradb.java.sql.mapper.SqlMapper))
    - Translates Sclera's internal SQL representation to the SQL for the underlying system. This is needed because the [SQL clauses and constructs across different systems vary significantly and sometimes do not follow the standard](http://troels.arvin.dk/db/rdbms/).

The [Sclera - MySQL Connector](../setup/components.md#sclera-mysql), included with the Sclera platform, is open source and implements the relational database interface mentioned above.
 
## Packaging and Deploying the Connector

The included [Sclera - MySQL Connector](../setup/components.md#sclera-mysql) implementation uses [sbt](http://www.scala-sbt.org) for building the connector [(installation details)](http://www.scala-sbt.org/release/docs/Getting-Started/Setup.html#installing-sbt). This is not a requirement -- any other build tool can be used instead.

### Dependencies

For Scala:

- The Scala implementation has a dependency on the [`"sclera-core"` library](../sdk/sdkintro.md#scalasdk). This library is available from the [Sclera repository](http://scleradb.releases.s3.amazonaws.com). Note that the dependency should be annotated `"provided"` since the `jar` for `"sclera-core"` will be available in the `CLASSPATH` when this connector is run with Sclera.

For Java:

- The Java implementation has a dependency on the [`"sclera-core"` library](../sdk/sdkintro.md#scalasdk), as wells as on the [`"sclera-extensions-java-sdk"` library](#javasdk). These libraries are available from the [Sclera repository](http://scleradb.releases.s3.amazonaws.com). The dependency on `"sclera-core"` should be annotated `"provided"` since the `jar` for `"sclera-core"` will be available in the `CLASSPATH` when this connector is run with Sclera.

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

**Note:** Please ensure that the identifier you assign to the connector is unique, that is - does not conflict with the identifier of any other available [`DBService`](#dbservice) instance.
