This document provides the information you need to know to start using Sclera's embedded JDBC driver. The Sclera JDBC driver provides a standard API to manage and analyze your data from within a Java (more generally, JVM) application.

This document is not a tutorial on how to use JDBC. JDBC is a Java standard; to get started on the basics, please refer to the the excellent tutorials and references provided by [Oracle](http://docs.oracle.com/javase/tutorial/jdbc/basics/index.html) and [PostgreSQL](http://jdbc.postgresql.org/documentation/92/index.html). Also, the [Wikipedia entry on JDBC](http://en.wikipedia.org/wiki/Java_Database_Connectivity) gives a quick overview.

## Setting the `CLASSPATH`
To interface with Sclera, you need to include Sclera's core library and the subscribed component JAR files in the `CLASSPATH`.

The bash script `$SCLERA_HOME/bin/setclasspath.sh` sets the required `CLASSPATH`; you need to `source` it into the current environment as (in bash):

    source $SCLERA_HOME/bin/setclasspath.sh

In the above, `$SCLERA_HOME` is the [directory where Sclera is installed](../setup/install.md#sclera-home).

## JDBC URL
The JDBC driver is accessed through the URL `jdbc:scleradb`.

## Supported Statements
The JDBC driver accepts all [SQL statements supported by Sclera](../sclerasql/sqlintro.md).

The queries return [`ResultSet` objects](http://docs.oracle.com/javase/tutorial/jdbc/basics/retrieving.html), as required by the standard. However, the non-query statements (`CREATE`, `INSERT`, `UPDATE` and `DELETE`), which may be [required by the standard](http://jdbc.postgresql.org/documentation/92/update.html) to return the number of rows inserted or updated, may not return the correct number; this is because the underlying sources with non-SQL/JDBC interfaces (such as NoSQL datastores) may not return the required information.

## Limitations
The JDBC support is partial (for instance, functions related to transaction processing and cursors are not supported, and only forward scans of resultsets are permitted). However, the supported API should suffice for most analytics applications, and for interfacing with most JDBC-compliant BI tools.

*A complete list of limitations will be posted here soon.*

## Driver Type
The driver is compatible with [JDBC type 4](http://en.wikipedia.org/wiki/JDBC_driver#Type_4_Driver_-_Database-Protocol_Driver.28Pure_Java_Driver.29), in the sense that it is pure Java and is platform independent.

## Connecting Sclera with your Existing Applications and Reporting Tools
The JDBC support also enables your existing applications and reporting tools to work with Sclera.

You need to [set the classpath](#setting-the-classpath) to make Sclera visible to the application. Note that Sclera's SQL is largely compatible with PostgreSQL, so you can mention PostgreSQL when asked, but actually use Sclera's JDBC driver.

Some applications and reporting tools can only accept (or generate) [standard SQL](../sclerasql/sqlregular.md). Such tools do not accept the [SQL extensions](../sclerasql/sqlintro.md) needed to perform advanced analytics within your queries. To use such extensions, you need to define [views](../sclerasql/sqlregular.md#creating-views) within Sclera to perform the required analytics, and then have the application or reporting tool use these views in its queries.

The exact directions depend upon the specific application that you want to link. If you face a problem in linking your application to Sclera, please let us know by sending an email to support@scleradb.com, or [filing a support ticket](/support).
