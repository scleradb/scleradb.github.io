Sclera is an open platform that can be extended, as needed, with extensions that connect Sclera with multiple data sources, platforms and analytics libraries. The [components documentation](../setup/components.md#optional-extensions) describes the pre-built extensions that are packaged with the Sclera installation.

These custom connectors can be built using the [Sclera Extensions Software Development Kit (SDK)](https://www.javadoc.io/doc/com.scleradb/sclera-core_2.13/latest/com/scleradb/index.html), provided natively by the [`sclera-core` component](../setup/components.md#sclera-core).

For details on the API, please refer to the [`sclera-core` scaladoc](https://www.javadoc.io/doc/com.scleradb/sclera-core_2.13/latest/com/scleradb/index.html).

The following type of components are covered:

## Data Access Connectors

These connectors enable ingestion of data from arbitrary sources in a ScleraSQL query. You only need to format the data as rows of a table, and Sclera will take care of evaluating streaming SQL queries on the same -- these queries can include transforming, filtering and aggregating this data, as well as joining this data with data ingested from other connectors, or with data in tables stored in other data stores.

[Sclera - CSV Connector](../setup/components.md#sclera-csv) and [Sclera - Text Files Connector](../setup/components.md#sclera-textfiles) are built using this SDK. For examples of how these connectors are used in Sclera, please refer to the [SQL documentation](../sclerasql/sqlextdataaccess.md).

To learn more about building data access connectors, please see the [Sclera Datasource Extensions SDK](../sdk/sdkextdataaccess.md) documentation.

## Database System Connectors

These connectors interface Sclera with an arbitrary database system, relational or non-relational, providing access to the underlying data, and also enable Sclera to push down computations in relevant parts user queries and commands on the interfaced database system.

[Sclera - Oracle Connector](../setup/components.md#sclera-oracle), [Sclera - MySQL Connector](../setup/components.md#sclera-mysql), and [Sclera - PostgreSQL Connector](../setup/components.md#sclera-postgresql) are built using this SDK.  For examples of how these connectors are used in Sclera, please refer to the [data platform connection reference documentation](../setup/dbms.md).

To learn more about building database system connectors, please see the [Sclera Database System Extensions SDK](../sdk/sdkextdbms.md) documentation.

## Machine Learning Library Connectors

These connectors interface Sclera with an arbitrary machine learning libraries, which provide implementations of classification, clustering and/or association rules mining. These connectors handle the invocation of the underlying library for training the models, and using them to label data in the query processing pipeline.

[Sclera - Weka Connector](../setup/components.md#sclera-weka) is built using this SDK. For examples of how these connectors are used in Sclera, please refer to the [SQL documentation](../sclerasql/sqlextml.md).

To learn more about building machine learning library connectors, please see the [Sclera Machine Learning Library Extensions SDK](../sdk/sdkextml.md) documentation.

## Text Analytics Library Connectors

These connectors interface Sclera with an arbitrary text analytics libraries, to perform specific text analytics tasks. These connectors handle the invocation of the underlying library to process text data in table columns, in the query processing pipeline.

[Sclera - Apache OpenNLP Connector](../setup/components.md#sclera-opennlp) is built using this SDK. For examples on how these connectors are used in Sclera, please refer to the [SQL documentation](../sclerasql/sqlexttext.md).

To learn more about building text analytics library connectors, please see the [Sclera Text Analytics Library Extensions SDK](../sdk/sdkexttext.md) documentation.
