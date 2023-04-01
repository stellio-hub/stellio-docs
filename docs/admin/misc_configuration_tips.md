# Misc configuration tips

## Increase the max allowed size for headers

If sending HTTP requests with headers having a large size, you may have to increate the max allowed size (which is by default set to 8KB).

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
