# hive-cassandra-handler

This handler provides hive support for cassandra.

## Check out and Installation

check out the project from *https://github.com/tuplejump/cash.git*.

    > git clone https://github.com/tuplejump/cash.git

Go to branch *cas-support-cql*

    > git checkout cas-support-cql

Go to cassandra-handler folder run the maven package command.

    > mvn package

This generates a **hive-cassandra-x.x.x.jar** file in target folder and all other project dependencies are downloaded to target/dependency.

Copy the **target/hive-cassandra-x.x.x.jar** to the hive lib directory

Copy **target/dependency/cassandra-all-x.x.x.jar** and **target/dependency/cassandra-thrift-x.x.x.jar** to hive lib directory.

## Running

Start hive

    > bin/hive

Create a database in hive (or use an existing one)

To create a cql3 table in cassandra from hive, execute the following command

     hive> CREATE EXTERNAL TABLE test.messages(message_id string, author string, body string)
        STORED BY 'org.apache.hadoop.hive.cassandra.cql.CqlStorageHandler'
        WITH SERDEPROPERTIES ("cql.primarykey" = "message_id, author", "comment"="check", "read_repair_chance" = "0.2",
        "dclocal_read_repair_chance" = "0.14", "gc_grace_seconds" = "989898", "bloom_filter_fp_chance" = "0.2",
        "compaction" = "{'class' : 'LeveledCompactionStrategy'}", "replicate_on_write" = "false", "caching" = "all")

   where 'test' is the keyspace in cassandra. The above query also creates a column family in cassandra if does not exist.

   *Note: By default hive tries to use cassandra running on localhost at port 9160.*
   *To change these specify **cassandra.host** and **cassandra.port** in the SerDeProperties while creating a table.*
   *Hive connects to the specified cassandra instance for further queries to the table.*

To create a keyspace that does not exist in cassandra execute the following query

    hive> CREATE EXTERNAL TABLE test.messages(row_key string, col1 string, col2 string)
    STORED BY 'org.apache.hadoop.hive.cassandra.cql.CqlStorageHandler' WITH SERDEPROPERTIES("cql.primarykey" = "row_key")
    TBLPROPERTIES ("cassandra.ks.name" = "mycqlks", "cassandra.ks.stratOptions"="'DC':1, 'DC2':1",
    "cassandra.ks.strategy"="NetworkTopologyStrategy");

  *Note: For brevity, only minimal SERDEPROPERTIES are  given in the above query.*

If 'test' keyspace does not exist in cassandra it will be created.

Inserting values into CQL3 table through hive:

    hive> insert into table messages select * from tweets;

The values from tweets table are appended to messages table.

  *Note: With Cassandra INSERT OVERWRITE is same as INSERT INTO as Cassandra merges changes if keys are same.*

Retrieving values from a CQL3 table using hive:

    hive> select * from messages;

  *Note: If local mode execution is not enabled, hive compiler generates **map-reduce jobs** for most queries. These jobs are then submitted to the Map-Reduce cluster.*
  *The map-reduce jobs need hive-cassandra-handler, cassandra-all and cassandra-thrift jars.*
  *Point the **HIVE_AUX_JARS_PATH** environment variable to the location containing these jars to run those jobs successfully.*

While CqlStorageHandler is used to create/access cql3 tables in cassandra, CassandraStorageHandler can be used to create/access
thrift tables in cassandra.
