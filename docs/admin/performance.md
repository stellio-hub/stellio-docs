# Performance

## Principles

Performance in Stellio is mainly related to the performance of the underlying database (PostgreSQL). However, the database settings to achieve good performance when writing data (as few indexes as possible) are often different to the settings needed to achieve good performance when reading data (specific indexes tuned to data retrieval).

Thus, by default, the Stellio databases are configured to provide a good balance between writing and reading. If you have more specific scenarios in one or another area, you may have to adapt these settings. You may also have to fine tune the settings of Timescale (for instance, sizes of chunks).

## Load test campaign

An NGSI-LD load test suite is available in [GitHub](https://github.com/stellio-hub/ngsild-load-tests). It describes how to install and configure it, as well as all the scenarions that are currently available.

The objective is to be as close as possible to data that is similar to data used in real life projects. Thus, the [template entity](https://github.com/stellio-hub/ngsild-load-tests/blob/master/src/data/template_entity.json) that is used in the different scenarios is composed of 6 properties (some being multi-instance ones), 1 relationship and 1 geolocation. Also these first results are currently focused on the most typical endpoints used in everyday life of a project.

## Current state of Stellio

The results presented here have been obtained in April 2023 using the following basic setup:

* Stellio: 8 vCPU / 16 GB RAM / 160 GB disk (VPS Elite from OVH)
* Load tests: 4 vCPU / 8 GB RAM / 160 GB disk (VPS Comfort from OVH)

Each scenario has been run 3 times, the numbers presented below are an average of the 3 runs for each scenario.

### Create entity

| VUs    | Number of entities created     | Requests / second    | Response time (95th percentile) | 
|--------|--------------------------------|----------------------|---------------------------------|
| 10     | 10000                          | 131                  | 137 ms                          |
| 10     | 100000                         | 163                  | 78 ms                           |
| 10     | 1000000                        | 155                  | 76 ms                           |
| 50     | 100000                         | 156                  | 366 ms                          |

### Partial attribute update

| VUs    | Initial setup     | Number of updates | Requests / second | Response time (95th percentile) |
|--------|-------------------|-------------------|-------------------|---------------------------------|
| 10     | 1000 entities     | 10000             | 142               | 79 ms                           |
| 10     | 1000 entities     | 100000            | 163               | 78 ms                           |
| 50     | 1000 entities     | 100000            | 156               | 366 ms                          |

### Query entities by type and values of properties

The query looks like `/ngsi-ld/v1/entities?type={randomType}&q=aProperty>{randomValue};anotherProperty=={aRandomExistingValue}`

| VUs    | Initial setup                          | Number of queries | Requests / second | Response time (95th percentile) |
|--------|----------------------------------------|-------------------|-------------------|---------------------------------|
| 10     | 1000 entities<br>5 different types     | 10000             | 39                | 311 ms                          |
| 10     | 10000 entities<br>10 different types   | 100000            | 39                | 318 ms                          |
| 10     | 100000 entities<br>10 different types  | 10000             | 11                | 1350 ms                         |

### Query entities by type and object of a relationship

The query looks like `/ngsi-ld/v1/entities?type={randomType}&q=aProperty>{randomValue};aRelationship=={aRandomExistingObject}`

| VUs    | Initial setup                         | Number of queries | Requests / second | Response time (95th percentile) |
|--------|---------------------------------------|-------------------|-------------------|---------------------------------|
| 10     | 1000 entities<br>5 different types    | 10000             | 77                | 169 ms                          |
| 10     | 10000 entities<br>10 different types  | 100000            | 43                | 298 ms                          |
| 10     | 100000 entities<br>10 different types | 10000             | 26                | 547 ms                          |

### Retrieve temporal evolution of an entity (aggregated values)

The query looks like `/ngsi-ld/v1/temporal/entities/{entityId}?timerel=after&timeAt=2023-01-01T00:00:00Z&aggrMethods={randomAggregateMethod}&aggrPeriodDuration=PT1S&options=aggregatedValues`

| VUs    | Initial setup                           | Number of queries | Requests / second | Response time (95th percentile) |
|--------|-----------------------------------------|-------------------|-------------------|---------------------------------|
| 10     | 10 entities<br>1000 temporal instances  | 1000              | 137               | 88 ms                           |
| 10     | 10 entities<br>10000 temporal instances | 1000              | 86                | 137 ms                          |
| 10     | 10 entities<br>100000 temporal instances| 1000              | 42                | 321 ms                          |
