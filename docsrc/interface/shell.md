The Sclera command line shell provides an interactive interface to manage and analyze your data, and also explore the Sclera metadata.

The shell can be started by executing the script `$SCLERA_HOME/bin/sclera`, where `$SCLERA_HOME` is the [directory where Sclera is installed](../setup/install.md#sclera-home).

This section lists the commands that are accepted by the shell. The commands can be given on the command line, and executed interactively. Or, they can be put in a script and executed using the `source` command.

The commands can span multiple lines, and are terminated by a semicolon (`;`).

In the following, the keywords appear in upper case to distinguish them from the other terms; however, Sclera is case-insensitive and keywords in actual commands and queries can be in upper or lower case.

## Configuration Management
This set of commands enable you to manage the configuration parameters.

### Setting the default and cache locations
The following command sets [the `DEFAULT` location](../setup/configuration.md#sclera-location-default) to a predefined location with name specified in `location_name`:

    SET DEFAULT LOCATION = location_name;

Similarly, the following command sets [the `CACHE` location](../intro/technical.md#cache-store) to a predefined location with name specified in `location_name`:

    SET CACHE LOCATION = location_name;

In either case, the `location_name` must be defined earlier using the [`ADD LOCATION` command](../setup/dbms.md#connecting-to-database-systems).

The update is for the current shell session only. It does not affect the locations used in a concurrent session, or when Sclera is [accessed through `JDBC`](../interface/jdbc.md). To persistently change the locations, please update the [`sclera.location.datacache`](../setup/configuration.md#sclera-location-datacache) and [`sclera.location.default`](../setup/configuration.md#sclera-location-default) configuration parameters.

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
The following command shows the current [configuration settings](../setup/configuration.md):

    SHOW CONFIG;

The following command shows the current logging configuration:

    LOGGERCONFIG;

## Metadata Management
Commands to manage (add and remove) data sources and underlying tables are covered in the [Sclera Database System Connection Reference](../setup/dbms.md) document.These commands can be submitted on the command line prompt.

### Creating and Dropping Metadata Tables
In addition, the following command creates the metadata (aka schema) tables on the [designated metadata store](../setup/configuration.md#sclera-location-schema-database):

    CREATE SCHEMA;

This is needed, for instance, if you want to change the [location of the metadata store](../setup/configuration.md#sclera-location-schema-database).

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
| (not specified) | All objects ([tables](../sclerasql/sqlregular.md#creating-base-tables) across all [locations](../setup/dbms.md#location), [views](../sclerasql/sqlregular.md#creating-views), [classifiers](../sclerasql/sqlextml.md#classification), [clusterers](../sclerasql/sqlextml.md#clustering) and [associators](../sclerasql/sqlextml.md#association-rule-mining)) |
| `REMAINING location_name` | All [tables](../sclerasql/sqlregular.md#creating-base-tables) in [location](../setup/dbms.md#location) `location_name` that have not been added  |
| `[ TABLE ] location_name.*` | All [tables](../sclerasql/sqlregular.md#creating-base-tables) in [location](../setup/dbms.md#location) `location_name` that have already been added |
| `[ TABLE ] location_name.table_name` | [Table](../sclerasql/sqlregular.md#creating-base-tables) with name `table_name` added to location `location_name`, if it exists; otherwise empty |
| `TABLE` | All [tables](../sclerasql/sqlregular.md#creating-base-tables) across all locations |
| `TABLE table_name` | All [tables](../sclerasql/sqlregular.md#creating-base-tables) with name `table_name` across all locations |
| `VIEW` | All [views](../sclerasql/sqlregular.md#creating-views) |
| `VIEW view_name` | [View](../sclerasql/sqlregular.md#creating-views) with name `view_name`, if it exists; otherwise empty |
| `CLASSIFIER` | All [classifiers](../sclerasql/sqlextml.md#classification) |
| `CLASSIFIER classifier_name` | [Classifier](../sclerasql/sqlextml.md#classification) with name `classifier_name`, if it exists; otherwise empty |
| `CLUSTERER` | All [clusterers](../sclerasql/sqlextml.md#clustering) |
| `CLUSTERER clusterer_name` | [Clusterer](../sclerasql/sqlextml.md#clustering) with name `clusterer_name`, if it exists; otherwise empty |
| `LOCATION` | All [locations](../setup/dbms.md#location) |

## ScleraSQL Commands
ScleraSQL queries and commands accepted by Sclera are discussed in the [ScleraSQL Reference](../sclerasql/sqlintro.md) document. These can be submitted on the command line prompt. Commands are executed silently, while returned query results are displayed in a table format.

### Looking Deeper with the Explain Command
In addition, the shell has an `EXPLAIN` command that explains a query is executed by Sclera. The `EXPLAIN` command comes in two variants.

#### Run-time Explain
The first variant, called runtime explain has been discussed [above](#activating-and-deactivating-runtime-explain). This command, called `EXPLAIN SCRIPT`, causes each ScleraSQL command or query to display the commands it executes on the underlying database systems.

This variant of `EXPLAIN` is useful when you want to trace what is happening under the hood while a query or command is executing. We refer to the [earlier discussion](#activating-and-deactivating-runtime-explain) for the details.

#### Compile-time Explain
The second variant, called the compile-time explain, has the following syntax:

    EXPLAIN table_expression;

This variant takes a [table expression `table_expression`](../sclerasql/sqlregular.md#table-expression) (i.e. [a SQL query](../sclerasql/sqlregular.md#querying-data-using-generalized-table-expressions)) as a parameter.

The `table_expression` is [parsed and optimized by the query processor](../intro/technical.md#query-processor), and the resulting plan is then displayed on the console. Note that unlike the `EXPLAIN SCRIPT`, the query is not executed; the output shows how Sclera *plans* to execute the query if given without the `EXPLAIN`.

This variant is useful when you want to explore how a query will be evaluated by Sclera, without actually evaluating the same.

## Script Execution

This command enables you to execute a script of commands from a file. The syntax is:

    SOURCE script_file_path;

where `script_file_path` is the full path of the command script to be executed.

The file is read, and each command therein is executed in sequence; the output of the command, if any, is displayed on the console as the command is executed.

## Usability Features
The shell maintains a command history, at a location given by the [`sclera.shell.history` configuration parameter](../setup/configuration.md#sclera-shell-history). You can navigate the history using the up/down keys.

You can complete words by pressing tab. The word completion alternatives presented are not context sensitive, but that may change in future versions.

## Reset Command
To reset all connections and recover to a clean state, issue the following command:

    RESET;

This is equivalent to exiting and restarting the shell.

## Comments
A line with two consecutive hyphens (`"--"`) as the first non-whitespace characters is considered a comment, and is ignored. Unlike standard SQL and PostgreSQL, comments cannot start midway in a line, after a valid input.

Comments are only permitted in the shell, and in scripts input to the shell.
