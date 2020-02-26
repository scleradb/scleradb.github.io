Installing Sclera involves the following simple steps:

+ [Download installer](#download-installer)
+ [Run Installer](#run-installer)
+ [Install licensed components](#install-licensed-components)

**System Requirements:** Sclera only works with Linux or Mac OS X running [Java SE 7](http://www.oracle.com/technetwork/java/javase/downloads/index.html). You need to have a working internet connection for the duration of the installation.

*We recommend against installing Sclera with root/admin permissions. Sclera does not need root access for installation or at runtime.*

## Download Installer
The installer is available at scleradb.com ([direct link](/download)).

Save the installer as `sclera-install.jar`.

## Run Installer
Run the installer using:

    java -jar sclera-install.jar

This will bring up the installer application. Please follow the instructions on screen.

+ The installer asks for your email id. Please enter the email id used to login into your [scleradb.com](http://scleradb.com) account.
+ <a class="anchor" name="sclera-home"></a>The installer also asks for the directory where you want to install Sclera. Please enter the appropriate path. For Sclera to install, you need to have write permissions to the specified path. The directory is created if it does not exist already. In the rest of this document, this path will be referred to as `$SCLERA_HOME`.

The installer proceeds to set up a basic, unlicensed version of Sclera in the specified directory `$SCLERA_HOME`. The structure of the directory is as follows:

    bin/         # executable scripts
    log/         # log files
    assets/
      history    # shell command history 
      data/      # embedded H2 data files
      config/    # configuration files
      licenses/  # license files
      install/   # installation files
      lib/       # downloaded libraries
    Uninstaller/ # uninstaller

The installation involves downloading core sclera components and associated libraries. This can take a few minutes. You can monitor the progress by viewing the generated logs in `$SCLERA_HOME/log/install.log`.

To check the progress so far, run `$SCLERA_HOME/bin/sclera.sh`. This should start the shell with the message:

    Welcome to Sclera
    Version 2.1.141018 [unlicensed]

    > _

This basic version is initialized with a single location -- an embedded H2 main-memory database. You cannot link your data sources or use the advanced analytics features in this basic version. To enable these features, you need to install your licensed components.

## Install Licensed Components

The next step is installing your licensed components. This can either be done automatically (recommended), or manually. [Automated installation](#automated-installation) installs all the components together, while [manual installation](#manual-installation) enables you to install the components one at a time.

### Automated Installation

The automated installation installs all your licensed components in a single step. Just run the following command:

    $SCLERA_HOME/bin/install-licensed.sh

The script starts by asking for your password -- please enter the password of your scleradb.com account. The script then securely downloads your license file to `$SCLERA_HOME/assets/licenses/SCLERA-LICENSE.txt`.

The downloaded license file includes the details of your active subscriptions. These details are then used to determine your downloads. The script uses this information to install all your licensed components and their dependencies. You can monitor the progress by viewing the generated logs in `$SCLERA_HOME/log/install.log`.

To check the installation, run `$SCLERA_HOME/bin/sclera.sh`. This should start the shell with the message:

    Welcome to Sclera
    Version 2.1.141018 [Licensed to user@example.com]

    > _

This completes the installation. Sclera is now ready for use.

### Manual Installation

Manual installation enables you to install the components one at a time. This makes sense if you do not need all the licensed components immediately -- for instance, in the trial version, you may not want to install [sclera-hbase](../setup/components.md#sclera-hbase) if you do not have Apache HBase installed.

The first step in the manual installation is to download the license. This is done as follows:

    $SCLERA_HOME/bin/license.sh download

You will be prompted for your scleradb.com password, and the license will be downloaded to `$SCLERA_HOME/assets/licenses/SCLERA-LICENSE.txt`. (You can alternatively download the license file through your browser from [www.scleradb.com/license](http://www.scleradb.com/license)).

You can now install the component `sclera-xxx` as follows:

    $SCLERA_HOME/bin/install.sh sclera-xxx

In the command above, `sclera-xxx` is the name of the component being installed. You will need run this command for each [licensed component](/subscriptions) you want to install. While the command is running, you can monitor the progress by viewing the generated logs in `$SCLERA_HOME/log/install.log`.

The following command lists your licensed components, along with their versions:

    $SCLERA_HOME/bin/license.sh products

## Update Installation

To reactivate after subscription renewal, just download the license again:

    $SCLERA_HOME/bin/license.sh download

To upgrade the licensed components, or to install additional components from an updated subscription, follow the steps outlined for [installing licensed components](#install-licensed-components) above (you can choose between [automated installation](#automated-installation) or [manual installation](#manual-installation)).
