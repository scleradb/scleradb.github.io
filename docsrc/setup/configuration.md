In this document we list the Sclera's configuration parameters, specified in `$SCLERA_HOME/assets/config/sclera.conf`, where `$SCLERA_HOME` is the [directory where Sclera is installed](../setup/install.md#sclera-home).

The syntax of the file should be self-evident. A detailed description of the underlying HOCON (Human-Optimized Config Object Notation) sytax specification can be accessed [here](https://github.com/typesafehub/config/blob/master/HOCON.md).

| Parameter | Description
| --------- | -----------
| <a class="anchor" name="sclera-exec-maxbulksize"></a>`sclera.exec.maxbulksize` | Number of rows to batch together for bulk input/output (default 100).
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
| <a class="anchor" name="sclera-user-email"></a>`sclera.user.email` | Email address (your login at the Sclera website)
| <a class="anchor" name="sclera-web-baseurl"></a>`sclera.web.baseurl` | Sclera website URL
