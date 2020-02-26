In this document we list out the third party dependencies for the Sclera components, and the installer.

These dependencies are not distributed with Sclera, but are [downloaded directly from a repository](https://www.scala-sbt.org/1.x/docs/Sbt-Launcher.html) during installation of the respective component.

## Sclera Component Dependencies
The following table lists the direct third-party library dependencies for each Sclera component, along with the dependency's license. Clicking on the mentioned license name will take you to the license statement.

| Component | Dependency | Dependency License |
| --------- | ---------- | ------------------ |
| [Sclera - Core Engine](../setup/components.md#sclera-core) | [H2 Database](http://www.h2database.com) | [H2 License version 1.0](http://www.h2database.com/html/license.html#h2_license) |
| | [Apache Commons Codec version 1.8](http://commons.apache.org/proper/commons-codec) | [Apache License version 2.0](http://www.apache.org/licenses/LICENSE-2.0) |
| | [Typesafe Config version 1.0.0](https://github.com/typesafehub/config) | [Apache License version 2.0](http://www.apache.org/licenses/LICENSE-2.0) |
| | [Logback version 1.0.13](http://logback.qos.ch) | [Eclipse Public License version 1.0](http://logback.qos.ch/license.html) |
| | [License3j version 1.0.4](http://verhas.github.io/License3j) | [GNU Lesser General Public License version 3](http://verhas.github.io/License3j/license.html) |
| | [HikariCP version 2.4.2](http://brettwooldridge.github.io/HikariCP) | [Apache License version 2.0](https://raw.githubusercontent.com/brettwooldridge/HikariCP/dev/LICENSE) |
| [Sclera - Command Line Shell](../setup/components.md#sclera-shell) | [JLine](http://jline.sourceforge.net) | [BSD License](http://jline.sourceforge.net/license.html) |
| | [Logback version 1.0.13](http://logback.qos.ch) | [Eclipse Public License version 1.0](http://logback.qos.ch/license.html) |
| [Sclera - Oracle Connector](../setup/components.md#sclera-oracle) | [Oracle JDBC Driver](http://www.oracle.com/technetwork/database/features/jdbc/jdbc-drivers-12c-download-1958347.html) | [OTN License Agreement](http://www.oracle.com/technetwork/licenses/distribution-license-152002.html) |
| [Sclera - MySQL Connector](../setup/components.md#sclera-mysql) | [MySQL Connector/J version 5.1.21](http://dev.mysql.com/doc/connector-j/en/index.html) | [GNU General Public License version 2](http://www.gnu.org/licenses/gpl.html) |
| [Sclera - PostgreSQL Connector](../setup/components.md#sclera-postgresql) | [PostgreSQL JDBC Driver](http://jdbc.postgresql.org) | [BSD License](http://jdbc.postgresql.org/about/license.html) |
| [Sclera - CSV File Connector](../setup/components.md#sclera-csv) | [Apache Commons CSV version 1.1](http://commons.apache.org/proper/commons-csv/) | [Apache License version 2.0](http://www.apache.org/licenses/LICENSE-2.0) |
| [Sclera - Apache OpenNLP Connector](../setup/components.md#sclera-opennlp) | [Apache OpenNLP version 1.5.3](http://opennlp.apache.org) | [Apache License version 2.0](http://www.apache.org/licenses/LICENSE-2.0) |
| [Sclera - Weka Connector](../setup/components.md#sclera-weka) | [Weka version 3.7.11](http://www.cs.waikato.ac.nz/ml/weka) | [GNU General Public License version 2](http://www.gnu.org/licenses/gpl.html) |
| [Sclera - Visualization](../setup/components.md#sclera-visualization) | [D3](http://d3js.org) | [D3 License](https://raw.githubusercontent.com/mbostock/d3/master/LICENSE) |
| | [D3 SVG Legend](http://d3-legend.susielu.com) | [D3-Legend License](https://raw.githubusercontent.com/susielu/d3-legend/master/LICENSE) |

## Installer Dependencies
The following table lists the dependencies for the [Sclera installer](../setup/install.md). Clicking on the mentioned license name will take you to the license statement.

| Dependency | Dependency License |
| ---------- | ------------------ |
| [SBT Launcher Interface](http://www.scala-sbt.org) | [Apache License 2.0](https://github.com/sbt/sbt-launcher-package/blob/master/LICENSE) |
