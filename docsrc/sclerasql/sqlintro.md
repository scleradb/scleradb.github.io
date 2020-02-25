This section introduces the SQL constructs -- standard and proprietary -- supported by Sclera. Detailed documentation on these constructs, including the syntax and example usage, appear in the later sections; please follow the links in the descriptions below.

The SQL constructs supported by Sclera come in the following flavors:

**[Regular SQL constructs](/doc/ref/sqlregular)**
These standard constructs enable you to create, populate, query, update and delete base tables. Sclera supports a large subset of standard SQL. In addition, Sclera makes the language modern and succint by adding shortcuts to several commonly used constructs.

See the [detailed documentation](/doc/ref/sqlregular) for further information.

**[SQL extensions for Cross-tabulation](/doc/ref/sqlcrosstab)**
These constructs are used to convert data in multiple rows to columns in a single row (`PIVOT`), and vice-versa (`UNPIVOT`).

The semantics of these functions is the same as in [Oracle 11g](http://www.oracle.com/technetwork/articles/sql/11g-pivot-097235.html) and [MS SQL Server 2008](http://goo.gl/gzzBgK), but with a slightly modified (in our opinion, simplified) syntax.

See the [detailed documentation](/doc/ref/sqlcrosstab) for further information.

**[SQL extensions for ad-hoc external data access](/doc/ref/sqlextdataaccess)**
These extensions enable you to access external data -- on-disk or streaming, local or over the internet -- from within SQL.

See the [detailed documentation](/doc/ref/sqlextdataaccess) for further information.

Note that Sclera's connectivity to external data sources is not limited to the above -- Sclera includes an SDK that can be used to develop extensions for arbitrary external datasources.  The [Sclera Connector Development SDK](/doc/ref/sdk) document describes the SDK and includes working examples.

**[SQL extensions for processing ordered data](/doc/ref/sqlextordered)**
These extensions enable powerful constructs that simplify processing of ordered data.

The features include immediate access to prior tuples and running aggregates, input splitting based on positional criteria, regular expression matches across rows, and much more -- available without any materialization and with bounded memory overheads. These features eliminate the need of self-joins in most queries, leading to orders of magnitude gains in query performance.

Furthermore, given the single-pass processing with minimal memory overheads, these constructs can be used to process streaming data as well.

See the [detailed documentation](/doc/ref/sqlextordered) for further information.

<!--
**[SQL extension for pattern learning and predictive analytics on ordered data](/doc/ref/sqlextpred)**
This extension learns a state machine from ordered datasets. This state machine can be seen as a statistical summary of the underlying events, and can be used to mine insights or predict future events.

A simple example is mining of visitor flow from a website clickstream, and predicting the next page a visitor may visit based on the past behavior.

See the [detailed documentation](/doc/ref/sqlextpred) for further information.
-->

**[SQL extensions for machine learning](/doc/ref/sqlextml)**
These extensions enable you to incorporate [classification](http://en.wikipedia.org/wiki/Statistical_classification), [clustering](http://en.wikipedia.org/wiki/Cluster_analysis) and [association rule learning](http://en.wikipedia.org/wiki/Association_rule_learning) in your SQL queries. Sclera interfaces with well-established external libraries (currently [Weka](http://www.cs.waikato.ac.nz/ml/weka/) and [Mahout](http://mahout.apache.org)) for the task; the query can optionally specify the specific library to use, the specific algorithm within the library and the associated parameter values.

See the [detailed documentation](/doc/ref/sqlextml) for further information.

**[SQL extensions for text analytics](/doc/ref/sqlexttext)**
This extension enables Sclera to [identify and extract named entities](http://en.wikipedia.org/wiki/Named-entity_recognition) from free-form text present in a table column.

A simple example is extracting person names, product names and locations from customer tweets; the extracted information can be used by companies to join with the "structured" customer and product data; this enriched data can be analyzed using the extensions mentioned earlier for helpful insights.

See the [detailed documentation](/doc/ref/sqlexttext) for further information.
