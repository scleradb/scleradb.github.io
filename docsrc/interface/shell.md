The Sclera command line shell provides an interactive interface to manage and analyze your data, and also explore the Sclera metadata. The command-line is a lightweight alternative to the [Sclera Visual Shell](/doc/ref/visualshell) and does not support the [Sclera Visualization](/doc/ref/visualization) language extensions.

The shell can be started by executing the script `$SCLERA_HOME/bin/sclera.sh`, where `$SCLERA_HOME` is the [directory where Sclera is installed](/doc/ref/install#sclera-home).

This section lists the commands that are accepted by the shell. The commands can be given on the command line, and executed interactively. Or, they can be put in a script and executed using the `source` command.

The commands can span multiple lines, and are terminated by a semicolon (`;`).

In the following, the keywords appear in upper case to distinguish them from the other terms; however, Sclera is case-insensitive and keywords in actual commands and queries can be in upper or lower case.

## Configuration Management
This set of commands enable you to manage the configuration parameters.

### Setting the default and cache locations
The following command sets [the `DEFAULT` location](/doc/ref/configuration#sclera-location-default) to a predefined location with name specified in `location_name`:

    SET DEFAULT LOCATION = location_name;

Similarly, the following command sets [the `CACHE` location](/doc/ref/technical#cache-store) to a predefined location with name specified in `location_name`:

    SET CACHE LOCATION = location_name;

In either case, the `location_name` must be defined earlier using the [`ADD LOCATION` command](/doc/ref/dbms#connecting-to-database-systems).

The update is for the current shell session only. It does not affect the locations used in a concurrent session, or when Sclera is [accessed through `JDBC`](/doc/ref/jdbc). To persistently change the locations, please update the [`sclera.location.datacache`](/doc/ref/configuration#sclera-location-datacache) and [`sclera.location.default`](/doc/ref/configuration#sclera-location-default) configuration parameters.

### Activating and Deactivating Runtime Explain
The queries and commands on the command line are translated by Sclera into a sequence of subqueries and subcommands that execute on the underlying database systems. The `EXPLAIN SCRIPT` command enables you to activate or decativate the display of these commands on the console as they are executed.

The following command activates this runtime explain feature:

    EXPLAIN SCRIPT [ ON ];

The `ON` at the end is a syntactic sugar, and can be omitted.

The following command deactivates this runtime explain feature:

    EXPLAIN SCRIPT OFF;

### Display the Parameter Settings
The following command shows the current default location, cache location and runtime explain settings:

    SHOW OPTIONS;

### Display the Configuration Parameters
The following command shows the current [configuration settings](/doc/ref/configuration):

    SHOW CONFIG;

The following command shows the current logging configuration:

    LOGGERCONFIG;

## Metadata Management
Commands to manage (add and remove) data sources and underlying tables are covered in the [Sclera Database System Connection Reference](/doc/ref/dbms) document.These commands can be submitted on the command line prompt.

### Creating and Dropping Metadata Tables
In addition, the following command creates the metadata (aka schema) tables on the [designated metadata store](/doc/ref/configuration#sclera-location-schema-database):

    CREATE SCHEMA;

This is needed, for instance, if you want to change the [location of the metadata store](/doc/ref/configuration#sclera-location-schema-database).

Also, the following command deletes the tables in the designated metadata store:

    DROP SCHEMA;

### Exploring Metadata

You can explore Sclera's metadata (the locations, tables, views, and other objects) using the `LIST` and `DESCRIBE` commands.

    LIST [ list_spec ];
    DESCRIBE [ list_spec ];

The two commands are identical, except that `LIST` outputs a list of objects obtained using `list_spec` (see below) in a short format, whereas `DESCRIBE` outputs the same list of objects in a more detailed, descriptive, format.

The following table presents the possible values of the optional parameter `list_spec` and the associated list of objects to be output.

| `list_spec` | Object List |
| ----------- | --------------- |
| (not specified) | All objects ([tables](/doc/ref/sqlregular#creating-base-tables) across all [locations](/doc/ref/dbms#location), [views](/doc/ref/sqlregular#creating-views), [classifiers](/doc/ref/sqlextml#classification), [clusterers](/doc/ref/sqlextml#clustering) and [associators](/doc/ref/sqlextml#association-rule-mining)) |
| `REMAINING location_name` | All [tables](/doc/ref/sqlregular#creating-base-tables) in [location](/doc/ref/dbms#location) `location_name` that have not been added  |
| `[ TABLE ] location_name.*` | All [tables](/doc/ref/sqlregular#creating-base-tables) in [location](/doc/ref/dbms#location) `location_name` that have already been added |
| `[ TABLE ] location_name.table_name` | [Table](/doc/ref/sqlregular#creating-base-tables) with name `table_name` added to location `location_name`, if it exists; otherwise empty |
| `TABLE` | All [tables](/doc/ref/sqlregular#creating-base-tables) across all locations |
| `TABLE table_name` | All [tables](/doc/ref/sqlregular#creating-base-tables) with name `table_name` across all locations |
| `VIEW` | All [views](/doc/ref/sqlregular#creating-views) |
| `VIEW view_name` | [View](/doc/ref/sqlregular#creating-views) with name `view_name`, if it exists; otherwise empty |
| `CLASSIFIER` | All [classifiers](/doc/ref/sqlextml#classification) |
| `CLASSIFIER classifier_name` | [Classifier](/doc/ref/sqlextml#classification) with name `classifier_name`, if it exists; otherwise empty |
| `CLUSTERER` | All [clusterers](/doc/ref/sqlextml#clustering) |
| `CLUSTERER clusterer_name` | [Clusterer](/doc/ref/sqlextml#clustering) with name `clusterer_name`, if it exists; otherwise empty |
| `ASSOCIATOR` | All [associators](/doc/ref/sqlextml#association-rule-mining) |
| `ASSOCIATOR associator_name` | [Associator](/doc/ref/sqlextml#association-rule-mining) with name `associator_name`, if it exists; otherwise empty |
| `LOCATION` | All [locations](/doc/ref/dbms#location) |

## ScleraSQL Commands
ScleraSQL queries and commands accepted by Sclera are discussed in the [ScleraSQL Reference](/doc/ref/sqlintro) document. These can be submitted on the command line prompt. Commands are executed silently, while returned query results are displayed in a table format.

### Looking Deeper with the Explain Command
In addition, the shell has an `EXPLAIN` command that explains a query is executed by Sclera. The `EXPLAIN` command comes in two variants.

#### Run-time Explain
The first variant, called runtime explain has been discussed [above](#activating-and-deactivating-runtime-explain). This command, called `EXPLAIN SCRIPT`, causes each ScleraSQL command or query to display the commands it executes on the underlying database systems.

This variant of `EXPLAIN` is useful when you want to trace what is happening under the hood while a query or command is executing. We refer to the [earlier discussion](#activating-and-deactivating-runtime-explain) for the details.

#### Compile-time Explain
The second variant, called the compile-time explain, has the following syntax:

    EXPLAIN table_expression;

This variant takes a [table expression `table_expression`](/doc/ref/sqlregular#table-expression) (i.e. [a SQL query](/doc/ref/sqlregular#querying-data-using-generalized-table-expressions)) as a parameter.

The `table_expression` is [parsed and optimized by the query processor](/doc/ref/technical#query-processor), and the resulting plan is then displayed on the console. Note that unlike the `EXPLAIN SCRIPT`, the query is not executed; the output shows how Sclera *plans* to execute the query if given without the `EXPLAIN`.

This variant is useful when you want to explore how a query will be evaluated by Sclera, without actually evaluating the same.

## Script Execution

This command enables you to execute a script of commands from a file. The syntax is:

    SOURCE script_file_path;

where `script_file_path` is the full path of the command script to be executed.

The file is read, and each command therein is executed in sequence; the output of the command, if any, is displayed on the console as the command is executed.

## Usability Features
The shell maintains a command history, at a location given by the [`sclera.shell.history` configuration parameter](/doc/ref/configuration#sclera-shell-history). You can navigate the history using the up/down keys.

You can complete words by pressing tab. The word completion alternatives presented are not context sensitive, but that may change in future versions.

## Reset Command
To reset all connections and recover to a clean state, issue the following command:

    RESET;

This is equivalent to exiting and restarting the shell.

## Comments
A line with two consecutive hyphens (`"--"`) as the first non-whitespace characters is considered a comment, and is ignored. Unlike standard SQL and PostgreSQL, comments cannot start midway in a line, after a valid input.

Comments are only permitted in the shell, and in scripts input to the shell.

## JDBC/ODBC Server

Starting with Sclera 2.2, the shell can be used to start a TCP server that implements the [PostgreSQL backend protocol](http://www.postgresql.org/docs/9.4/static/protocol-overview.html) 3.0, which is compatible with PostgreSQL 7.4+.

Though this server, Sclera can interface with the latest [PostgreSQL ODBC](https://odbc.postgresql.org/) and [JDBC](https://jdbc.postgresql.org/) drivers, [PostgreSQL’s shell (`psql`)](http://www.postgresql.org/docs/9.4/static/app-psql.html), and anything else that uses [PostgreSQL’s native protocol (libpq)](http://www.postgresql.org/docs/9.4/static/libpq.html).

Details on how to start/stop this server and configure the connections appear in a [separate document](/doc/ref/server).

## User and Password Management

Starting with Sclera 2.2, the shell can be used to add a password, and to create and remove users. This is discussed in a [separate document](/doc/ref/users).

