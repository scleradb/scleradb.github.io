Designed with business intelligence professionals in mind, Sclera directly addresses the challenges an IT organization faces while implementing analytics applications.

This means that you can now focus on generating insights from your data, and leave the rest to Sclera.

## Problem: Lack of time to develop systems/programming skills

Moving to analytics requires developing a whole new set of skills -- learning new languages, new library APIs, and new architectures. Expertise is rare, and needs time to develop.

**Sclera's solution**
Sclera supports a large subset of standard SQL. So, if you know SQL, you can start firing cross-system queries rightaway.

Later, when you need, you can start including complex analytics in your SQL queries. You do not need to know the complex APIs of the underlying systems and the analytics libraries. Sclera exposes analytics operations as straightforward SQL extensions, which are translated to appropriate API calls at runtime.

As such, in the spirit of SQL, you only need to understand the analytics concepts -- not the implementation details of a specific system.

## Problem: Analytics is disruptive, expensive to setup

Moving to analytics requires setting up new systems, allocating resources and setting up processes for ETL. This is not only disruptive, but also requires an upfront investment. 

**Sclera's solution**
Sclera overlays over your existing infrastructure. You do not have to allocate additional resources.

Sclera does not require you to move data or preprocess it (ETL). You just need to link Sclera to the data sources/database systems and Sclera takes care of executing the query across these sources/systems.

You can update the infrastructure, add new systems later, if needed. Sclera will adapt to the changes with minimal disruption.

## Problem: Data is spread across silos, hard to consolidate

Enterprise data is typically spread across multiple data sources. Consolidating this data for analysis is a major challenge. Typically, this implies physically moving data from each source to a centralized analytics platform (through a time and resource-intensive, multi-stage ETL process).

These ETL processes are a processing bottleneck, and hinder the development of real-time analytics solutions.

**Sclera's solution**
Sclera overlays over the data silos and presents them as an unified environment.

Currently, Sclera provides support for Oracle, PostgreSQL, MySQL and HBase as well as read-only data sources such as web-services, flat files (plain text, CSV) through optional extensions. Support for additional sources is coming soon.

At runtime, Sclera breaks up your queries based on the location of queried data, pushes down the computation to the respective platform and pipes the result to the appropriate analytics libraries; the resulting cross-platform workflow is executed under the hood without the need of ETL and data duplication -- the user just fires the query and gets the result. 

## Problem: Multi-structured data

Not all data is relational. Valuable data exists as unstructured text files (such as customer emails), or semi-structured logs (such as clickstreams, or output of a legacy system). Current systems are unable process this data.

**Sclera's solution**
Sclera comes with an extension that can extract structured data from unstructured text. Moreover, Sclera can read flat files (plain text, CSV) stored on your disk (or on HDFS), and also fetch data from web-services.

*Coming soon:* Support for JSON data, log data

## Problem: Vendor Lock-In

Analytics is a highly experimental activity. It is hard to pin down the exact infrastructure needs for an organization in advance. However, most analytics solutions are not agile. As such, enterprises are afraid of a vendor lock-in.

**Sclera's solution**
Sclera decouples the application from the backend. This means that you can replace the backend libraries based on your needs. In fact, Sclera can help you with vendor selection -- write your application first, and then pick the backend that works best for your needs.

## Problem: Tightly coupled data storage, analytics, and reporting
Analytics solutions available today have a tight coupling of their database system, analytics libraries, and even the reporting (visualization) interface. This is a compromise, since it is unlikely that a product is the best at each of the three.

**Sclera's solution**
Sclera bridges across multiple database systems and multiple analytics libraries; each of these is an independent extension. This enables you to pick the best database system and the best analytics library independently.

Currently, Sclera supports Oracle, MySQL, PostgreSQL and HBase (database systems) and Weka and Mahout (analytics libraries).

Sclera provides a standard JDBC interface, which means that you can use Sclera with the JDBC-compatible reporting/visualization engine of your choice.

## Problem: Lack of extensibility
Each organization's analytics needs are different. A closed system with a specific set of analytics libraries might not be able to capture exactly what you need. For instance, you could have your data in some proprietary format, or need to interface with a legacy system with a proprietary API, or even have a proprietary algorithm to analyze your data.

**Sclera's solution**
Sclera is an open, extensible system. You can develop your own extensions for your proprietary data sources, or add your own analytics routines. These extensions can be called from within SQL queries.
