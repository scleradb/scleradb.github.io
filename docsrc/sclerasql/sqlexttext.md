Sclera can identify and extract entities, such as person and location names, from free-form text. This is done using the [Sclera - Apache OpenNLP Connector](/doc/ref/components#sclera-opennlp) which builds on the [Apache OpenNLP](http://opennlp.apache.org) library.

While we may do not know the context in which these entities were mentioned in the text, this is still very useful as it enables you to identify related information from your database by doing joins with tables containing product information, and also filter and/or aggregate the messages based on location, persons mentioned, and so on.

This feature introduces a new [table expression](#table-expression) with the following syntax:

    table_expression TEXT [ ( language_code ) ] EXTRACT( entity_type [, ...] ) IN input_column_name [ TO ( entity_column_name [, label_column_name ] ) ]

- The mandatory `table_expression` is a table expression
- The optional `language_code` specifies the language of the text; it can be `"da"` (for Danish), `"de"` (for German), `"en"` (for English), `"dl"` (for Dutch), `"pt"` (for Portuguese) or `"se"` (for Swedish). If not specified, it is taken to be `"en"` (i.e. English).
- The mandatory `input_column_name` is the name of a column in `table_expression` containing free-form text. This forms the input to the text extractors.
- Each `entity_type` in the mandatory comma-separated list specifies the type of entity to extract. The possible values for this parameter depends upon the available entity extraction models; see the [setup notes](#model-setup) for details.
- The optional `entity_column_name` is the name of a new column that contains the extracted entities. If not specified, Sclera assigns the name `{input_column_name}_entity` to the column.
- The optional `label_column_name` is the name of a new column that contains the labels of the extracted entities in `entity_column_name`; label of an extraction is the `entity_type` associated with the extracted entity. If not specified, Sclera assigns the name `{input_column_name}_label` to the column.

The result contains one row per entity found. Each row contains all the columns in the input `table_expressions` and the two new columns containing the extracted entity and its label respectively.

As an example, consider a table `messages` containing customer messages, with columns `custid` containing the customer id, `msgdate` containing the message date, and `msgcontent` containing the content of the message from the customer.

To identify the locations and persons in the messages in the column `msgcontent` in the table `message`:

    > messages TEXT EXTRACT("location", "person") IN msgcontent TO (entity, entitylabel);

The output of this expression is a table with all columns in `messages`, and two new columns `entity` and `entitylabel` which contain the extracted entity and its label (`"location"` or `"person"`) respectively. This table contains one row in the output for each entity discovered; the columns in `messages` are repeated in each row that contains an associated entity.

The table expression can be used in a query just like a base table or a view. For instance, the following query joins the `messages` table with a `stores` table and find the unique number of messages for each store:

    > SELECT S.storeid, COUNT(distinct M.msgid)
      FROM (messages TEXT EXTRACT "location" IN msgcontent TO location) M
           JOIN stores S ON (M.location = S.location)
      GROUP BY S.storeid;

## Model Setup

To be able to extract an `entity_type` in the language specified by `language_code`, the following files need to exist:

- `"$SCLERA_ASSETS/opennlp/{language_code}-sent.bin"`, containing the sentence detector model for the language
- `"$SCLERA_ASSETS/opennlp/{language_code}-ner-{entity_type}.bin"`, containing the extraction model for the entity type in the language

In the above, `$SCLERA_ASSETS` is the directory given by the [configuration parameter `sclera.services.assetdir`](/doc/ref/configuration#sclera-services-assetdir).

For instance, to use a `location` model for English (`"en"`), the sentence detector file `"$SCLERA_ASSETS/opennlp/en-sent.bin"` and the extraction model file `"$SCLERA_ASSETS/opennlp/en-ner-location.bin"` need to exist.

The text mining engine used by Sclera is [Apache OpenNLP version 1.5](http://opennlp.apache.org/), and the pre-trained models are [available for download](http://opennlp.sourceforge.net/models-1.5/).

The [available pre-trained models](http://opennlp.sourceforge.net/models-1.5/) include `person`, `location`, `organization`, `date`, `time`, `money` and `percentage` models for Danish, German, English, Dutch, Portuguese and Swedish. Please [download](http://opennlp.sourceforge.net/models-1.5/) the required models and place them in the `"$SCLERA_ASSETS/opennlp"` directory.

## Model Accuracy

Text extraction models are not perfect. The accuracy of an extraction model depends on how similar the document on which the extraction is being applied is to the set of documents used to train the extraction model. As such, the extraction may occasionally extract erroneous entities, or miss certain entities.

If the accuracy with the [available models](http://opennlp.sourceforge.net/models-1.5/) on your data is not acceptable, you may want to generate your own models. New models can be generated on custom text collections using the [Apache OpenNLP command line tools](http://opennlp.apache.org/documentation/1.5.3/manual/opennlp.html) for training [sentence detectors](http://opennlp.apache.org/documentation/1.5.3/manual/opennlp.html#tools.sentdetect.training) and [entity extractors](http://opennlp.apache.org/documentation/1.5.3/manual/opennlp.html#tools.namefind.training).


