## What is Sclera?

Sclera is a stand-alone SQL processor with native support for machine learning, data virtualization and streaming data. Sclera can be deployed as an independent application, or can embed within your applications to enable advanced real-time analytics capabilities.

## I am a BI professional. Why do I need Sclera?

As a BI professional, you are conversant with SQL, and understand the need to move to advanced analytics. But so far, exploring advanced analytics has meant sparing valuable time learning the myriad APIs, and getting down to coding in Java, R, Python, or whatever it takes.

For you, Sclera is the most efficient way to build analytics applications. You only need to learn a handful of SQL extensions to start exploiting the power of sophisticated libraries such as Weka and Apache Mahout, incorporate external data from web-services, and perform complex event processing and stream analytics, and more -- all using familiar SQL. Moreover, it works on your existing database systems, with no need for any hardware setup and no need to move your data.

## I am an analytics consultant. Why do I need Sclera?

As an analytics consultant, you develop analytics solutions for your clients.

You understand that a project is never fully specified. So, it is crucial for your solutions to be agile, and be able to incorporate incremental changes as quickly as possible.

Sclera gives you a modular architecture out of the box, providing exceptional agility to your solutions.

Specifically, Sclera separates the analytics logic from the processing and data access. The analytics logic is specified declaratively as SQL queries with Sclera's analytics extensions. This is just a few lines of code, which can be changed easily. The analytics libraries, database systems and external data sources form their own modules and are separated from the analytics logic. The analytics queries are compiled by Sclera into optimized workflows that dynamically tie everything together.

Sclera thus provides a highly modular and customizable end-to-end stack for analytics, and enables you to experiment and iterate at the speed of thought.

Even after deployment of the solution, Sclera's modular architecture significantly reduces the maintenance complexity and simplifies upgrades. For instance, new technologies (database systems, analytics libraries) can be incorporated by just adding appropriate drivers, with minimal change to the application.

## I am a data scientist. Why do I need Sclera?

As a data scientist, you are an expert in machine learning. You know the underlying mathematics, and your task is to develop algorithms to gain insights.

However, to get access and experiment on real-life data, you need to get into implementation. You need to combine data from multiple sources. Occasionally, you find that you need to understand nontrivial APIs to figure how to perform your computations -- and your bookshelf is full of "In Action" and "Definitive Guides" that you need time to read. You spend more time poring over system logs and complex undocumented open-source code than working on your algorithms. This makes experimentation hard, distracts and slows you down.

