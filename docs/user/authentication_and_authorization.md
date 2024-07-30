# Authentication and authorization

## Pre-requisites

For all the API operations described in this page, the [EGM's authorization context](https://easy-global-market.github.io/ngsild-api-data-models/authorization/jsonld-contexts/authorization.jsonld) has to be included in every operation.
These operations respect the rules of the section 6.3.5 of the NGSI-LD specification ("JSON-LD @context resolution"), with the exception that the [EGM's compound authorization context](https://easy-global-market.github.io/ngsild-api-data-models/authorization/jsonld-contexts/authorization-compound.jsonld) is considered as the default context for all operations found here.

## Terminology

- `user`: an entity that is authenticated and can access Stellio. It can be a physical user or a client with a service account enabled.

## Specific access policy

Stellio supports providing a specific property, called `specificAccessPolicy`, to allow any authenticated user to read or update an entity.

This property can be set at entity creation time and it can later be updated by any user who has admin rights on the entity. In case an unauthorized user tries to modify it, a 403 HTTP error is returned.

It currently supports the following two values (more may be added in the future):

- `AUTH_READ`: any authenticated user can read the entity
- `AUTH_WRITE`: any authenticated user can update the entity (it of course implies the `AUTH_READ` right)

### Update the specific access policy of an entity

This endpoint allows an user to update the specific access policy set on a entity.

It is available under `/ngsi-ld/v1/entityAccessControl/{entityId}/attrs/specificAccessPolicy` and can be called with a `POST` request.

The expected request body is an NGSI-LD Attribute Fragment:

```JSON
{
    "type": "Property",
    "value": "AUTH_READ"
}
```

### Remove the specific access policy from an entity

This endpoint allows an user to remove the specific access policy set on a entity.

It is available under `/ngsi-ld/v1/entityAccessControl/{entityId}/attrs/specificAccessPolicy` and can be called with a `DELETE` request.

## Endpoints for rights management

Stellio exposes endpoints that help in managing rights on entities inside the context broker.

### Get authorized entites for the currently authenticated user 

This endpoint allows an user to get the rights it has on entities.

It is available under `/ngsi-ld/v1/entityAccessControl/entities` and can be called with a `GET` request.

The following request parameters are supported: 

* `attrs`: restrict returned entities to the ones with a specific right. Only `canRead`, `canWrite`, `canAdmin` and `isOwner` are accepted. A list is accepted (e.g, `attrs=canRead,canWrite`). This request parameter has no effect when user has the _stellio-admin_ role
* `type`: restrict returned entities to given entity types
* `id`: restrict returned entities to given ids (comma separated list of strings)

There are several possible answers:

* If user is not admin nor owner of the entity but it has a right on it (`canRead` or `canWrite`), the response body will be under this form:  

```JSON
[
    { 
        "id": "urn:ngsi-ld:Entity:01",
        "type": "Entity",
        "right": { 
            "type": "Property", 
            "value": "canWrite" 
        }, 
        "specificAccessPolicy": { 
            "type": "Property", 
            "value": "AUTH_READ" 
        }, 
        "@context": "https://easy-global-market.github.io/ngsild-api-data-models/authorization/jsonld-contexts/authorization-compound.jsonld"
    },
    {
        ...
    }
]
```

* If user is admin or owner of the entity or has the _stellio-admin_ role, the response body will be under this form: 

```json
[
    { 
        "id": "urn:ngsi-ld:Entity:01",
        "type": "Entity",
        "right": { 
            "type": "Property", 
            "value": "canAdmin" 
        }, 
        "specificAccessPolicy": { 
            "type": "Property", 
            "value": "AUTH_READ" 
        }, 
        "canRead": [  
            {  
                "type": "Relationship",  
                "object": "urn:ngsi-ld:User:2194588E-D3CE-47F9-B060-B77DB6EAAAD8",
                "datasetId": "urn:ngsi-ld:Dataset:2194588E-D3CE-47F9-B060-B77DB6EAAAD8",
                "subjectInfo": {
                    "type": "Property",
                    "value": {
                        "kind": "User",
                        "username": "stellio-user"
                    }
                }
            }, 
            {  
                "type": "Relationship",
                "object": "urn:ngsi-ld:Client:D7A09461-4FD1-4B96-A15E-1DCABD11FE04",
                "datasetId": "urn:ngsi-ld:Dataset:D7A09461-4FD1-4B96-A15E-1DCABD11FE04",
                "subjectInfo": {
                    "type": "Property",
                    "value": {
                        "kind": "Client",
                        "clientId": "client-id"
                    }
                }
            }, 
            {  
                "type": "Relationship",  
                "object": "urn:ngsi-ld:Group:5AD29EF5-5427-46DA-9573-7CA03F842701",
                "datasetId": "urn:ngsi-ld:Dataset:5AD29EF5-5427-46DA-9573-7CA03F842701",
                "subjectInfo": {
                    "type": "Property",
                    "value": {
                        "kind": "Group",
                        "name": "Stellio Team"
                    }
                }
            } 
        ], 
        "canWrite": [ 
            … 
        ], 
        "canAdmin": [ 
            … 
        ],
        "isOwner": {
            // only if user is not the owner of the entity, in which case the value of the "right" property will be "isOwner"
        }
        "@context": "https://easy-global-market.github.io/ngsild-api-data-models/authorization/jsonld-contexts/authorization-compound.jsonld"
    },
    {
        ...
    }
]
```

The body also contains the other users who have a right on the entity.

* If authentication is not enabled, a 204 (No content) response is returned. 

### Get groups the currently authenticated user belongs to 

This endpoint allows an user to get the groups it belongs to.

It is available under `/ngsi-ld/v1/entityAccessControl/groups` and can be called with a `GET` request.

There are several possible answers:

* If user is not _stellio-admin_: 

```json
[
    { 
        "id": "urn:ngsi-ld:Group:01",
        "type": "Group",
        "name": { 
            "type": "Property", 
            "value": "EGM" 
        }, 
        "@context": "https://easy-global-market.github.io/ngsild-api-data-models/authorization/jsonld-contexts/authorization-compound.jsonld"
    }
]
```

* If user is _stellio-admin_, all groups are returned:

```json
[
    { 
        "id": "urn:ngsi-ld:Group:01",
        "type": "Group",
        "name": { 
            "type": "Property", 
            "value": "EGM" 
        },
        "isMemberOf": {
            "type": "Property", 
            "value": true
        },
        "@context": "https://easy-global-market.github.io/ngsild-api-data-models/authorization/jsonld-contexts/authorization-compound.jsonld"
    }
]
```

The body also contains membership information. 

* If authentication is not enabled, a 204 (No content) response is returned. 

### Get users

This endpoint allows an user with `stellio-admin` role to get a list of all users

It is available under `/ngsi-ld/v1/entityAccessControl/users` and can be called with a `GET` request.

* If user is not _stellio-admin_, an error 403 is returned
* If user is _stellio-admin_, all users are returned (`givenName` and `familyName` are optional fields that may not be part of the response):

```json
[
    {
        "id": "urn:ngsi-ld:User:01",
        "type": "User",
        "username": {
            "type": "Property",
            "value": "username"
        },
        "givenName": {
            "type": "Property",
            "value": "givenname"
        },
        "familyName": {
            "type": "Property",
            "value": "familyname"
        },
        "subjectInfo": {
            "type": "Property",
            "value": {
                "gender": "Male",
                "city": "Nantes"
            }
        },
        "@context": "https://easy-global-market.github.io/ngsild-api-data-models/authorization/jsonld-contexts/authorization-compound.jsonld"
    }
]
```

The `subjectInfo` property is present only if other user attributes are known to the system.

* If authentication is not enabled, a 204 (No content) response is returned.

### Add rights on entities for a Stellio User

This endpoint allows an user to give rights on entities it is admin or owner of.

The rights will be given to another user or group. It is necessary to provide the `sub` of the target user or group (and not the pseudo entity id of an user or group).

It is available under `/ngsi-ld/v1/entityAccessControl/{sub}/attrs` and can be called with a `POST` request.

The expected request body is a JSON object containing NGSI-LD Relationships:

```json
{
  "canRead": [ 
    { 
      "type": "Relationship", 
      "object": "entityId1",
      "datasetId": "urn:ngsi-ld:Dataset:01"
    },
    { 
      "type": "Relationship", 
      "object": "entityId2",
      "datasetId": "urn:ngsi-ld:Dataset:02"
    }
  ],
  "canWrite": [
    …
  ],
  "canAdmin": [
    …
  ],
}
```

Only `canRead` and `canWrite` and `canAdmin` attributes are allowed by this operation.

It returns:

- 204 if all operations succeeded
- 207 if some failed (typically insufficient rights to perform the operation)

### Remove rights on an entity for a Stellio User

This endpoint allows an user to remove rights of an user (or group) on an entity it is admin or owner of.

As when adding rights, the user or group whose the right is to be removed is identified by its `sub` (and not by the pseudo entity id of an user or group). It is necessary to provide the `entity_id` of the target entity. Removing the ownership of an user on an entity is not permitted.

It is available under `/ngsi-ld/v1/entityAccessControl/{sub}/attrs/{entityId}` and can be called with a `DELETE` request.

It returns 204 if the operation succeeded.


### Delete entities owned by Stellio User if said user is deleted

Stellio allows the deletion of all entities owned by a user, if said user is deleted. 

This feature is only active if `search.on-owner-delete-cascade-entities` is set to `true` in Search Service `application.properties`.