In this document, we show how to build custom connectors to any NLP / text analytics library to perform text analytics tasks. These connectors handle the invocation of the underlying library to process text data in table columns, in the query processing pipeline.

The component [Sclera - Apache OpenNLP Connector](/doc/ref/components#sclera-opennlp) is built using this SDK. For examples of how the connector is used in Sclera, please refer to the [documentation on using text analytics in SQL](/doc/ref/sqlexttext).

## Building Text Analytics Library Connectors

To build a custom datasource connector, you need to provide implementations of the following abstract classes in the SDK:

- <a class="anchor" name="nlpservice"></a> `NlpService` ([Scala](http://scleradb.github.io/sclera-core-sdk/index.html#com.scleradb.analytics.nlp.service.MLService), [Java](http://scleradb.github.io/sclera-extensions-java-sdk/index.html#com.scleradb.java.analytics.nlp.service.DBService))
    - Provides text analytics operators as a service to Sclera, using the specified library.
    - Contains an `id` that identifies this service.
    - Contains the method `createObject` that is used to create a new task object for the task named in the parameter `taskName` for this service.
- <a class="anchor" name="nlptask"></a> `NlpTask` ([Scala](http://scleradb.github.io/sclera-core-sdk/index.html#com.scleradb.analytics.nlp.objects.NlpTask), [Java](http://scleradb.github.io/sclera-extensions-java-sdk/index.html#com.scleradb.java.analytics.nlp.objects.NlpTask))
    - Wrapper over classes implementing text analytics algorithms.
    - Provides a function `eval` that takes a data stream (an iterator over rows, with associated metadata) as input and returns the same data stream, with each row augmented by columns `resultCols` containing the output of executing the task `taskName` on the text in column `inputCol`. If the evaluation on a row emits multiple evaluation results, the input row is repeated in the output for each such result.

The [Sclera - Apache OpenNLP Connector](/doc/ref/components#sclera-opennlp), included with the Sclera platform, is open source and implements the interface mentioned above. The code for the [Sclera - Apache OpenNLP Connector](/doc/ref/components#sclera-opennlp), in Scala, also appears as an illustrative example in the [Sclera Extensions (Scala) Github repository](https://github.com/scleradb/sclera-extensions-scala).
 
## Packaging and Deploying the Connector

The included [Sclera - Apache OpenNLP Connector](/doc/ref/components#sclera-mysql) implementation uses [sbt](http://www.scala-sbt.org) for building the connector [(installation details)](http://www.scala-sbt.org/release/docs/Getting-Started/Setup.html#installing-sbt). This is not a requirement -- any other build tool can be used instead.

### Dependencies

For Scala:

- The Scala implementation has a dependency on the [`"sclera-core"` library](/doc/sdk/sdkintro#scalasdk). This library is available from the [Sclera repository](http://scleradb.releases.s3.amazonaws.com); see the included [sbt build file](https://github.com/scleradb/sclera-extensions-scala/blob/master/sclera-opennlp/build.sbt) for the details. Note that the dependency is annotated `"provided"` since the `jar` for `"sclera-core"` will be available in the `CLASSPATH` when this connector is run with Sclera.

For Java:

- The Java implementation has a dependency on the [`"sclera-core"` library](/doc/sdk/sdkintro#scalasdk), as wells as on the [`"sclera-extensions-java-sdk"` library](#javasdk). These libraries are available from the [Sclera repository](http://scleradb.releases.s3.amazonaws.com). Note that the dependency on `"sclera-core"` is annotated `"provided"` since the `jar` for `"sclera-core"` will be available in the `CLASSPATH` when this connector is run with Sclera.

### Deployment Steps

The connector can be deployed using the following steps:

#### Alternative 1 (Recommended)
- First, publish the implementation as a local package. In sbt, this is done by running `sbt publish-local`.
- Run the following to install the component and its dependencies:

    <pre><code>> $SCLERA_HOME/bin/install.sh package_name package_version package_org</code></pre>

    - `$SCLERA_HOME` is the directory where Sclera is installed
    - `package_name` is the name of the package being installed
    - `package_version` is the version of the package being installed
    - `package_org` is the org of the package being installed
- Run the following to include the path to the installed component package jar and the dependencies in the `CLASSPATH`:

    <pre><code>> . $SCLERA_HOME/assets/install/classpath/package_name-classpath.sh</code></pre>

    - The bash script `package_name-classpath.sh` was automatically created in the previous step, while installing the package. The `package_name` part of the file name is the name of the package installed.

The connector should now be visible to Sclera, and can be used in the queries.

#### Alternative 2
- First, compile and package the implementation into a `jar` file. In sbt, this is done by running `sbt package`. If you have external dependencies, you may need to assemble everything into a single `jar` -- in `sbt`, this can be done using the [`sbt-assembly` plugin](https://github.com/sbt/sbt-assembly).
- Next, add the path to the generated `jar` file to the `CLASSPATH` environment variable.

The connector should now be visible to Sclera, and can be used in the queries.

**Note:** Please ensure that the identifier you assign to the connector is unique, that is - does not conflict with the identifier of any other available [`NlpService`](#nlpservice) instance.
