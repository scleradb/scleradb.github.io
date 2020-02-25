Sclera is an open platform that can be extended, as needed, with extensions that connect Sclera with multiple data sources, platforms and analytics libraries. The [components documentation](/doc/ref/components#extensions) describes the pre-built extensions that are packaged with the Sclera installation.

These custom connectors can be built using the Sclera Extensions Software Development Kit (SDK). Sclera provides separate Scala and Java variants of the SDK -- the Scala SDK is provided natively by the [`sclera-core` component](/doc/ref/components#sclera-core), while the Java SDK is a wrapper over the native Scala SDK that takes care of Java-Scala interoperability.

The SDK documentation, with illustrative source code for sample extensions, is publicly available on GitHub:

- <a class="anchor" name="scalasdk"></a> For Scala:
    - SDK for building extensions: [scleradb.github.io/sclera-core-sdk/index.html](http://scleradb.github.io/sclera-core-sdk/index.html)
    - Examples of extensions in Scala built using this SDK: [github.com/scleradb/sclera-extensions-scala](https://github.com/scleradb/sclera-extensions-scala)
- <a class="anchor" name="javasdk"></a> For Java:
    - SDK for building extensions: [scleradb.github.io/sclera-extensions-java-sdk/index.html](http://scleradb.github.io/sclera-extensions-java-sdk/index.html)
    - Examples of extensions in Java built using this SDK: [github.com/scleradb/sclera-extensions-java](https://github.com/scleradb/sclera-extensions-java)

The following type of components are covered:

## Data Access Connectors

These connectors enable ingestion of data from arbitrary sources in a ScleraSQL query. You only need to format the data as rows of a table, and Sclera will take care of evaluating streaming SQL queries on the same -- these queries can include transforming, filtering and aggregating this data, as well as joining this data with data ingested from other connectors, or with data in tables stored in other data stores.

[Sclera - Stock Ticker Connector](/doc/ref/components#sclera-stockticker), [Sclera - CSV Connector](/doc/ref/components#sclera-csv), and [Sclera - Text Files Connector](/doc/ref/components#sclera-textfiles) are built using this SDK. For examples of how these connectors are used in Sclera, please refer to the [SQL documentation](/doc/ref/sqlextdataaccess).

To learn more about building data access connectors, please see the [Sclera Datasource Extensions SDK](/doc/sdk/sdkextdataaccess) documentation.

## Database System Connectors

These connectors interface Sclera with an arbitrary database system, relational or non-relational, providing access to the underlying data, and also enable Sclera to push down computations in relevant parts user queries and commands on the interfaced database system.

[Sclera - Oracle Connector](/doc/ref/components#sclera-oracle), [Sclera - MySQL Connector](/doc/ref/components#sclera-mysql), [Sclera - PostgreSQL Connector](/doc/ref/components#sclera-postgresql), and [Sclera - Apache HBase Connector](/doc/ref/components#sclera-hbase) are built using this SDK.  For examples of how these connectors are used in Sclera, please refer to the [data platform connection reference documentation](/doc/ref/dbms).

To learn more about building database system connectors, please see the [Sclera Database System Extensions SDK](/doc/sdk/sdkextdbms) documentation.

## Machine Learning Library Connectors

These connectors interface Sclera with an arbitrary machine learning libraries, which provide implementations of classification, clustering and/or association rules mining. These connectors handle the invocation of the underlying library for training the models, and using them to label data in the query processing pipeline.

[Sclera - Weka Connector](/doc/ref/components#sclera-weka) and [Sclera - Apache Mahout Connector](/doc/ref/components#sclera-mahout) are built using this SDK. For examples of how these connectors are used in Sclera, please refer to the [SQL documentation](/doc/ref/sqlextml).

To learn more about building machine learning library connectors, please see the [Sclera Machine Learning Library Extensions SDK](/doc/sdk/sdkextml) documentation.

## Text Analytics Library Connectors

These connectors interface Sclera with an arbitrary text analytics libraries, to perform specific text analytics tasks. These connectors handle the invocation of the underlying library to process text data in table columns, in the query processing pipeline.

[Sclera - Apache OpenNLP Connector](/doc/ref/components#sclera-opennlp) is built using this SDK. For examples on how these connectors are used in Sclera, please refer to the [SQL documentation](/doc/ref/sqlexttext).

To learn more about building text analytics library connectors, please see the [Sclera Text Analytics Library Extensions SDK](/doc/sdk/sdkexttext) documentation.
