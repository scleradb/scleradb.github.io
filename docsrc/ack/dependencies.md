In this document we list out the third party dependencies for the Sclera components, and the installer.

These dependencies are not distributed with Sclera, but are [downloaded directly from a repository](https://www.scala-sbt.org/1.x/docs/Sbt-Launcher.html) during installation of the respective component.

## Sclera Component Dependencies
The following table lists the direct third-party library dependencies for each Sclera component, along with the dependency's license. Clicking on the mentioned license name will take you to the license statement. For a more complete list, please see the file NOTICE.txt in the source of the respective components.

| Component | Dependency | Dependency License |
| --------- | ---------- | ------------------ |
| [Sclera - Core Engine](../setup/components.md#sclera-core) | [H2 Database](http://www.h2database.com) | [H2 License version 1.0](http://www.h2database.com/html/license.html#h2_license) |
| | [Logback](http://logback.qos.ch) | [Eclipse Public License version 1.0](http://logback.qos.ch/license.html) |
| | [HikariCP](http://brettwooldridge.github.io/HikariCP) | [Apache License version 2.0](https://raw.githubusercontent.com/brettwooldridge/HikariCP/dev/LICENSE) |
| [Sclera - Configuration Manager](../setup/components.md#sclera-config) | [Typesafe Config](https://github.com/typesafehub/config) | [Apache License version 2.0](http://www.apache.org/licenses/LICENSE-2.0) |
| | [Logback](http://logback.qos.ch) | [Eclipse Public License version 1.0](http://logback.qos.ch/license.html) |
| [Sclera - Command Line Shell](../setup/components.md#sclera-shell) | [JLine3](https://jitpack.io/p/jline/jline3) | [BSD 3-Clause License](https://opensource.org/licenses/BSD-3-Clause) |
| | [Jansi](https://fusesource.github.io/jansi/) | [Apache License v2.0](http://www.apache.org/licenses/LICENSE-2.0) |
| | [Logback](http://logback.qos.ch) | [Eclipse Public License version 1.0](http://logback.qos.ch/license.html) |
| [Sclera - Oracle Connector](../setup/components.md#sclera-oracle) | [Oracle JDBC Driver](http://www.oracle.com/technetwork/database/features/jdbc/jdbc-drivers-12c-download-1958347.html) | [OTN License Agreement](http://www.oracle.com/technetwork/licenses/distribution-license-152002.html) |
| [Sclera - MySQL Connector](../setup/components.md#sclera-mysql) | [MySQL Connector/J](http://dev.mysql.com/doc/connector-j/en/index.html) | [GNU GPL v2 with FOSS Exception](https://oss.oracle.com/licenses/universal-foss-exception/) |
| [Sclera - PostgreSQL Connector](../setup/components.md#sclera-postgresql) | [PostgreSQL JDBC Driver](http://jdbc.postgresql.org) | [BSD License](http://jdbc.postgresql.org/about/license.html) |
| [Sclera - CSV File Connector](../setup/components.md#sclera-csv) | [Apache Commons CSV](http://commons.apache.org/proper/commons-csv/) | [Apache License version 2.0](http://www.apache.org/licenses/LICENSE-2.0) |
| [Sclera - Apache OpenNLP Connector](../setup/components.md#sclera-opennlp) | [Apache OpenNLP version 1.5.3](http://opennlp.apache.org) | [Apache License version 2.0](http://www.apache.org/licenses/LICENSE-2.0) |
| [Sclera - Weka Connector](../setup/components.md#sclera-weka) | [Weka](http://www.cs.waikato.ac.nz/ml/weka) | [GNU General Public License version 2](http://www.gnu.org/licenses/gpl.html) |
| [Sclera - Web Display](../setup/components.md#sclera-visualization) | [D3](http://d3js.org) | [BSD 3-Clause License](https://opensource.org/licenses/BSD-3-Clause) |
| | [D3 SVG Legend](http://d3-legend.susielu.com) | [D3-Legend License](https://raw.githubusercontent.com/susielu/d3-legend/master/LICENSE) |
| | [Javalin](https://javalin.io) | [Apache License v2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| | [Logback](http://logback.qos.ch) | [Eclipse Public License version 1.0](http://logback.qos.ch/license.html) |

## Installer Dependencies
The following table lists the dependencies for the [Sclera installer](../setup/install.md). Clicking on the mentioned license name will take you to the license statement.

| Dependency | Dependency License |
| ---------- | ------------------ |
| [SBT Launcher Interface](http://www.scala-sbt.org) | [Apache License 2.0](https://github.com/sbt/sbt-launcher-package/blob/master/LICENSE) |
