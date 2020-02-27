Sclera provides machine learning via first-class constructs in SQL. You can create objects such as classifiers and clusterers as easily as you create tables -- using a single `CREATE ... AS` statement. Further, Sclera provides SQL operators that enable classification/clustering of rows as a part of a SQL query.

Off the shelf libraries such as [Weka](http://www.cs.waikato.ac.nz/ml/weka/) enable you to write applications with embedded machine learning. But to use these libraries without Sclera, you need to learn the proprietary APIs, and write code that complies with the same. This takes a lot of preparation and background, and is highly disruptive.

Sclera provides a set of machine learning operators; these are a part of the SQL language, just like `JOIN`, `WHERE`, `ORDER BY`, `GROUP BY`, `HAVING` and `LIMIT` in regular SQL. These new operators are evaluated by calling the analytics library of choice -- reformatting the data, calling the right API functions, collating and reformatting the results and all the other boilerplate happens automatically, behind the scenes.

The result is a highly compact "declarative" way of doing analytics, something that you can get started on almost immediately if you know basic SQL.

## Classification
Consider a table `survey` with columns:

    age int
    gender char(1)
    region char(4)
    income real
    married boolean
    children int
    car boolean
    save_act boolean
    current_act boolean
    mortgage boolean
    isinterested boolean

This table contains data from a customer survey, with one row per response. Apart from the customer attributes, it includes a column `isinterested` on whether or not the customer is interested in your product.

Now, suppose you want to consider a new group of people and identify which of them might be interested in your product versus not.

A [classifier](http://en.wikipedia.org/wiki/Statistical_classification) is the right tool to help you do this. A classifier estimates the value of a discrete-valued "target" variable given the values of certain "feature" variables. The classifier can be trained given the sample target and feature values for a set of sample instances, called the "training data".

In the example above, the table `survey` is the training data; each row in the table (representing a unique customer) is a data instance, with the column `isinterested` as the target variable and the other columns as the feature variables.

The trained classifier can be used to "classify" new data instances -- that is, estimate target values given the feature values in these data instances. In terms of the example, this means identifying the value of `isinterested` for a new customer, which is what we started out to do.

In this section, we show [how to train classifiers](#classifier-training) in Sclera, and [use them to classify new data instances](#classifier-application) within a SQL query.

### Classifier Training
The formal syntax for creating a classifier is as follows:

    CREATE [ { TEMPORARY | TEMP } ] CLASSIFIER classifier_name ( target_column_name ) USING table_expression

This creates a classifier with the name specified by `classifier_name`. The classifier is trained using the result of the [table expression](../sclerasql/sqlregular.md#table-expression) `table_expression`, with the column with the name specified by `target_column_name` as the target, and all the remaining columns in the result as features. The optional `TEMPORARY` (shortened form: `TEMP`) modifier creates a classifier that exists for the duration of the Sclera session only; when the session ends, the classifier is deleted.

Notice the similarity with the [`CREATE TABLE AS` statement](#creating-tables-with-query-results).

In our running example, the following creates the classifier `myclassifier` and trains it on the table `survey`, with the column `isinterested` as target and the other columns as features:

    > CREATE CLASSIFIER myclassifier(isinterested) USING survey;

After the classifier is created, you can see the classifier description using the [`DESCRIBE` command](../interface/shell.md#describe):

    > DESCRIBE myclassifier;

### Classifier Application
The [classifier training](#classifier-training) learns a function that estimates the value of the target column given the values of the feature columns. Applying the classifier on a table involves computing this function on each row of the table.

This feature introduces a new [table expression](../sclerasql/sqlregular.md#table-expression) with the following syntax:

    table_expression CLASSIFIED WITH classifier_name ( class_column_name )

In this expression, the classifier with the name specified by `classifier_name` is used to classify the rows in the result of the [table expression](../sclerasql/sqlregular.md#table-expression) `table_expression`, which must include all the feature columns present in the data on which the [classifier was trained](#classifier-training) (it can include additional columns).

The result contains one row per input row. Each row contains all the columns in the input `table_expressions` and a new column, with the name as specified in `class_column_name`, containing the classifier output -- which is the classifier's estimate of the target variable given the feature values in the row.

Continuing our example, consider a new table `profiles` containing profile of a new group of people. The table `profiles` includes all the feature columns in of the classifier `myclassifier`.

We apply the classifier as follows:

    > profiles CLASSIFIED WITH myclassifier(isprospect);

The output is a table with all columns in `profiles`, and a new column `isprospect` which, for each row, contains the result of applying the classifier function given the feature column values in that row.

This expression can be used in a query just like a table or a view. For instance, the following query gives the count of the prospects and non-prospects in the `profiles` table.

    > SELECT isprospect, COUNT(*)
      FROM (profiles CLASSIFIED WITH myclassifier(isprospect))
      GROUP BY isprospect;

## Clustering
Consider a table `customers`, with one row per customer, and columns:

    age int
    income real
    married boolean
    children int
    car boolean

To understand your customers better, you may want to group these customers based on similarity across these attributes. Note that, unlike the classifier, we do not have a apriori input on these groups; there is no "target", and all the columns in the input are "feature" columns.

A [clusterer](http://en.wikipedia.org/wiki/Cluster_analysis) is the right tool for this task. A clusterer maps data instances to one among a finite set of "clusters", such that data instances with similar features belong to the same cluster, and data instances with dissimilar features belong to different clusters. The clusterer can be trained given the feature values of a set of representative data instances, called the "training data".

In the example above, the table `customers` is the training data; each row in the table (representing a unique customer) is a data instance, with the columns as the feature variables. In business terminology, the clusters are [market segments](http://en.wikipedia.org/wiki/Market_segment), and this task is an instance of [market segmentation](http://en.wikipedia.org/wiki/Market_segmentation).

The trained clusterer can be used to assign the data instances to clusters. Since these data instances are assumed to be representative, we can use the same clusterer to map additional customers to clusters. In terms of the example, this means identifying the right market segment for a new customer.

In this section, we show [how to train clusterers](#clusterer-training) in Sclera, and [use them to assign clusters to new data instances](#clusterer-application) within a SQL query.

### Clusterer Training
The formal syntax for creating a classifier is as follows:

    CREATE [ { TEMPORARY | TEMP } ] CLUSTERER clusterer_name USING table_expression

This creates a classifier with the name specified by `clusterer_name`. The clusterer is trained using the result of the [table expression](../sclerasql/sqlregular.md#table-expression) `table_expression`, with all the columns in the result as features. The optional `TEMPORARY` (shortened form: `TEMP`) modifier creates a clusterer that exists for the duration of the Sclera session only; when the session ends, the clusterer is deleted.

Notice the similarity with the [`CREATE TABLE AS` statement](#creating-tables-with-query-results) and [`CREATE CLASSIFIER` statement](#classifier-training).

In our running example, the following creates the clusterer `myclusterer` and trains it on the table `customers`:

    > CREATE CLUSTERER myclusterer USING customers;

After the clusterer is created, you can see the clusterer description using the [`DESCRIBE` command](../interface/shell.md#describe):

    > DESCRIBE myclusterer;

### Clusterer Application
The [clusterer training](#clusterer-training) learns a function that computes the cluster for a data instance given the values of all the feature columns. Applying the clusterer on a table involves computing this function on each row of the table.

This feature introduces a new [table expression](../sclerasql/sqlregular.md#table-expression) with the following syntax:

    table_expression CLUSTERED WITH clusterer_name ( cluster_column_name )

In this expression, the clusterer with the name specified by `clusterer_name` is used to assign clusters to the rows in the result of the [table expression](../sclerasql/sqlregular.md#table-expression) `table_expression`, which must include all the feature columns present in the data on which the [clusterer was trained](#clusterer-training) (it can include additional columns).

The result contains one row per input row. Each row contains all the columns in the input `table_expressions` and a new column, with the name as specified in `cluster_column_name`, containing the clusterer output.

Continuing our example, the clusterer can be applied to the table used to train the clusterer (`customers` in the above example), or to any other table or output of a query, as long as it includes all the feature columns of the training data.

We apply the clusterer to the table `customers` as follows:

    > customers CLUSTERED WITH myclusterer(clusterid);

The output is a table with all columns in `customers`, and a new column `clusterid` which, for each row, contains the id of the cluster (an integer) assigned to that row.

This expression can be used in a query just like a table or a view. For instance, the following query gives the count of the customers in each cluster.

    > SELECT clusterid, COUNT(*)
      FROM (customers CLUSTERED WITH myclusterer(clusterid))
      GROUP BY clusterid;

<a class="anchor" name="sclera-weka"></a>
## Extended Syntax for Using Specific Libraries and Algorithms
The classifier/clusterer syntax above is agnostic of the underlying library. Sclera uses [Weka](http://www.cs.waikato.ac.nz/ml/weka/) as the default library, and specific classification/clustering algorithms as default. The default library is specified in the [configuration file](../setup/configuration.md#sclera-service-default-mlservice), and can be changed if required to an alternative supported library and the algorithms therein.

The [default library](../setup/configuration.md#sclera-service-default-mlservice) can be overriden by explicitly mentioning the library (currently, [`WEKA`](http://www.cs.waikato.ac.nz/ml/weka/)) in the `CREATE` statements.

Moreover, you can select the specific algorithms to use for classification/clustering, and even provide the parameters. The algorithm names and parameters depend on the specific library. Sclera does not interpret the specified algorithm parameters, and merely passes them on to the appropriate APIs of the chosen library.

For instance, the following uses `FOOBARML` as the underlying library for creating the classifier (instead of the default, [as earlier](#classifier-training)):

    > CREATE FOOBARML CLASSIFIER myclassifier(isinterested) USING survey;

(The above assumes that a [plugin for `FOOBAR`](../sdk/sdkextml.md) has been installed.)

The following specifies the use of [`SIMPLEKMEANS` algorithm](http://weka.sourceforge.net/doc.dev/weka/clusterers/SimpleKMeans.html) for clustering in Weka:

    > CREATE WEKA CLUSTERER("SIMPLEKMEANS") myclusterer USING customers;

To further specify cluster counts as 3 (overriding the default 2):

    > CREATE WEKA CLUSTERER("SIMPLEKMEANS", "-N 3") myclusterer USING customers;

The extended formal syntax for classification/clustering, incorporating these overrides, is discussed below.

### Extended Syntax for Classification
The extended formal syntax for creating a classifier is as follows:

    CREATE [ { TEMPORARY | TEMP } ] [ library_name ] CLASSIFIER [ ( algorithm_name [ , algorithm_options ] ) ] classifier_name ( target_column_name ) USING table_expression

- The optional `library_name` specifies the machine learning library to use for the task.
    - If not specified, the `WEKA` is used (this default can be modified using the [`sclera.service.default.mlservice` configuration parameter](../setup/configuration.md#sclera-service-default-mlservice)).
- The optional `algorithm_name` identifies the algorithm to be used in training the classifier.
    - The following are supported: for `WEKA`:
        - `J48` ([documentation](http://weka.sourceforge.net/doc.dev/weka/classifiers/trees/J48.html))
        - `HOEFFDINGTREE` ([documentation](http://weka.sourceforge.net/doc.dev/weka/classifiers/trees/HoeffdingTree.html))
        - `LMT` ([documentation](http://weka.sourceforge.net/doc.dev/weka/classifiers/trees/LMT.html))
        - `M5P` ([documentation](http://weka.sourceforge.net/doc.dev/weka/classifiers/trees/M5P.html))
        - `RANDOMFOREST` ([documentation](http://weka.sourceforge.net/doc.dev/weka/classifiers/trees/RandomForest.html))
        - `REPTREE` ([documentation](http://weka.sourceforge.net/doc.dev/weka/classifiers/trees/REPTree.html))
        - `CLASSIFICATIONVIAREGRESSION` ([documentation](http://weka.sourceforge.net/doc.dev/weka/classifiers/meta/ClassificationViaRegression.html))
        - `DECISIONTABLE` ([documentation](http://weka.sourceforge.net/doc.dev/weka/classifiers/rules/DecisionTable.html))
        - `M5RULES` ([documentation](http://weka.sourceforge.net/doc.dev/weka/classifiers/rules/M5Rules.html))
        - `ONER` ([documentation](http://weka.sourceforge.net/doc.dev/weka/classifiers/rules/OneR.html))
        - `LOGISTIC` ([documentation](http://weka.sourceforge.net/doc.dev/weka/classifiers/functions/Logistic.html))
        - `NAIVEBAYES` ([documentation](http://weka.sourceforge.net/doc.dev/weka/classifiers/bayes/NaiveBayes.html))
    - If not specified, `J48` (for `WEKA`) is used as the default.
- The optional `algorithm_options` provides the configuration options (parameters) for the algorithm identified by `algorithm_name`.
    - These options are passed as a *single* string, just as in a command line.
    - Please refer to the Weka documentation the respective algorithms, linked above, for details of the accepted options and defaults.
    - If not specified, the default parameters for the specified algorithm are used.
- Remaining parameters are as in the [abridged syntax](#classifier-training) discussed earlier.

### Extended Syntax for Clustering
The extended formal syntax for creating an clusterer is as follows:

    CREATE [ { TEMPORARY | TEMP } ] [ library_name ] CLUSTERER [ ( algorithm_name [ , algorithm_options ] ) ] clusterer_name USING table_expression

- The optional `library_name` specifies the machine learning library to use for the task.
    - In the current version, only `WEKA` is accepted as Sclera currently only interfaces with [Weka](http://www.cs.waikato.ac.nz/ml/weka) for clustering.
    - If not specified, `WEKA` is used (this default can be modified using the [`sclera.service.default.mlservice` configuration parameter](../setup/configuration.md#sclera-service-default-mlservice)).
- The optional `algorithm_name` identifies the algorithm to be used in training the clusterer.
    - The following are supported for `WEKA`:
        - `SIMPLEKMEANS` ([documentation](http://weka.sourceforge.net/doc.dev/weka/clusterers/SimpleKMeans.html))
        - `COBWEB` ([documentation](http://weka.sourceforge.net/doc.dev/weka/clusterers/Cobweb.html))
        - `EM` ([documentation](http://weka.sourceforge.net/doc.dev/weka/clusterers/EM.html))
        - `FARTHESTFIRST` ([documentation](http://weka.sourceforge.net/doc.dev/weka/clusterers/FarthestFirst.html))
        - `HIERARCHICAL` ([documentation](http://weka.sourceforge.net/doc.dev/weka/clusterers/HierarchicalClusterer.html))
    - If not specified, `SIMPLEKMEANS` is used as the default.
- The optional `algorithm_options` provides the configuration options (parameters) for the algorithm identified by `algorithm_name`.
    - These options are passed as a *single* string, just as in a command line.
    - Please refer to the Weka documentation the respective algorithms, linked above, for details of the accepted options and defaults.
    - If not specified, the default parameters for the specified algorithm are used.
- Remaining parameters are as in the [abridged syntax](#clusterer-training) discussed earlier.
