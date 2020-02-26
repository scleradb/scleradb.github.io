This document describes Sclera extensions for data cleaning.

## Automatic Type Inference

Data sourced from web-services typically does not include the data types -- everything from numbers to dates are available as strings. These string values need to be cast to appropriate data types before they can be used in further computations.

When the data types are known, the type-casting is can be hardcoded; but this is dangerous when the data format from an external service changes without notice.

When the data types are not known, as in the case of ad-hoc data access, the data types need to be inferred manually. This is not only error-prone, it also does not scale to data sets with large number of columns.

Sclera's `TYPEINFER` operator automates the type inference and casting. It intelligently infers the type of the specified columns from the data (you can optionally provide hints on how many rows are enough), and casts the string values accordingly.

The syntax is as follows:

    table_expression TYPEINFER [ ( [ target_columns ] [ NULLS ( null_values ) ] [ LIMIT limit ] ) ]

where:

- `table_expression` is an arbitrary [table expression](../sclerasql/sqlregular.md#table-expression)
- `target_columns` is a comma-separated list of columns in `table_expression` for which the type inference is to be done. If the type of any of these columns is not `CHAR` or `VARCHAR`, it is silently ignored. We can also specify a [wildcard](../sclerasql/sqlregular.md#abbreviations) instead of an explicit column list. If the `target_columns` list is not specified, all columns of type `CHAR` or `VARCHAR` in the input are included.
- `null_values` is a list of values which, if seen in any of the `target_columns`, must be substituted for `NULL`. For instance, a common string used to mark unavailable values in datasets is "N/A" and "not found". Saying `NULLS("N/A", "not found")` will replace all occurence of the strings with NULL in the output.
- `limit` is the number of rows that the operator should see before it decides on the column types. If not specified, all input rows will be scanned to infer the column types, and then a second scan will cast the values in each row to the inferred types.

As an example, the following statement infers the types of the columns in the input file `input.csv` by looking at the first data row, and then casts all values to the inferred types on the fly as they are read from the file.

    EXTERNAL CSV("input.csv") TYPEINFER(LIMIT 1)

Automatic type inference discussed above works well when the input values (strings) contain the data in a standardised format that can be easily parsed. Integers and floating point numbers have standardized format, so running `TYPEINFER` on a CSV file containing only numeric values works well.

However, dates and times come in all sorts of formats; `TYPEINFER` only understands the standard formats such as "2016-01-06 09:15:59.0". Anything non-standard, such as "08/01", and `TYPEINFER` will not know how to parse (actually, neither will a human - does "08/01" mean Jan 8 or Aug 1?). This is where the `TEXT PARSE` clause discussed next becomes useful.

## Text Parsing

Test parsing involves parsing the values in an input column to generate one or more columns in the output.

The syntax is:

    table_expression TEXT PARSE ( pattern ) IN input_column TO target_columns

where:

- `table_expression` is an arbitrary [table expression](../sclerasql/sqlregular.md#table-expression)
- `patterns` is a [Java regular expression](https://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html), containing one or more [capturing groups](https://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html), which are subpatterns enclosed in parenthesis
- `input_column` is a column in the output of the `table_expression`, of type `CHAR` or `VARCHAR`
- `target_columns` is a list of column names, one for each capturing group in the `pattern`; each column in the list is associated with a capturing group, in order of occurrence

For each row in the output of `table_expression` (the "input row"), the operator parses the string in the column `input_column` using the specified `patern`.

The output consists of all columns values in the input row, augmented with new columns of type `VARCHAR`, with names specified in the `target_columns`, containing the substrings extracted by matching the `capturing groups` in order.

For instance, consider a table `input_table` with data as follows:

    id | month
    ---+------
    1  | '1/86'
    2  | '2/86'

Then, the following statement parses the string in column `month` in each row, separating the numbers before and after the `/` and places them in result columns `m` and `y` respectively.

    input_table TEXT parse "(\d+)/(\d+)" IN month TO (m, y)

The result is:

    id | m   | y
    ---+-----+----
    1  | '1' | '86'
    2  | '2' | '86'

The type of columns `m` and `y` is `VARCHAR`; they can be cast to `INT` if needed.

## Data Imputation

Consider a dataset with missing values in a column. A missing value is assumed as represented by `NULL` in the following discussion.

Data imputation involves filling in the missing values with reasonable estimates. In this section, we describe the ways in which data imputation can be done in Sclera.

### Data Imputation with Regular SQL

In the simplest case, we may want to fill in the missing data with a constant, or an value we know how to compute.

This can be done using a trivial application of the [`COALESCE` function](http://www.postgresql.org/docs/current/static/functions-conditional.html#FUNCTIONS-COALESCE-NVL-IFNULL). The [CASE expression](http://www.postgresql.org/docs/current/static/functions-conditional.html#FUNCTIONS-CASE) is more general, in that it allws you to fill in values conditionally.

The harder case is when you *do not* know how to compute the value to fill in; this is discussed next.

### Data Imputation using Machine Learning

Recall that [classifiers](../sclerasql/sqlextml.md#classification) learn how to compute the value of a given column (target) given the values of other columns in a row.

The idea behind the machine learning approach to data imputation is to:

- [Train a classifier](../sclerasql/sqlextml.md#classifier-training) on clean dataset. This clean dataset could be a subset of the input dataset, containing rows with all values available. Or, it could be a reference dataset available independently.
- [Apply the classifier](../sclerasql/sqlextml.md#classifier-application) to the rows with missing values of the classifier's target column. This will generate estimates for the missing values.

The syntax is as follows:

    table_expression IMPUTED WITH classifier_name ( target_column ) [ FLAG flag_column ) ]

where:

- `table_expression` is an arbitrary [table expression](../sclerasql/sqlregular.md#table-expression)
- `classifier_name` is the name of classifier, already trained on a clean subset of the given dataset
- `target_column` is the column containing the missing values, which need to be estimated and filled in
- `flag_column` is an optional column name -- if specified, a column of this name in the result will contain a value `true` if the `target_column` was originally `NULL` and is now filled with an estimated value, or `false` otherwise.
