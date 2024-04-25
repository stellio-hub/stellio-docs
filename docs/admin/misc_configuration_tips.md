# Misc configuration tips

## Increase the max allowed size for headers

If sending HTTP requests with headers having a large size, you may have to increase the max allowed size (which is by default set to 8KB).

It may be done by configuring the `server.max-http-request-header-size` property. It has to be done both in api-gateway and in search-service (because both are running an HTTP server).

If running Stellio from `docker-compose`, it can be configured in the environment section of both services:

```
      - SERVER_MAX-HTTP-REQUEST-HEADER-SIZE=100KB
```

If running Stellio in Kubernetes, it can be configured in the deployments:

```
- name: SERVER_MAX_HTTP_REQUEST_HEADER_SIZE
  value: 512KB
```

## Increase the default and maximum limit for pagination

Stellio has a default pagination limit of 30 and a maximum of 100. 

These values can be changed by configuring the `application.pagination.limit-default` and `application.pagination.limit-max` properties in `shared.properties` file.

If running Stellio from `docker-compose`, it can be configured in the environment section of the target service (`search-service` or `subscription-service`) : 

```
      - APPLICATION_PAGINATION_LIMIT_DEFAULT=30
      - APPLICATION_PAGINATION_LIMIT_MAX=100
```




## Change the log level of a library / namespace

Add a new environment for the target namespace. For instance, adding:

```
      - LOGGING_LEVEL_IO_R2DBC_POSTGRESQL_QUERY=DEBUG
```

Will set the DEBUG level for the `io.r2dbc.postgresql.QUERY` namespace.

For this change to take effet, the Docker container has to be re-created.
