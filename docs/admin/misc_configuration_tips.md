# Misc configuration tips

## Increase the max allowed size for headers

If sending HTTP requests with headers having a large size, you may have to increate the max allowed size (which is by default set to 8KB).

It may be done by configuring the `server.max-http-request-header-size` property. It has to be done both in api-gateway and in search-service (because both are running an HTTP server).

If running Stellio from `docker-compose`, it can be configured in the environment section of both services:

```yaml
    environment:
      - SERVER_MAX-HTTP-REQUEST-HEADER-SIZE=100KB
```

If running Stellio in Kubernetes, it can be configured in the deployments:

```yaml
- name: SERVER_MAX_HTTP_REQUEST_HEADER_SIZE
  value: 512KB
```

## Change the log level of a library / namespace

Add a new environment for the target namespace. For instance, adding:

```yaml
    environment:
      - LOGGING_LEVEL_IO_R2DBC_POSTGRESQL_QUERY=DEBUG
```

Will set the DEBUG level for the `io.r2dbc.postgresql.QUERY` namespace.

For this change to take effet, the Docker container has to be re-created.

## Change the JVM parameters in a container

Starting from Stellio 2.11.0, Docker images built for the Stellio services do not have any default JVM settings (until version 2.11.0, they had default memory settings for the heap size, but the JVM is able to set [sensible default settings](https://learn.microsoft.com/en-us/azure/developer/java/containers/overview#understand-jvm-default-ergonomics) so they have been removed).

If you want to add some specific JVM settings, you can do so by using the `JDK_JAVA_OPTIONS` environment variable. For instance, if you want to set the heap size memory:

```yaml
    environment:
      - JDK_JAVA_OPTIONS=-Xms1024m -Xmx2048m
```
