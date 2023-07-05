# Multitenancy

## Design

In Stellio, each tenant:

* Is defined by an URI (as mandated by the NGSI-LD specification)
* Maps to a specific schema in the database
* Binds to a specific realm in Keycloak (if authentication is enabled)

The detailed behavior is defined in the NGSI-LD specification, section 4.14 - Supporting multiple tenants.

## Declaration

Tenants are defined in the `shared.properties` file in the `shared` module.

```
# each tenant maps to a different KC realm (if authentication is enabled) and DB schema
# a tenant declaration is composed of a (tenant) URI, an OIDC issuer URL and a DB schema
application.tenants[0].uri = urn:ngsi-ld:tenant:default
application.tenants[0].issuer = https://sso.eglobalmark.com/auth/realms/stellio
application.tenants[0].dbSchema = public
application.tenants[1].uri = urn:ngsi-ld:tenant:stellio-dev
application.tenants[1].issuer = https://sso.eglobalmark.com/auth/realms/stellio-dev
application.tenants[1].dbSchema = stellio-dev
```

Default tenant must always be declared with the `urn:ngsi-ld:tenant:default` URI (but, as specified by the NGSI-LD API specification, it does not have to be declared in the HTTP requests and is used if no tenant is specified in a request).

To add a tenant:

* Declare it in the `shared.properties` file (as shown above)
* If authentication is enabled, create the realm in Keycloak
* Restart the context broker
* The DB schema is automatically created when the context broker restarts

When running with the Docker images and using the docker-compose configuration, it is possible to declare the tenants in the environment section of the search and subscription services:

```yaml
  search-service:
    environment:
      - APPLICATION_TENANTS_0_ISSUER=${APPLICATION_TENANTS_0_ISSUER}
      - APPLICATION_TENANTS_0_URI=${APPLICATION_TENANTS_0_URI}
      - APPLICATION_TENANTS_0_DBSCHEMA=${APPLICATION_TENANTS_0_DBSCHEMA}
```

Where the 3 environment variables can be declared in the `.env` file:

```shell
APPLICATION_TENANTS_0_ISSUER=https://sso.eglobalmark.com/auth/realms/stellio
APPLICATION_TENANTS_0_URI=urn:ngsi-ld:tenant:default
APPLICATION_TENANTS_0_DBSCHEMA=public
```

An example configuration can be seen in the docker-compose file available in the GitHub repository.

Please note that the associated Keycloak realm must exist when the application starts since the context broker will try to read its configuration.
