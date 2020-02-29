Sclera is a stand-alone SQL processor with native support for machine learning, data virtualization and streaming data. Sclera can be deployed as:

- [an independent application with an interactive command-line shell](#installing-and-maintaining-sclera-command-line-application), or
- [as a library that embeds within your applications to enable advanced real-time analytics capabilities]().

**Prerequisite:** Sclera requires [Java version 8 or higher](https://java.com/en/download/help/download_options.xml).

*We recommend against installing Sclera with root/admin permissions. Sclera does not need root access for installation or at runtime.*

## Installing and Maintaining Sclera Command Line Application

The recommended way to install Sclera's interactive command line shell is through Sclera platform administration tool, `scleradmin`.

We first describe how to [install `scleradmin`](#installing-scleradmin). Next, we use `scleradmin` to [install Sclera's core packages and command line interface](#installing-sclera-core-packages-and-shell). We then show how to [add and remove optional Sclera plugins](#plugin-management). Finally, we show how to [update the installed core packages and plugins](#updating-installed-packages-and-plugins).

**Prerequisites:** [Python 3.8+](https://python.org/downloads) and [pip](https://pip.pypa.io/en/stable/installing/).

### Installing scleradmin

The following installs the latest version of `scleradmin` as a Python package in your system:

    > pip install scleradmin

In the following sections, we show how to use `scleradmin` for installing Sclera and managing the installation. For full details on `scleradmin` usage, you may want to read the help information:

    > scleradmin --help

### Installing Sclera Core Packages and Shell

The following command installs Sclera:

    > scleradmin --install --root <sclera-root>

Please substitute your location of choice for `<sclera-root>`. This directory must not exist before installation, it is created by the command. (This is enforced to avoid accidental overwrites.)

The installation involves downloading core sclera components and associated libraries. This might take a few minutes; you can monitor the progress by viewing the generated logs in `<sclera-root>/install/log/install.log`. The contents of the directory after installation are described later in this document.

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

In the above, `<plugins>` is a space-separated list of plugins to be added, and `<sclera-root>`, as earlier, is the root directory. For instalce, to install the Sclera - CSV Connector and Sclera - Text Files Connector plugins, to the Sclera instance installed at `/path/to/sclera` the command is: 

    > scleradmin --add sclera-csv-plugin sclera-textfiles-plugin --root /path/to/sclera

To remove installed plugins, the syntax is similar. The following command removes the plugins installed above:

    > scleradmin --remove sclera-csv-plugin sclera-textfiles-plugin --root /path/to/sclera

You can specify a list of plugins to add and another list of plugins to remove in the same command.

### Updating Installed Packages and Plugins

The following command updates Sclera's core packages as well as the plugins to the latest version:

    > scleradmin --update
