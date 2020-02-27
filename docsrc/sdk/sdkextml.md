In this document, we show how to build custom connectors to any machine learning library. These connectors enable Sclera to include machine learning objects (such as classifiers) as first-class SQL objects (at par with [Tables](../sclerasql/sqlregular.md#base-tables) and [Views](../sclerasql/sqlregular.md#views), for instance), and also include machine learning tasks (such as classification) as relational operators within SQL.

This is achieved by mapping the training and execution to the interfaced library's API, transforming the input and the output, and converting the result to a relational stream for consumption of other SQL operators.

[Sclera - Weka Connector](../setup/components.md#sclera-weka) is built using this SDK. For examples of how these connectors are used in Sclera, please refer to the [documentation on using machine learning in SQL](../sclerasql/sqlextml.md).

## Building Machine Learning Library Connectors

To build a custom datasource connector, you need to provide implementations of the following abstract classes in the SDK:

- <a class="anchor" name="mlservice"></a> `MLService` ([Scala](http://scleradb.github.io/sclera-core-sdk/index.html#com.scleradb.analytics.ml.service.MLService), [Java](http://scleradb.github.io/sclera-extensions-java-sdk/index.html#com.scleradb.java.analytics.ml.service.DBService))
    - Provides machine learning operators as a service to Sclera, using the specified library.
    - Contains an `id` that identifies this service.
    - Contains the method `createClassifier`, `createClusterer` that is used to create (i.e. train), respectively, a new [Classifier](#classifier) or [Clusterer](#clusterer) for this service.
- <a class="anchor" name="classifier"></a> `Classifier` ([Scala](http://scleradb.github.io/sclera-core-sdk/index.html#com.scleradb.analytics.ml.classifier.objects.Classifier), [Java](http://scleradb.github.io/sclera-extensions-java-sdk/index.html#com.scleradb.java.analytics.ml.classifier.objects.Classifier))
    - Wrapper over classes implementing clustering algorithms. [Training](../sclerasql/sqlextml.md#classifier-training) involves learning a classifier with a designated `targetAttr` using the feature attributes `featureAttrs`, all of which must be present in the input.
    - Provides a function `classifyOpt` that returns the label for a new data point, if one can be assigned by the classifier; this is used by the [`CLASSIFIED WITH` clause](../sclerasql/sqlextml.md#classifier-application).
- <a class="anchor" name="clusterer"></a> `Clusterer` ([Scala](http://scleradb.github.io/sclera-core-sdk/index.html#com.scleradb.analytics.ml.clusterer.objects.Clusterer), [Java](http://scleradb.github.io/sclera-extensions-java-sdk/index.html#com.scleradb.java.analytics.ml.clusterer.objects.Clusterer))
    - Wrapper over classes implementing clustering algorithms. [Training](../sclerasql/sqlextml.md#clusterer-training) involves clustering the given data.
    - Provides a function `cluster` that assigns a cluster id to a new data point; this is used by the [`CLUSTERED WITH` clause](../sclerasql/sqlextml.md#clusterer-application).

The [Sclera - Weka Connector](../setup/components.md#sclera-weka), included with the Sclera platform, is open source and implements the interface mentioned above. The code for the [Sclera - Weka Connector](../setup/components.md#sclera-weka), in Scala, also appears as an illustrative example in the [Sclera Extensions (Scala) Github repository](https://github.com/scleradb/sclera-extensions-scala).
 
## Packaging and Deploying the Connector

The included [Sclera - Weka Connector](../setup/components.md#sclera-mysql) implementation uses [sbt](http://www.scala-sbt.org) for building the connector [(installation details)](http://www.scala-sbt.org/release/docs/Getting-Started/Setup.html#installing-sbt). This is not a requirement -- any other build tool can be used instead.

### Dependencies

For Scala:

- The Scala implementation has a dependency on the [`"sclera-core"` library](../sdk/sdkintro.md#scalasdk). This library is available from the [Sclera repository](http://scleradb.releases.s3.amazonaws.com); see the included [sbt build file](https://github.com/scleradb/sclera-extensions-scala/blob/master/sclera-weka/build.sbt) for the details. Note that the dependency is annotated `"provided"` since the `jar` for `"sclera-core"` will be available in the `CLASSPATH` when this connector is run with Sclera.

For Java:

- The Java implementation has a dependency on the [`"sclera-core"` library](../sdk/sdkintro.md#scalasdk), as wells as on the [`"sclera-extensions-java-sdk"` library](#javasdk). These libraries are available from the [Sclera repository](http://scleradb.releases.s3.amazonaws.com). Note that the dependency on `"sclera-core"` is annotated `"provided"` since the `jar` for `"sclera-core"` will be available in the `CLASSPATH` when this connector is run with Sclera.

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

**Note:** Please ensure that the identifier you assign to the connector is unique, that is - does not conflict with the identifier of any other available [`MLService`](#mlservice) instance.
