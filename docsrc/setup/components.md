This document lists the various components of a Sclera installation.

## Required Components
This section lists the components that are required for Sclera to work.

<a class="anchor" name="sclera-core"></a>
### Sclera - Core Engine
This is the core Sclera engine, which is responsible for parsing, optimizing and evaluating [SQL commands and queries](../sclerasql/sqlintro.md) with the help of the other components. For the details, please see the [technical details](../intro/technical.md) document.

This component includes an embedded [H2 database](http://www.h2database.com), which serves as the default [metadata store](../intro/technical.md#metadata-store) and [data cache](../intro/technical.md#cache-store).

## Extensions
The components listed in this section are optional extensions -- they are not core to the working of the Sclera engine.

<a class="anchor" name="sclera-shell"></a>
### Sclera - Command Line Shell
This component provides a command-line shell for interactive SQL processing.

This shell accepts SQL queries and returns the result in a formatted manner. In addition, it supports administrative commands, and also additional commands for querying the metadata.

For the details on how to use the command line shell, please refer to the [Sclera Command Line Shell Reference](../interface/shell.md) document.

<a class="anchor" name="sclera-jdbc"></a>
### Sclera - JDBC Driver
This component provides an embedded [JDBC](http://en.wikipedia.org/wiki/Java_Database_Connectivity) 4 interface to Sclera.

The JDBC support is partial (for instance, functions related to transaction processing are not supported, and only forward scans of resultsets are permitted). However, the supported API should suffice for most analytics applications, and for interfacing with most JDBC-compliant BI tools.

A detailed description on how to use the JDBC API appears in the [Sclera JDBC Reference](../interface/jdbc.md) document.

<a class="anchor" name="sclera-oracle"></a>
### Sclera - Oracle Connector
This component enables Sclera to work with your data stored in [Oracle](http://www.oracle.com).

You just need to link your Oracle database with Sclera, then import the metadata of select tables within the database. All this gets done in a couple of commands -- and enables you to include these tables within your Sclera queries.

The link uses the [Oracle Thin JDBC Driver](http://www.oracle.com/technetwork/database/features/jdbc/jdbc-drivers-12c-download-1958347.html), which is *not* downloaded as a part of the installation of this component. You need to download the driver manually before using this component.

Details on how to link your Oracle source to with Sclera can be found in the [Sclera Database System Connection Reference](../setup/dbms.md#connecting-to-oracle) document.

<a class="anchor" name="sclera-mysql"></a>
### Sclera - MySQL Connector
*To work with Sclera, MySQL should be configured in the [case-insensitive mode](http://dev.mysql.com/doc/refman/5.6/en/identifier-case-sensitivity.html).*

This component enables Sclera to work with your data stored in [MySQL](http://www.mysql.com).

You just need to link your MySQL database with Sclera, then import the metadata of select tables within the database. All this gets done in a couple of commands -- and enables you to include these tables within your Sclera queries.

The connector uses [MySQL Connector/J](http://dev.mysql.com/doc/connector-j/en/index.html), which is automatically downloaded during the installation of this component.

Details on how to link your MySQL source to with Sclera can be found in the [Sclera Database System Connection Reference](../setup/dbms.md#connecting-to-mysql) document.

<a class="anchor" name="sclera-mysql-license"></a> **Important**
The [MySQL Connector/J](http://dev.mysql.com/doc/connector-j/en/index.html) JDBC driver is licensed under the [GNU General Public License version 2 with FOSS exception](https://oss.oracle.com/licenses/universal-foss-exception/). Sclera - MySQL Connector is licensed under the [Apache License version 2.0](http://www.apache.org/licenses/LICENSE-2.0), which is compatible with the said FOSS exception.

<a class="anchor" name="sclera-postgresql"></a>
### Sclera - PostgreSQL Connector
This component enables Sclera to work with your data stored in [PostgreSQL](http://www.postgresql.org).

You just need to link your PostgreSQL database with Sclera, then import the metadata of select tables within the database. All this gets done in a couple of commands -- and enables you to include these tables within your Sclera queries.

The link uses the [PostgreSQL JDBC Driver](http://jdbc.postgresql.org), which is downloaded as a part of the installation of this component.

Details on how to link your PostgreSQL source to with Sclera can be found in the [Sclera Database System Connection Reference](../setup/dbms.md#connecting-to-postgresql) document.

<a class="anchor" name="sclera-heroku"></a>
### Sclera - Heroku PostgreSQL Connector
This component enables Sclera to work with your data stored in [PostgreSQL](http://www.postgresql.org) database [hosted at Heroku](https://www.heroku.com/postgres).

You just need to link your Heroku PostgreSQL database with Sclera, then import the metadata of select tables within the database. All this gets done in a couple of commands -- and enables you to include these tables within your Sclera queries.

Details on how to link your Heroku PostgreSQL source to with Sclera can be found in the [Sclera Database System Connection Reference](../setup/dbms.md#connecting-to-heroku-postgres) document.

<a class="anchor" name="sclera-csv"></a>
### Sclera - CSV File Connector
This component enables Sclera to work with your data stored on your disk as [CSV files](http://en.wikipedia.org/wiki/Comma-separated_values).

The CSV files are viewed as tables, and can be accessed in a manner similar to tables in a SQL query. You can also join the CSV file with tables in your database, with other CSV files, and aggregate the data as needed.

Further details on how to use the connector are in the [ScleraSQL Reference](../sclerasql/sqlextdataaccess.md#sclera-csv) document.

This is a sample component showcasing Sclera's ability to interface with external data. For the implementation details, please see the [Sclera Data Access Connector Development](../sdk/sdkextdataaccess.md) document.

<a class="anchor" name="sclera-textfiles"></a>
### Sclera - Text File Connector
This component enables Sclera to work with free-form text files.

The text files are viewed as tables, with two columns: an identifier column containing the file's path, and another column containing the file's contents. These files can now be accessed in a manner similar to tables in a SQL query.

A limitation for the current version is that the contents must be less than 255 characters; this limitation will be removed in later versions of the component.

A common use case is to use this in conjunction with the [Sclera - OpenNLP Connector](#sclera-opennlp) which can be used to extract entities from the file contents.

For details on how to use the connector, please see the [ScleraSQL Reference](../sclerasql/sqlextdataaccess.md#sclera-textfiles) document.

This is a sample component showcasing Sclera's ability to interface with external data. For the implementation details, please see the [Sclera Data Access Connector Development](../sdk/sdkextdataaccess.md) document.

<a class="anchor" name="sclera-opennlp"></a>
### Sclera - Apache OpenNLP Connector
This component enables Sclera to perform text analytics on free-form text.

Current version of this component only supports extracting entities (such as names of persons and places, dates, emails) from the text. Later versions will include additional features such as sentiment/opinion mining.

The entity extraction is exposed as a SQL operator (Sclera's extension) which can act on any relational input. The operator is given the name of the column containing the text data, and the output is the input will additional columns containing the extracted information. The output can then be aggregated, joined with other tables, etc. as usual within the SQL query.

This component uses the [Apache OpenNLP](http://opennlp.apache.org) library, which is downloaded automatically as a part of the installation.

To use this component, you will also need to provide Sclera with trained models for a sentence detector and name finders (extractors) for your language. These are not packaged with Sclera, but can be downloaded separately from the [Apache OpenNLP models repository](http://opennlp.sourceforge.net/models-1.5/). The site provides models in Danish (code: `da`), German (code: `de`), English (code: `en`), Dutch (code: `dl`), Portuguese (code: `pt`) and Swedish (code: `se`). The models files can be downloaded from the site and kept in the directory `$SCLERA_ASSETS/opennlp`, where `$SCLERA_ASSETS` is the directory given by the [`sclera.services.assetdir` configuration parameter](../setup/configuration.md#sclera-services-assetdir).

For greater accuracy on your data, you can also [create your own name finders using Apache OpenNLP's toolkit](http://opennlp.apache.org/documentation/1.5.3/manual/opennlp.html#tools.namefind.training).

Please refer to the [ScleraSQL Reference](../sclerasql/sqlexttext.md#sclera-opennlp) document for details on using the component's features in a SQL query.

<a class="anchor" name="sclera-weka"></a>
### Sclera - Weka Connector
This component enables Sclera to perform [classification](http://en.wikipedia.org/wiki/Cluster_analysis), [clustering](http://en.wikipedia.org/wiki/Cluster_analysis) and [association rule mining](http://en.wikipedia.org/wiki/Association_rule) on data from within SQL.

With this component, a classifier or a clusterer can be trained in just a single SQL command. Scoring new data using the classifier, or segmenting data using the clusterer gets done using a simple SQL operator (Sclera's extension) that seamlessly embeds within your SQL query.

The component uses the [Weka](http://www.cs.waikato.ac.nz/ml/weka) library, which is downloaded automatically as a part of the installation.

Please refer to the [ScleraSQL Reference](../sclerasql/sqlextml.md#sclera-weka) document for details on using the component's features in a SQL query.

<a class="anchor" name="sclera-weka-license"></a> **Important**
The [Weka](http://www.cs.waikato.ac.nz/ml/weka) library is licensed under the [GNU General Public License version 2](http://www.gnu.org/licenses/old-licenses/gpl-2.0.html). For [compatibility](http://www.gnu.org/licenses/gpl-faq.html#AllCompatibility), this component is licensed under the [GNU General Public License version 2](http://www.gnu.org/licenses/old-licenses/gpl-2.0.html) as well. Please use this component in accordance with this license. To get a commercial license for Weka, please refer to the [Weka FAQ](http://weka.wikispaces.com/Can+I+use+WEKA+in+commercial+applications%3F).

In keeping with the provisions of the GNU General Public License version 2, the source code for this component is available for download at the [Sclera repository](https://s3.amazonaws.com/scleradb.releases/com.scleradb/sclera-weka_2.9.3/1.0.140115-BETA/srcs/sclera-weka_2.9.3-sources.jar).

*This component is an OPTIONAL extension. As such, this component's license does NOT affect your use of any other Sclera component, or the core Sclera platform.*

<a class="anchor" name="sclera-matcher"></a>
### Sclera - Regular Expression Matcher
This component enables Sclera to efficiently and flexibly analyze ordered streaming data.

The component introduces a construct that enables matching regular expressions over streaming data, and using them to compute sophisticated aggregates. This is a powerful construct, proprietary to Sclera, and enables computations that are ridiculously hard to express and expensive to compute using standard SQL.

For details and examples on using these constructs in a SQL query, please refer to the [ScleraSQL Reference](../sclerasql/sqlextordered.md) document.

<a class="anchor" name="sclera-visualization"></a>
### Sclera - Visual Shell / ScleraViz
This component enables users to use Sclera in a web-browser. This enables a richer, more visual experience with extensive support for data visualization.

Specifically, you can run queries and display the results as a table, or use a very expressive graphics language to plot the results as regular, multilayered and faceted graphs in just a few lines of code. The graph specification language is inspired by the "Grammar of Graphics" (implemented in `R` as [ggplot2](http://ggplot2.org)), and is rendered using [D3](http://d3js.org) in [SVG](https://en.wikipedia.org/wiki/Scalable_Vector_Graphics).

Unlike `ggplot2`, the resulting plots are interactive, and can display streaming data in a continuous manner. Moreover, the specification language is well-integrated with [ScleraSQL](../sclerasql/sqlintro.md).

For details and examples on using these constructs, please refer to the [ScleraSQL Visualization Reference](../sclerasql/visualization.md) document.