Sclera gives you a standardized interface to run your algorithms within SQL, and a clean API (in the Sclera's Extensions SDK) that you can use to integrate your algorithms, in just a few lines of code. You can now quickly experiment, iterate and thus focus on your analytics tasks, while Sclera takes care of the legwork.

## What is the productivity impact of using Sclera?

Sclera simplifies the path from idea to insights. Using Sclera saves the need for deep systems skills, months of resources, and hundreds of lines of code in building analytics applications, the accompanying test and maintenance, and so on.

This simplicity enables you to quickly experiment and iterate over alternatives, and focus on your analytics tasks without bothering about the complex implementation details.

## What is the performance impact of using Sclera?

Sclera's SQL engine has three components: the query compiler, the embedded streaming SQL processor, and the embedded analytics evaluator.

- The query compiler compiles the input query into a plan -- this happens once per query, before the evaluation, and the compilation time is negligible as compared to the evaluation time. If the entire query gets pushed to an underlying database system, the cost is thus effectively zero.

- The embedded streaming SQL processor is used to evaluate SQL (relational) operators on streaming data, or on intermediate results to avoid materialization. This evaluation proceeds in a pipeline, in a single pass, with minimal memory overheads, and in the same JVM as your application.

- The embedded analytics evaluator, likewise, proceeds in a pipeline, in a single pass, and in the same JVM as your application. A handcoded Java program would have identical overheads when it uses the same library.

Sclera also includes a query optimizer that optimizes the workflow before each run. These optimizations can potentially speed up the evaluation in ways that a handcoded Java application cannot.

*We are working on performance benchmarks, and will share the results soon.*

## How fast is Sclera?

Sclera aggressively pushes down query computations to the underlying database systems, and uses external analytics libraries for analytics evaluation where needed. Thus, the performance of Sclera is thus determined by the performance of the underlying database systems and analytics libraries.

This means that you can start with your existing infrastructure, identify the bottlenecks, and add more resources intelligently as and when needed -- all without modifying your applications.

## How is Sclera different from a database system?

To the user, Sclera is just like a relational database system -- with SQL as the interface language, and JDBC as the access mechanism from the application programs. However:

- Sclera does not store data. It works on data from the connected database system and/or external data sources (on-disk file, web-service, etc.) specified in your query. Sclera queries can work on data across multiple database systems and external data sources.
- Sclera natively supports analytics. Analytics operations (such as classification) are provided as SQL language extensions, and analytics objects (such as classifiers) as first class-objects, at par with SQL tables. 
- Sclera includes an embedded SQL processor, but also pushes SQL computation to an underlying database systems wherever possible. Sclera's optimizer understands the capabilities of the underlying database systems and the data stored therein, and intelligently decides where to locate the computations.

## Does Sclera replace my database system?

No, Sclera complements your database systems. Sclera works with your database systems, and extends their capability to perform advanced analytics.

## Is Sclera another SQL-on-Hadoop system?

Sclera is a "SQL-on-Anything" system.

The [Sclera - Apache HBase Connector](/doc/ref/components#sclera-hbase) enables Sclera to run SQL on Apache HBase (which sits over Hadoop).

If you need to work on files stored in HDFS, you can build a custom connector using the [Hadoop FileSystem API](https://hadoop.apache.org/docs/current/api/index.html?org/apache/hadoop/fs/FileSystem.html) and the [Sclera Extensions SDK](/doc/sdk/sdkextdataaccess). Sclera can provide a pre-built extension on request  -- please send a mail to support@scleradb.com.

## How does Sclera compare with R?

[R](http://www.r-project.org) is a powerful programming language. The main advantage R brings over other languages is its extensive set of modules, developed over the years by statisticians, and its excellent charting capabilities.

At the same time, since R is a low-level programming language, working with R needs programming skills. Especially when you need to move beyond using the provided modules in R. You also need to be conversant with dataframes, factors and such -- which are unique to the statistician's world view. Also, R's execution engine was not designed handle large volumes of data.

Further, it is hard to efficiently integrate R with the rest of your eco-system. Working with R thus needs an independent setup -- this adds to the number of processing and data silos.

Sclera does not claim to provide the functionality in the hundreds of R modules -- but its own set of analytics extensions should provide most of the capabilities you need, and are well-integrated with your existing ecosystem.

As an alternative, the [Sclera Extensions SDK](/doc/sdk/sdkextdataaccess) can be used to call R from with SQL, using the [Java/R Interface - JRI](http://rforge.net/JRI/).

<a class="anchor" name="scleraviz"></a>
## How does Sclera Visualization compare with ggplot2 and D3?

[ggplot2](http://ggplot2.org) is a graphics library for [R](http://www.r-project.org). `ggplot2` is inspired by the [Grammar of Graphics](http://vita.had.co.nz/papers/layered-grammar.html), which makes it very expressible and powerful. However, `ggplot2` can only be used to generate static graphs -- this means no interactivity and no support for streaming data. Also, since it is a part of the R ecosystem, you need to be a proficient R programmer and will need to go through some hoops to get it working with the rest of your ecosystem.

[D3](http://d3js.org) is a brilliant Javascript library for creating dynamic, interactive graphs in your browser. To use D3, you need to be a proficient Javascript programmer. Support for streaming data has to be built from ground-up in Javascript as well. Furthermore, D3 is confined to the web-browser, so all the needed data transformations needed for visualization happen within the browser, which is computationally expensive for devices such as mobile phones.

[Sclera Visualization (ScleraViz)](/scleraviz), like `ggplot2`, is also inspired by the [Grammar of Graphics](http://vita.had.co.nz/papers/layered-grammar.html). ScleraViz is implemented as an extension to [ScleraSQL](/doc/ref/sqlintro) and uses [D3](http://d3js.org) as the rendering engine.

ScleraViz brings the expressibility of `ggplot2` and the power of D3 to SQL users. Unlike `ggplot2`, ScleraViz can clean, analyze and plot streaming data. Also, unlike visualization implemented in D3, Sclera pushes expensive computations to the backend database servers, keeping the rendering lean and efficient.

## Can Sclera work with my database in the cloud?

Yes. Sclera works with any database system that can be accessed with an API. You just need a connector to interface with the system.

[Google Cloud SQL](https://cloud.google.com/sql/) is compatible with MySQL, and [Amazon RDS](http://aws.amazon.com/rds/) provides MySQL, PostgreSQL and Oracle instances in the cloud. These database systems can be accessed using the relevant included database connector ([sclera-mysql](/doc/ref/components#sclera-mysql-connector), [sclera-postgresql](/doc/ref/components#sclera-postgresql-connector) or [sclera-oracle](/doc/ref/components#sclera-oracle-connector)), simply by putting the appropriate JDBC URL in the [ADD LOCATION](/doc/ref/dbms#connecting-to-database-systems) statements.

## Can Sclera work with web-services?

Sclera provides a [Sclera Extensions SDK](/doc/sdk/sdkextdataaccess) that enables ingestion of data from of any data source into Sclera. A specific connector for the [Google Finance web-service](/doc/ref/components#stock-ticker) is included as an [illustrative example](/doc/sdk/sdkextdataaccess#example). The code for the same can be accessed at [GitHub](https://github.com/scleradb).

## Can Sclera work with my legacy data store?

The [Sclera Extensions SDK for external data access](/doc/sdk/sdkextdataaccess) or the [Sclera Extension SDK for database systems](/doc/sdk/sdkdbms) can be used to build custom connectors to your legacy data store. The former is used when you just want to source data from the data sore, and the latter when you want to push computation (such as filter, join) to the data store.

You can build the connector yourself using the [Sclera Extensions SDK](/doc/sdk/sdkextdbms), or request us by sending a mail to support@scleradb.com with the details.

## Can Sclera work with my reporting software?

Sclera understands a large subset of PostgreSQL's dialect of SQL. Therefore, Sclera should work with any reporting tool that works with PostgreSQL.

In such tools, Sclera's JDBC driver, downloaded as a part of the installation, can be used as a drop-in replacement for PostgreSQL JDBC driver. The details on the usage of the driver appear in the [Sclera JDBC reference](/doc/ref/jdbc) document.

Alternatively, Sclera can work in a server mode that implements the [PostgreSQL backend protocol](http://www.postgresql.org/docs/9.4/static/protocol-overview.html) 3.0, which is compatible with PostgreSQL 7.4+.

Though this server, Sclera can interface with the latest [PostgreSQL ODBC](https://odbc.postgresql.org/) and [JDBC](https://jdbc.postgresql.org/) drivers, [PostgreSQL’s shell (`psql`)](http://www.postgresql.org/docs/9.4/static/app-psql.html), and anything else that uses [PostgreSQL’s native protocol (libpq)](http://www.postgresql.org/docs/9.4/static/libpq.html).

## How do I use Sclera's analytics operators in my reporting software?

Connected to Sclera, your reporting software can be used to query data across multiple underlying data sources. However, since these tools do not understand Sclera's extensions, they cannot generate queries with embedded analytics.

A simple workaround is to [create views](/doc/ref/sqlregular#views) in Sclera -- the view definition can contain arbitrary Sclera extensions, but to the external tool, they are equivalent to a relational table. Any query on such a view generated by the tool will evaluate the analytics operators included in the view definition.

## Can Sclera work with an analytics library of my choice?

Yes, but the support is currenly limited to classification, clustering and association rule mining. You will need to map the classification, clustering and/or association rule API in the [Sclera Extensions SDK](/doc/sdk/sdkextml) to your library's API.

The code for the [Sclera - Weka Connector](/doc/ref/components#sclera-weka) appears as an illustrative example on [GitHub](https://github.com/scleradb/sclera-extensions-scala).

## How do I ingest data streams into Sclera?

Sclera provides a very simple API though the [Sclera Extensions SDK](/doc/sdk/sdkextdataaccess). A connector built using this API can be used to ingest data streams -- these data streams can then be used in the `FROM` clause of SQL queries.

## What do I need to know before using Sclera?

If you know SQL, you can start firing cross-system queries rightaway. Then, incorporate analytics using the analytics extensions we have baked into SQL.

Sclera enables use of advanced analytics constructs such as classification and clustering within SQL, but assumes you know what they are useful for. For the background, we recommend reading up <a href="http://www.data-science-for-biz.com" target="_blank">a good book on data science</a>.

## How do I get support?

We are always around for help. Send a mail to support@scleradb.com, or leave a voice message at +1-909-979-9796 - we will respond within 24 hours.
