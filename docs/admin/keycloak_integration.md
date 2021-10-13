# Keycloak integration

Stellio can be used with the [Keycloak](https://www.keycloak.org) IAM solution.

Installation and configuration of Keycloak is not described here, as it is well documented on the Keycloak site.

Summary of steps to be performed to integrate Stellio with Keycloak:
- Create a realm in Keycloak (if not yet done while setting it up)
- Create the builtin roles known and used by Stellio
- Configure Keycloak to propagate user, group and client events to Stellio
- Activate and configure authentication in Stellio

## Create a realm

Follow instructions here on how to create a new realm in Keycloak: https://www.keycloak.org/docs/latest/server_admin/index.html#_create-realm

## Create the builtin roles

Stellio will natively interpret two specific Realm roles [that must first be created in Keycloak](https://www.keycloak.org/docs/latest/server_admin/index.html#realm-roles):
- stellio-creator: it gives an user the right to create entities in the context broker
- stellio-admin: it gives an user the administrator rights in the context broker

## Configure Keycloak event listener

In the Keycloak Docker configuration, fill the Kafka configuration:

```
      - KAFKA_BOOTSTRAP_SERVERS=stellio/10.5.1.207:29092
      - KAFKA_KEY_SERIALIZER_CLASS=org.apache.kafka.common.serialization.StringSerializer
      - KAFKA_VALUE_SERIALIZER_CLASS=org.apache.kafka.common.serialization.StringSerializer
      - KAFKA_ACKS=all
      - KAFKA_DELIVERY_TIMEOUT_MS=3000
      - KAFKA_REQUEST_TIMEOUT_MS=2000
      - KAFKA_LINGER_MS=1
      - KAFKA_BATCH_SIZE=16384
      - KAFKA_BUFFER_MEMORY=33554432
```

Then, in the admin console, select your realm and go to the Events section. Switch to the Config tab and add stellioEventListener in the list of events listeners.

## Configure authentication in Stellio

Finally, on Stellio side, configure the Keycloak URLs in entity, search and subscription:

```yaml
  entity-service:
    environment:
      - SPRING_SECURITY_OAUTH2_RESOURCESERVER_JWT_ISSUER-URI=${SPRING_SECURITY_OAUTH2_RESOURCESERVER_JWT_ISSUER_URI}
      - SPRING_SECURITY_OAUTH2_RESOURCESERVER_JWT_JWK-SET-URI=${SPRING_SECURITY_OAUTH2_RESOURCESERVER_JWT_JWK_SET_URI}
```

Values can be like:
- https://sso.yourcompany.com/auth/realms/{realm_name}
- https://sso.yourcompany.com/auth/realms/{realm_name}/protocol/openid-connect/certs

Notes:
- Depending on your deployment config, you may add SSL certificates authentication between Keycloak and Kafka
