In this document we list the Sclera's configuration parameters, specified in `<sclera-root>/config/sclera.conf`, where `<sclera-root>` is the [directory where Sclera is installed](../setup/install.md#installing-sclera-core-packages-and-shell).

The syntax of the file should be self-evident. For a detailed description of the underlying HOCON (Human-Optimized Config Object Notation) syntax, please refer to the [specification document](https://github.com/typesafehub/config/blob/master/HOCON.md).

| Parameter | Description
| --------- | -----------
| <a class="anchor" name="sclera-exec-batchsize"></a>`sclera.exec.batchsize` | Number of rows to batch together for bulk input/output (default 100).
| <a class="anchor" name="sclera-location-datacache"></a>`sclera.location.datacache` | Location for cache data/intermediate results
| <a class="anchor" name="sclera-location-default"></a>`sclera.location.default` | Location to be used when explicit location is not specified
| <a class="anchor" name="sclera-location-schema-database"></a>`sclera.location.schema.database` | Database in the [schema DBMS](#sclera-location-schema-dbms) for storing the [data schema](../intro/technical.md#metadata-store)
| <a class="anchor" name="sclera-location-schema-dbms"></a>`sclera.location.schema.dbms` | DBMS for storing the [data schema](../intro/technical.md#metadata-store)
| <a class="anchor" name="sclera-location-temporary"></a>`sclera.location.temporary` | Location for storing [temporary data](../intro/technical.md#cache-store)
| <a class="anchor" name="sclera-service-assetdir"></a>`sclera.service.assetdir` | Directory for storing component assets
| <a class="anchor" name="sclera-service-default-mlservice"></a>`sclera.service.default.mlservice` | Default machine learning service
| <a class="anchor" name="sclera-service-default-nlpservice"></a>`sclera.service.default.nlpservice` | Default natural language processing service
| <a class="anchor" name="sclera-shell-explain"></a>`sclera.shell.explain` | Default value for explain (true = on, or false = off)
| <a class="anchor" name="sclera-shell-history"></a>`sclera.shell.history` | File for storing Sclera shell history
| <a class="anchor" name="sclera-shell-prompt"></a>`sclera.shell.prompt` | Shell prompt
| <a class="anchor" name="sclera-storage-datadir"></a>`sclera.storage.datadir` | Directory for storing datafiles for embedded H2
| <a class="anchor" name="sclera-storage-objectdir"></a>`sclera.storage.objectdir`| Directory for storing Sclera objects 
