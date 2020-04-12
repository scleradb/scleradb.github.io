In this document, we show how to build custom connectors to any relational/non-relational database management system. These connectors interface Sclera with an arbitrary database system, relational or non-relational, providing access to the underlying data, and also enable Sclera to push down computations in relevant parts user queries and commands on the interfaced database system.

[Sclera - Oracle Connector](../setup/components.md#sclera-oracle), [Sclera - MySQL Connector](../setup/components.md#sclera-mysql), and [Sclera - PostgreSQL Connector](../setup/components.md#sclera-postgresql) are built using this SDK.  For examples of how these connectors are used in Sclera, please refer to the [documentation on connecting Sclera to database systems](../setup/dbms.md).

## Building Database System Connectors

To build a custom datasource connector, you need to provide implementations of the following abstract classes in the SDK:

- <a class="anchor" name="dbservice"></a> `DBService` ([API Link](/api/sclera-core/com/scleradb/dbms/service/DBService.html))
    - Provides the database system as a service to Sclera.
    - Contains an `id` that identifies this service.
    - Contains a method `createLocation` that is used to create a new [location instance](#location) for this service.
- <a class="anchor" name="location"></a> `Location` ([API Link](/api/sclera-core/com/scleradb/dbms/location/Location.html))
    - Represents the underlying database system.
    - Provides the configuration parameters, and properties (e.g. temporary or persistent, read-only or read-write, etc.).
    - Provides the [driver](#statementdriver) for interfacing with the underlying system.
- <a class="anchor" name="statementdriver"></a> `StatementDriver` ([API Link](/api/sclera-core/com/scleradb/dbms/driver/StatementDriver.html))
    - Executes the statements provided by Sclera on the underlying database system, and passes back the results.
    - Provides the [metadata driver](#statementmetadatadriver) for accessing the metadata for the data stored in the underlying database system.
- <a class="anchor" name="statementmetadatadriver"></a> `StatementMetadataDriver` ([API Link](/api/sclera-core/com/scleradb/dbms/driver/StatementMetadataDriver.html))
    - Provides the metadata for the data stored in the underlying database system.

### Special case: Relational Databases
If the underlying system is a relational database system, which talks SQL and uses JDBC as the interface, Sclera does most of the work. For such systems, you need to provide implementations of the following:

- <a class="anchor" name="dbservice"></a> `DBService` ([API Link](/api/sclera-core/com/scleradb/dbms/service/DBService.html))
    - Provides the relational database system as a service to Sclera.
- <a class="anchor" name="rdbmslocation"></a> `RdbmsLocation` ([API LInk](/api/sclera-core/com/scleradb/dbms/rdbms/location/RdbmsLocation.html))
    - Represents the underlying relational database system.
    - Provides a standard implementation of [`StatementDriver`](#statementdriver) and [`StatementMetadataDriver`](#statementmetadatadriver), based on JDBC. You only need to configure the JDBC configuration parameters (e.g. the JDBC URL) for the underlying system, and additional location properties (e.g. temporary or persistent, read-only or read-write, etc.).
    - Provides the [SQL mapper](#sqlmapper) for translating Sclera's internal SQL representation to the SQL for the underlying system.
- <a class="anchor" name="sqlmapper"></a> `SqlMapper` ([API LInk](/api/sclera-core/com/scleradb/sql/mapper/SqlMapper.html))
    - Translates Sclera's internal SQL representation to the SQL for the underlying system. This is needed because the [SQL clauses and constructs across different systems vary significantly and sometimes do not follow the standard](http://troels.arvin.dk/db/rdbms/).

The [Sclera - MySQL Connector](../setup/components.md#sclera-mysql), included with the Sclera platform, is open source and implements the relational database interface mentioned above.
 
## Packaging and Deploying the Connector

The included [Sclera - MySQL Connector](../setup/components.md#sclera-mysql) implementation uses [sbt](http://www.scala-sbt.org) for building the connector [(installation details)](http://www.scala-sbt.org/release/docs/Getting-Started/Setup.html#installing-sbt). This is not a requirement -- any other build tool can be used instead.

### Dependencies

The implementation has a dependency on:

- the database system's driver (e.g. the appropriate JDBC driver for relational database systems).
- the [`"sclera-core"`](../setup/components.md#sclera-core) and [`"sclera-config"`](../setup/components.md#sclera-config) core components. Note that these dependencies is annotated `"provided"` since these libraries will already be available in the `CLASSPATH` when this connector is run with Sclera.
- (optional) the test framework [`scalatest`](http://www.scalatest.org/) for running the tests.

These are specified in the build file. As an example, see the [Sclera - MySQL Connector's build file](https://github.com/scleradb/sclera-plugin-mysql/blob/master/build.sbt).

### Deployment Steps

Follow steps similar to those described [here](../sdk/sdkextdataaccess.md#deployment-steps).

**Note:** Please ensure that the identifier you assign to the connector is unique, that is - does not conflict with the identifier of any other available [`DBService`](#dbservice) instance.
