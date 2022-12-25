# Authentication and authorization

## Specific access policy

Stellio supports providing a specific property, called `specificAccessPolicy` (defined in [EGM's authorization context](https://raw.githubusercontent.com/easy-global-market/ngsild-api-data-models/master/authorization/jsonld-contexts/authorization.jsonld)), to allow any authenticated user to read or update an entity.

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

## Application of rights in queries on entities

For `/entities` and `temporal/entities` queries, we generally have the id of the entity which is the subject of the request (except for search, that is dealt with separately). To be able to apply security checks, we need to know if the user has the right to perform this operation as an individual (this is just a check on the existence of a relationship of a given type(s)), as a member of a group (the same kind of check but with one level of indirection) or as a (OAuth2) client.

For `/entityOperations` queries, checks are applied on a per-entity basis. Trying to modify or delete an unauthorized entity is of course denied and is notified in the details of the response.

## Endpoints for rights management

Stellio exposes endpoints that help in managing rights inside the context broker.

### Get authorized entites for the currently authenticated user 

This endpoint allows an user to get the rights it has on entities.

It is available under `/ngsi-ld/v1/entityAccessControl/entities` and can be called with a `GET` request.

The following request parameters are supported: 

* `q`: restrict returned entities to the ones with a specific right. Only `rCanRead` and `rCanWrite` and `rCanAdmin` are accepted. A list is accepted (e.g, `q=rCanRead;rCanWrite`). This request parameter has no effect when user has the _stellio-admin_ role
* `type`: restrict returned entities to a given entity type
* `options`: use `sysAttrs` value to get the system attributes

There are several possible answers:

* If user is not admin of the entity but it has a right on it (`rCanRead` or `rCanWrite`), the response body will be under this form:  

```JSON
[
    { 
        “id”: "urn:ngsi-ld:Entity:01",
        “type”: "Entity",
        “datasetId”: "urn:ngsi-ld:Dataset:Entity:01",
        “right”: { 
            “type”: “Property”, 
            “value”: “rCanWrite” 
        }, 
        “specificAccessPolicy”: { 
            “type”: “Property”, 
            “value”: “AUTH_READ” 
        }, 
        @context: [ "https://my.context/context.jsonld" ] 
    },
    {
        ...
    }
]
```

* If user is admin of the entity, the response body will be under this form: 

```json
[
    { 
        “id”: "urn:ngsi-ld:Entity:01",
        “type”: "Entity",
        “datasetId”: "urn:ngsi-ld:Dataset:Entity:01",
        “right”: { 
            “type”: “Property”, 
            “value”: “rCanAdmin” 
        }, 
        “specificAccessPolicy”: { 
            “type”: “Property”, 
            “value”: “AUTH_READ” 
        }, 
        “rCanRead”: [  
            {  
                “type”: “Relationship”,  
                “object”: “urn:ngsi-ld:User:01”     
            }, 
            {  
                “type”: “Relationship”,  
                “object”: “urn:ngsi-ld:User:02”
            } 
        ], 
        “rCanWrite”: [ 
            … 
        ], 
        “rCanAdmin”: [ 
            … 
        ], 
        @context: [ "https://my.context/context.jsonld" ] 
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
        “id”: "urn:ngsi-ld:Group:01",
        “type”: "Group",
        “name”: { 
            “type”: “Property”, 
            “value”: "EGM" 
        }, 
        @context: [ "https://my.context/context.jsonld" ] 
    }
]
```

* If user is _stellio-admin_, all groups are returned:

```json
[
    { 
        “id”: "urn:ngsi-ld:Group:01",
        “type”: "Group",
        “name”: { 
            “type”: “Property”, 
            “value”: “EGM” 
        },
        “isMemberOf”: {
            “type”: “Property”, 
            “value”: true
        },
        @context: [ "https://my.context/context.jsonld" ] 
    }
]
```

The body also contains membership information. 

* If authentication is not enabled, a 204 (No content) response is returned. 

### Add rights on entities for a Stellio User

POST /ngsi-ld/v1/entityAccessControl/{subjectId}/attrs
Sample payload: 
{
  “rCanRead”: [ 
    { 
      “type”: “Relationship”, 
      “object”: “entityId1”,
      “datasetId”: “urn:ngsi-ld:Dataset:01”
    },
    { 
      “type”: “Relationship”, 
      “object”: “entityId2” 
      “datasetId”: “urn:ngsi-ld:Dataset:02”
    }
  ],
  “rCanWrite”: [
    …
  ],
  “rCanAdmin”: [
    …
  ],
}

Only "rCanRead" and "rCanWrite" and "rCanAdmin" attributes can be appended
Return: 204 if all operations succeeded, 207 if some failed (typically insufficient rights to perform the operation)

### Remove rights on an entity for a Stellio User

DELETE /ngsi-ld/v1/entityAccessControl/{subjectId}/attrs/{entityId}
Same principle than above but applied to delete rights a Stellio User has on a specific entity.
Return: 204 if the operation succeeded
