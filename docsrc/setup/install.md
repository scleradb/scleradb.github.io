Sclera is a stand-alone SQL processor with native support for machine learning, data virtualization and streaming data. Sclera can be deployed as:

- [an independent application with an interactive command-line shell](#installing-and-maintaining-sclera-command-line-application), or
- [as a library that embeds within your applications to enable advanced real-time analytics capabilities](#embedding-sclera-in-applications).

**Prerequisite:** Sclera requires [Java 13 or higher](https://openjdk.java.net/).

*We recommend against installing Sclera with root/admin permissions. Sclera does not need root access for installation or at runtime.*

## Installing and Maintaining Sclera Command Line Application

The recommended way to install Sclera's interactive command line shell is through Sclera platform administration tool, `scleradmin`.

We first describe how to [install `scleradmin`](#installing-scleradmin). Next, we use `scleradmin` to [install Sclera's core packages and command line interface](#installing-sclera-core-packages-and-shell). We then show how to [add and remove optional Sclera plugins](#plugin-management). Finally, we show how to [update the installed core packages and plugins](#updating-installed-packages-and-plugins).

**Prerequisites:** [Python 3.8+](https://python.org/downloads) and [pip](https://pip.pypa.io/en/stable/installing/).

### Installing scleradmin

The following installs the [latest version of `scleradmin`](https://pypi.org/project/scleradmin/) as a Python package:

    > python3.8 -m pip install scleradmin

where `python3.8` stands for the executable for Python version 3.8 or above in your setup.

In the following sections, we show how to use `scleradmin` for installing Sclera and managing the installation. For full details on `scleradmin` usage, you may want to read the help information:

    > scleradmin --help

### Installing Sclera Core Packages and Shell

The following command installs Sclera:

    > scleradmin --install --root <sclera-root>

In the above, `<sclera-root>` is the directory where you want Sclera to be installed. This directory must not exist before installation, it is created by the command (this is a safeguard againt accidental overwrites). The contents of the directory after installation are described [later in this document](#root-directory-structure).

The installation involves downloading core sclera components and associated libraries. This might take a few minutes; you can monitor the progress by viewing the generated logs in `<sclera-root>/install/log/install.log`.

#### Using the Shell

The shell can be started using the following command:

    > <sclera-root>/bin/sclera

This starts the shell, wherein you can interactively run queries. When done, you can terminate the session by typing `Control-D`.

    Welcome to Sclera 4.0

    > select "Hello, world!" as greeting;
    ---------------
     GREETING
    ---------------
     Hello, world!
    ---------------
    (1 row)

    > ^D
    Goodbye!

For details on using the shell, please refer to the [Command Line Shell Reference](../interface/shell.md).

#### Root Directory Structure

After installation, the root directory has the following structure:

    [<sclera-root>]
      bin/
        sclera.cmd         # executable command file (generated for Windows systems)
        sclera             # executable bash (generated for Linux, macOS, and other Unix-based systems)
      config/
        sclera.conf        # configuration file
      extlib/              # directory for additional libraries, plugins (initially empty)
      home/
        assets/
          data/            # data stored by the embedded temporary database (H2), etc.
        history            # shell command history
        log/
          sclera.log       # execution log, contains details of runtime progress
      install/
        boot/              # specification files for sclera components (core or plugin)
        launcher*.jar      # SBT launcher, used for installing sclera components
        log/
          install.log      # installation log, contains details of installation progress
      lib/                 # installation artifacts (jars, etc.) of installed components and their dependencies

### Plugin Management

Sclera provides [a variety of plugins](../setup/components.md) that can be added using `scleradmin`. The command syntax is:

    > scleradmin --add <plugins> --root <sclera-root>

In the above, `<plugins>` is a space-separated list of plugins to be added, and `<sclera-root>`, [as earlier](#installing-sclera-core-packages-and-shell), is the root directory. For instance, to add the [Sclera - CSV File Connector](../setup/components.md#sclera-csv-file-connector) and [Sclera - Text File Connector](../setup/components.md#sclera-text-file-connector) plugins to the Sclera instance installed at `/path/to/sclera`, the command is: 

    > scleradmin --add sclera-csv-plugin sclera-textfiles-plugin --root /path/to/sclera

To remove installed plugins, the syntax is similar. The following command removes the plugins installed above:

    > scleradmin --remove sclera-csv-plugin sclera-textfiles-plugin --root /path/to/sclera

You can specify a list of plugins to add and another list of plugins to remove in the same command.

For a list of available plugins and other components, please refer to the [components documentation](../setup/components.md).

### Updating Installed Packages and Plugins

The following command updates Sclera's core packages as well as the plugins to the latest version:

    > scleradmin --update --root <sclera-root>

where `<sclera-root>`, [as mentioned earlier](#installing-sclera-core-packages-and-shell), is the root directory.

## Embedding Sclera in Applications

Sclera's JDBC driver [sclera-jdbc](https://github.com/scleradb/sclera/tree/master/modules/interfaces/jdbc) provides a [JDBC](http://en.wikipedia.org/wiki/Java_Database_Connectivity) type 4 interface. Applications can therefore interface with Sclera using [JDBC API](https://docs.oracle.com/javase/tutorial/jdbc/overview/index.html).

Since JDBC is a well-known standard, the application is written the same way as for any other JDBC compliant database system. The difference is that the queries that the application can submit are now in the much richer [Sclera SQL](../sclerasql/sqlexamples.md) with access to the [Sclera plugins](../setup/components.md) than the standard SQL.

Alternatively, you can use [Sclera's proprietary API](#sclera-proprietary-api-example) in your applications. This API is much simpler than JDBC, but is not a standard.

### Sclera - JDBC Example

We illustrate the use of the JDBC interface with Sclera using an example application. The code is available on GitHub, both in Java and Scala:

- [Sclera - JDBC Example (Java version) on GitHub](https://github.com/scleradb/sclera-example-java-jdbc)
- [Sclera - JDBC Example (Scala version) on GitHub](https://github.com/scleradb/sclera-example-scala-jdbc)

This example application shows how an application can interface with Sclera using the standard [JDBC API](https://docs.oracle.com/javase/tutorial/jdbc/overview/index.html).

To use Sclera through JDBC, the application needs to:

- specify the Sclera home directory by setting the `SCLERA_ROOT` environment variable (if not set, the default is `$HOME/.sclera`)
- add the following dependencies:
    - Sclera Configuration Manager, [sclera-config](https://github.com/scleradb/sclera/tree/master/modules/config),
    - Sclera Core Engine, [sclera-core](https://github.com/scleradb/sclera/tree/master/modules/core),
    - Sclera JDBC Driver, [sclera-jdbc](https://github.com/scleradb/sclera/tree/master/modules/interfaces/jdbc), and
    - Sclera plugins needed (if any).
- connect to Sclera's JDBC driver using the JDBC URL `jdbc:scleradb`, and execute commands and queries using the standard [JDBC API](https://docs.oracle.com/javase/tutorial/jdbc/overview/index.html).

The example application described below is a command line tool to initialize Sclera, and execute queries. See [here](#executable-script) for details on the usage.

#### Specify Sclera Root Directory

We need to specify a directory where Sclera can keep its configuration, metadata, and internal database. This is done by setting the environment variable `SCLERA_ROOT`. If not specified, the default is `$HOME/.sclera`.

#### Add Package Dependencies

This example uses [SBT](https://scala-sbt.org) as the build tool, and the build file is `build.sbt`.

The required dependencies are added as:

```scala
libraryDependencies ++= Seq(
    "com.scleradb" %% "sclera-config" % "4.0",
    "com.scleradb" %% "sclera-core" % "4.0",
    "com.scleradb" %% "sclera-jdbc" % "4.0"
)
```

This is a minimal example, and does not include any Sclera plugins. If your example needs a Sclera Plugin, it should be added to the `libraryDependencies` as well.

The latest versions of all Sclera components are listed at [https://github.com/scleradb/sclera-version-map/blob/master/versions.ini](https://github.com/scleradb/sclera-version-map/blob/master/versions.ini).

#### Interface with Sclera using the JDBC API

This application consists of a single source file, [`JdbcExample.java`](https://github.com/scleradb/sclera-example-java-jdbc/blob/master/src/main/java/com/example/sclera/jdbc/JdbcExample.java) / [`JdbcExample.scala`](https://github.com/scleradb/sclera-example-scala-jdbc/blob/master/src/main/scala/JdbcExample.scala).

There are two procedures:

- `initialize()`: This initializes Sclera's schema (metadata). This is called when `--init` is specified on the command line.
- `runQueries()`: This executes queries provided on the command line and displays the results.

***Code Details: `initialize()`***

- Links with Sclera's JDBC driver and gets a JDBC `Connection`.
- Creates a JDBC `Statement` using the JDBC connection.
- Executes the statement `create schema` on Sclera using the JDBC `Statement`.

When a connection is initialized, Sclera first checks the sanity of its Schema and issues a warning if anything is wrong. Since we are initializing the schema, we bypass this step by passing a flag `checkSchema` in the properties while creating a connection.

***Code Details: `runQueries(...)`***

- Links with Sclera's JDBC driver and gets a JDBC `Connection`.
- Creates a JDBC `Statement` using the JDBC connection.
- For each query in the list passed as the parameter,
    - Executes the query using the JDBC `Statement`, getting the JDBC `ResultSet`
    - Get the JDBC `ResultSetMetadata` for the `ResultSet` -- this provides the number of columns in the result and their names.
    - Output the column names, followed by the result values one row at a time.

### Sclera - Proprietary API Example

We illustrate the use of [Sclera's proprietary API](https://www.javadoc.io/doc/com.scleradb/sclera-core_2.13/latest/com/scleradb/exec/Processor.html) using the same example application as above. The code is available on GitHub:

- [Sclera - Proprietary API Example on GitHub](https://github.com/scleradb/sclera-example-scala-api)

To use Sclera through the proprietary API, the application needs to:

- specify the Sclera home directory by setting the `SCLERA_ROOT` environment variable (if not set, the default is `$HOME/.sclera`)
- add the following dependencies:
    - Sclera Configuration Manager, [sclera-config](https://github.com/scleradb/sclera/tree/master/modules/config),
    - Sclera Core Engine, [sclera-core](https://github.com/scleradb/sclera/tree/master/modules/core),
    - Sclera plugins needed (if any).

The example application described below is a command line tool to initialize Sclera, and execute queries. See [here](#executable-script) for details on the usage.

#### Specify Sclera Root Directory

We need to specify a directory where Sclera can keep its configuration, metadata, and internal database. This is done by setting the environment variable `SCLERA_ROOT`. If not specified, the default is `$HOME/.sclera`.

#### Add Package Dependencies

This example uses [SBT](https://scala-sbt.org) as the build tool, and the build file is [`build.sbt`](https://github.com/scleradb/sclera-example-scala-api/blob/master/build.sbt).

The required dependencies are added as:

```scala
libraryDependencies ++= Seq(
    "com.scleradb" %% "sclera-config" % "4.0",
    "com.scleradb" %% "sclera-core" % "4.0"
)
```

This is a minimal example, and does not include any Sclera plugins. If your example needs a Sclera Plugin, it should be added to the `libraryDependencies` as well.

#### Interface with Sclera using the Proprietary API

This application consists of a single source file, [`ApiExample.scala`](https://github.com/scleradb/sclera-example-scala-api/blob/master/src/main/scala/ApiExample.scala).

There are two procedures:

- `initialize()`: This initializes Sclera's schema (metadata). This is called when `--init` is specified on the command line.
- `runQueries()`: This executes queries provided on the command line and displays the results.

***Code Details: `initialize()`***

- Creates and initializes an instance of Sclera [`Processor`](https://www.javadoc.io/doc/com.scleradb/sclera-core_2.13/latest/com/scleradb/exec/Processor.html)
- Executes the statement `create schema` on Sclera using the `Processor` instance.

When the `Processor` instance is initialized, Sclera first checks the sanity of its Schema and issues a warning if anything is wrong. Since we are initializing the schema, we bypass this step by passing a flag `checkSchema` in the properties while creating the `Processor` instance.

***Code Details: `runQueries(...)`***

- Creates and initializes an instance of Sclera [`Processor`](https://www.javadoc.io/doc/com.scleradb/sclera-core_2.13/latest/com/scleradb/exec/Processor.html)
- For each query in the list passed as the parameter,
    - Executes the query using the `Processor` instance, getting the result as an instance of type [`TableResult`](https://www.javadoc.io/doc/com.scleradb/sclera-core_2.13/latest/com/scleradb/sql/result/TableResult.html), containing an iterator over the returned rows and the metadata for the row columns. The rows are of type [`TableRow`](https://www.javadoc.io/doc/com.scleradb/sclera-core_2.13/latest/com/scleradb/sql/result/TableRow.html), and the metadata is a list of instances of type [`Column`](https://www.javadoc.io/doc/com.scleradb/sclera-core_2.13/latest/com/scleradb/sql/datatypes/Column.html).
    - Output the column names, followed by the result values one row at a time.

### Executable Script

The build file in each of the above cases contains a task `mkscript` that generates an executable script for the application, called `scleraexample` in the `bin` subdirectory. You can generate the script using the command:

    > sbt mkscript

The script is run as follows:

    > bin/scleraexample --init

    > bin/scleraexample "select 'Hello' as greeting1, 'World!' as greeting2"
    GREETING1, GREETING2
    Hello, World!
