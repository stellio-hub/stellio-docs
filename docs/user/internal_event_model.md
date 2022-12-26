# Internal event model

As explained in the [Getting Started page](../quick_start_guide.md), Stellio internally uses a Kafka message broker to decouple communication between the 2 main micro-services.

This communication is based on an event model inspired by the NGSI-LD API and does its best to follow the same design principles.

Currently, the following events are flowing inside the platform:

- Operations done on entities through the NGSI-LD API (including batch operations)
    - Used by the subscription service to trigger notification if there are some matching subscriptions
- Subscriptions and notifications
    - History of notifications raised for each subscription is stored by the search service
- Identity and access management events sent by Keycloak
    - Listened by the micro-services to provision users, groups and clients, as well as their global roles and groups memberships
- Telemetry events received from sensors

One core principle of the event model is that an event is atomic, i.e., it contains one and only one operation. So, for instance, if 5 entities are created in a batch operation, 5 events will be propagated to Kafka. In the same way, if 2 attributes are added to an entity, 2 events will be propagated.

## Types and structure of events

Here is the list of supported events, accompanied by a sample payload:

- Entity creation


```json
{
    "entityId": "urn:ngsi-ld:Vehicle:A4567",
    "entityType": "Vehicle",
    "operationPayload": "{\"id\": \"urn:ngsi-ld:Vehicle:A4567\", \"type\": \"Vehicle\", \"brandName\": { \"type\": \"Property\", \"value\": \"Mercedes\"}}",
    "contexts": [
        "https://some.host/my-context.jsonld",
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.3.jsonld"
    ],
    "operationType": "ENTITY_CREATE"
}
```

- Entity update

Same payload as when creating an entity, but with an `OPERATION_TYPE` set to `ENTITY_UPDATE`.

- Entity replace

Same payload as when creating an entity, but with an `OPERATION_TYPE` set to `ENTITY_REPLACE`.

- Entity deletion

```json
{
    "entityId": "urn:ngsi-ld:Vehicle:A4567",
    "entityType": "Vehicle",
    "contexts": [
        "https://some.host/my-context.jsonld",
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.3.jsonld"
    ],
    "operationType": "ENTITY_DELETE"
}
```

- Attribute append

```json
{
    "entityId": "urn:ngsi-ld:Vehicle:A4567",
    "entityType": "Vehicle",
    "attributeName": "speed",
    "datasetId": "urn:ngsi-ld:Dataset:GPS",
    "overwrite": true,
    "operationPayload": "{ \"value\": 76, \"unitCode\": \"KMH\", \"observedAt\": \"2021-10-26T22:35:52.98601Z\", \"datasetId\": \"urn:ngsi-ld:Dataset:GPS\" }",
    "updatedEntity": "(entity payload after the append operation)",
    "contexts": [
        "https://some.host/my-context.jsonld",
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.3.jsonld"
    ],
    "operationType": "ATTRIBUTE_APPEND"
}
```

- Attribute replace

Same payload as when appending an attribute, but with an `OPERATION_TYPE` set to `ATTRIBUTE_REPLACE` and no `overwrite` attribute.

- Attribute update

```json
{
    "entityId": "urn:ngsi-ld:Vehicle:A4567",
    "entityType": "Vehicle",
    "attributeName": "speed",
    "datasetId": "urn:ngsi-ld:Dataset:GPS",
    "operationPayload": "{ \"value\":60, \"observedAt\": \"2021-10-26T23:35:52.98601Z\" }",
    "updatedEntity": "(entity payload after the update operation)",
    "contexts": [
        "https://some.host/my-context.jsonld",
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.3.jsonld"
    ],
    "operationType": "ATTRIBUTE_UPDATE"
}
```

- Attribute deletion

```json
{
    "entityId": "urn:ngsi-ld:Vehicle:A4567",
    "entityType": "Vehicle",
    "attributeName": "speed",
    "datasetId": "urn:ngsi-ld:Dataset:GPS",
    "updatedEntity": "(entity payload after the delete operation)",
    "contexts": [
        "https://some.host/my-context.jsonld",
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.3.jsonld"
    ],
    "operationType": "ATTRIBUTE_DELETE"
}
```

- Full attribute deletion

```json
{
    "entityId": "urn:ngsi-ld:Vehicle:A4567",
    "entityType": "Vehicle",
    "attributeName": "speed",
    "updatedEntity": "(entity payload after the delete operation)",
    "contexts": [
        "https://some.host/my-context.jsonld",
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.3.jsonld"
    ],
    "operationType": "ATTRIBUTE_DELETE_ALL_INSTANCES"
}
```
