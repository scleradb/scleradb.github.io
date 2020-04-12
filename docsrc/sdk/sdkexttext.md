In this document, we show how to build custom connectors to any NLP / text analytics library to perform text analytics tasks. These connectors handle the invocation of the underlying library to process text data in table columns, in the query processing pipeline.

The component [Sclera - Apache OpenNLP Connector](../setup/components.md#sclera-opennlp) is built using this SDK. For examples of how the connector is used in Sclera, please refer to the [documentation on using text analytics in SQL](../sclerasql/sqlexttext.md).

## Building Text Analytics Library Connectors

To build a custom datasource connector, you need to provide implementations of the following abstract classes in the SDK:

- <a class="anchor" name="nlpservice"></a> `NlpService` ([API Link](/api/sclera-core/com/scleradb/analytics/nlp/service/NlpService.html))
    - Provides text analytics operators as a service to Sclera, using the specified library.
    - Contains an `id` that identifies this service.
    - Contains the method `createObject` that is used to create a new task object for the task named in the parameter `taskName` for this service.
- <a class="anchor" name="nlptask"></a> `NlpTask` ([API Link](/api/sclera-core/com/scleradb/analytics/nlp/objects/NlpTask.html))
    - Wrapper over classes implementing text analytics algorithms.
    - Provides a function `eval` that takes a data stream (an iterator over rows, with associated metadata) as input and returns the same data stream, with each row augmented by columns `resultCols` containing the output of executing the task `taskName` on the text in column `inputCol`. If the evaluation on a row emits multiple evaluation results, the input row is repeated in the output for each such result.

The [Sclera - Apache OpenNLP Connector](../setup/components.md#sclera-opennlp), included with the Sclera platform, is open source and implements the interface mentioned above. The code for the [Sclera - Apache OpenNLP Connector](../setup/components.md#sclera-opennlp), in Scala, also appears as an illustrative example in the [Sclera Extensions (Scala) Github repository](https://github.com/scleradb/sclera-extensions-scala).
 
## Packaging and Deploying the Connector

The included [Sclera - Apache OpenNLP Connector](../setup/components.md#sclera-mysql) implementation uses [sbt](http://www.scala-sbt.org) for building the connector [(installation details)](http://www.scala-sbt.org/release/docs/Getting-Started/Setup.html#installing-sbt). This is not a requirement -- any other build tool can be used instead.

### Dependencies

The implementation has a dependency on:

- the library for the text analytics package used.
- the [`"sclera-core"`](../setup/components.md#sclera-core) and [`"sclera-config"`](../setup/components.md#sclera-config) core components. Note that these dependencies is annotated `"provided"` since these libraries will already be available in the `CLASSPATH` when this connector is run with Sclera.
- (optional) the test framework [`scalatest`](http://www.scalatest.org/) for running the tests.

These are specified in the build file. As an example, see the [Sclera - Apache OpenNLP Connector's build file](https://github.com/scleradb/sclera-plugin-opennlp/blob/master/build.sbt).

### Deployment Steps

Follow steps similar to those described [here](../sdk/sdkextdataaccess.md#deployment-steps).

**Note:** Please ensure that the identifier you assign to the connector is unique, that is - does not conflict with the identifier of any other available [`NlpService`](#nlpservice) instance.
