In this document, we show how to build custom connectors to any machine learning library. These connectors enable Sclera to include machine learning objects (such as classifiers) as first-class SQL objects (at par with [Tables](../sclerasql/sqlregular.md#base-tables) and [Views](../sclerasql/sqlregular.md#views), for instance), and also include machine learning tasks (such as classification) as relational operators within SQL.

This is achieved by mapping the training and execution to the interfaced library's API, transforming the input and the output, and converting the result to a relational stream for consumption of other SQL operators.

[Sclera - Weka Connector](../setup/components.md#sclera-weka) is built using this SDK. For examples of how these connectors are used in Sclera, please refer to the [documentation on using machine learning in SQL](../sclerasql/sqlextml.md).

## Building Machine Learning Library Connectors

To build a custom datasource connector, you need to provide implementations of the following abstract classes in the SDK:

- <a class="anchor" name="mlservice"></a> `MLService` ([API Link](/api/sclera-core/com/scleradb/analytics/ml/service/MLService.html))
    - Provides machine learning operators as a service to Sclera, using the specified library.
    - Contains an `id` that identifies this service.
    - Contains the method `createClassifier`, `createClusterer` that is used to create (i.e. train), respectively, a new [Classifier](#classifier) or [Clusterer](#clusterer) for this service.
- <a class="anchor" name="classifier"></a> `Classifier` ([API Link](/api/sclera-core/com/scleradb/analytics/ml/classifier/objects/Classifier.html))
    - Wrapper over classes implementing clustering algorithms. [Training](../sclerasql/sqlextml.md#classifier-training) involves learning a classifier with a designated `targetAttr` using the feature attributes `featureAttrs`, all of which must be present in the input.
    - Provides a function `classifyOpt` that returns the label for a new data point, if one can be assigned by the classifier; this is used by the [`CLASSIFIED WITH` clause](../sclerasql/sqlextml.md#classifier-application).
- <a class="anchor" name="clusterer"></a> `Clusterer` ([API Link](/api/sclera-core/com/scleradb/analytics/ml/clusterer/objects/Clusterer.html))
    - Wrapper over classes implementing clustering algorithms. [Training](../sclerasql/sqlextml.md#clusterer-training) involves clustering the given data.
    - Provides a function `cluster` that assigns a cluster id to a new data point; this is used by the [`CLUSTERED WITH` clause](../sclerasql/sqlextml.md#clusterer-application).

The [Sclera - Weka Connector](../setup/components.md#sclera-weka), included with the Sclera platform, is open source and implements the interface mentioned above.
 
## Packaging and Deploying the Connector

The included [Sclera - Weka Connector](../setup/components.md#sclera-mysql) implementation uses [sbt](http://www.scala-sbt.org) for building the connector [(installation details)](http://www.scala-sbt.org/release/docs/Getting-Started/Setup.html#installing-sbt). This is not a requirement -- any other build tool can be used instead.

### Dependencies

The implementation has a dependency on:

- the library for the machine learning package used.
- the [`"sclera-core"`](../setup/components.md#sclera-core) and [`"sclera-config"`](../setup/components.md#sclera-config) core components. Note that these dependencies is annotated `"provided"` since these libraries will already be available in the `CLASSPATH` when this connector is run with Sclera.
- (optional) the test framework [`scalatest`](http://www.scalatest.org/) for running the tests.

These are specified in the build file. As an example, see the [Sclera - Weka Connector's build file](https://github.com/scleradb/sclera-plugin-weka/blob/master/build.sbt).

### Deployment Steps

Follow steps similar to those described [here](../sdk/sdkextdataaccess.md#deployment-steps).

**Note:** Please ensure that the identifier you assign to the connector is unique, that is - does not conflict with the identifier of any other available [`MLService`](#mlservice) instance.
