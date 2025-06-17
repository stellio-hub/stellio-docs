# Temporal data compression

## Warning only work with timescale version > 2.18 or stellio/stellio-timescale-postgis version > 16-2.20

## Enabling compression
You can enable timescale compression on temporal instances by running.
see [timescale documentation](https://docs.tigerdata.com/api/latest/hypertable/)
```sql
-- enable column store
ALTER TABLE attribute_instance SET(
    timescaledb.enable_columnstore,
    timescaledb.orderby = 'time DESC',
    timescaledb.segmentby = 'temporal_entity_attribute');

-- add compression policy
CALL add_columnstore_policy('attribute_instance', after => INTERVAL '30d');
-- prevent potential "tuple decompression limit exceeded by operation" issue
SET timescaledb.max_tuples_decompressed_per_dml_transaction = 1000000;
```

## Configure compression 
### Configure the job
Enabling compression create a job that compress your data every 12h. You can find the job by using : 
```sql
-- get all compression jobs on attribute_instance
SELECT * FROM timescaledb_information.jobs
WHERE proc_name = 'policy_compression' AND hypertable_name='attribute_instance';

```
you can also change the scheduled time with :
```sql
-- change execution to once a day 
SELECT alter_job(1000,schedule_interval=>'24 hours'::interval); -- replace 1000 by your job id if different
```

### Configure the chunk
Timescale compression work with chunks, a chunk is a part of the table that is compressed. 
By default a chunk contain 7 days of data, which is optimal for data points every hour to every 10 minutes. 
Increasing the chunk time interval will reduce the size of your data but will impact request performance (see [choosing the right chunk interval](https://docs.timescale.com/use-timescale/latest/hypertables/#best-practices-for-time-partitioning))
```sql
SELECT set_chunk_time_interval('attribute_instance', INTERVAL '30 days');
```

### configuration troubleshoot
If you run into `tuple decompression limit exceeded by operation` error you should augment the max_tuples_decompressed_per_dml_transaction variables.
````sql
SET timescaledb.max_tuples_decompressed_per_dml_transaction = 1000000;
````
You can find other troubleshoot [here](https://docs.timescale.com/use-timescale/latest/compression/troubleshooting/)


## See compression statistic
If you want to see the statistic of your compression you can do :
````sql
SELECT * FROM hypertable_compression_stats('attribute_instance');
````

## Disabling compression
You can also disable compression rule by adding :
````sql
CALL remove_columnstore_policy('attribute_instance');
````

## manipulate chunks

see your chunks : 
````sql
-- with chunk names
SELECT show_chunks('attribute_instance');
-- with some statistics :
SELECT * FROM chunk_columnstore_stats('attribute_instance');
-- with even more statistics :
SELECT  *
FROM timescaledb_information.chunks
WHERE hypertable_name = 'attribute_instance';
````

compress or decompress a chunk :
````sql
SELECT compress_chunk('_timescaledb_internal._hyper_2_1_chunk');
SELECT decompress_chunk('_timescaledb_internal._hyper_2_1_chunk');
````
split or merge chunks :
````sql
CALL split_chunk('_timescaledb_internal._hyper_2_8_chunk','2025-06-15 00:00:00.000000 +00:00');
CALL merge_chunks('_timescaledb_internal._hyper_2_1_chunk', '_timescaledb_internal._hyper_2_2_chunk');
````

## performance consideration 
### data storage
Enabling timescale compression can help reduce the size taken by the temporal instances by up to 80% for huge chunk size.

### Endpoint response time 

#### Basic consumption (GET /entities & /attrs)
The basic consumptions endpoints are not impacted by the temporal data compression.

#### Temporal Consumption (Get /temporal)
Compression as a really low impact on temporal data consumption.
We have seen a 12-20 ms increased time for temporal get request when enabling temporal data compression.
example of request impact :
 - get 100 entities with lastn=1 :  130ms -> 147ms (13%)
 - get 200 entities with lastn=1 :  201ms -> 214ms (6%)
 - get 100 entities with all data:  1030ms -> 1051ms (2%)

#### Creation (Post /temporal, Put,Patch,Post /entities & /attrs)
Compression of new data is done in a separate job running every 12h so the creation of new temporal data is not affected by compression.
Special case: adding a new temporal instance with the same observedAt as an existing instance will update the instance instead of adding a new one.  
In that case you will have performance impact as described below.
- add a new temporal instance

####  Modification (Patch & Delete /temporal)
Compression as a bigger impact when modifying already compressed data since the modified chunks as to be recompressed.
example of request impact :
 - update a temporal instance :  23ms -> 207ms
