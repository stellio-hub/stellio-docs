# Authentication and authorization

## Pre-requisites

For all the API operations described in this page, the [EGM's authorization context](https://easy-global-market.github.io/ngsild-api-data-models/authorization/jsonld-contexts/authorization.jsonld) has to be included in every operation.
These operations respect the rules of the section 6.3.5 of the NGSI-LD specification ("JSON-LD @context resolution"), with the exception that the [EGM's compound authorization context](https://easy-global-market.github.io/ngsild-api-data-models/authorization/jsonld-contexts/authorization-compound.jsonld) is considered as the default context for all operations found here.

## Terminology

- `user`: an entity that is authenticated and can access Stellio. It can be a physical user or a client with a service account enabled.

## Specific access policy

Stellio supports providing a specific property, called `specificAccessPolicy`, to allow any authenticated user to read or update an entity.

This property can be set at entity creation time.

It currently supports the following two values (more may be added in the future):

- `read`: any authenticated user can read the entity
- `write`: any authenticated user can update the entity (it of course implies the `read` right)

## Endpoints for permission management

Stellio exposes endpoints that help in managing permissions on entities (and soon on entities types and scopes) inside the context broker.

The permissions are represented by a `Permission` data type.

### Permission data type

```json
{
  "id" : "urn:permission:uuid",
  "type" : "Permission",
  "target" : {
    "id" : "my:entity:id",
    "types" : ["CompactedType1", "CompactedType2"],
    "scopes" :  "my/scope"
  },
  "assignee" : "subjectID", 
  "assigner": "subjectID",
  "action":  "write",
  "@context": "https://easy-global-market.github.io/ngsild-api-data-models/authorization/jsonld-contexts/authorization-compound.jsonld"
}
```

The properties are based on ODRL Permission class but do not respect the entire ODRL model.

The following properties are used:
- "id" : a unique identifier of the permission (should be a URI)
- "type" : should always be "Permission"
- "target" :
  - "id" :
    - an id of an existing entity
    - the permission gives right to the entity with the specified id  
    - can only be specified if types and scopes are null
  - "types" :
    - a type or a list of types
    - the permission gives right to entities having at least one of specified types.
    - if null the permission is considered for every types
    - can only be specified if id is null
  - "scopes" :
    - a scope or a list of scopes
    - the permission gives right to entities having at least one of specified scopes.
    - if null the permission is considered for every scopes
    - you can specify '@none' for the default scopes
    - can only be specified if id is null
- "assignee" : id of the subject (group or user) getting the permission. If null the permission is considered to be for everyone
- "assigner" : id of the creator
- "action"   : can be "read", "write", "admin" and "own" ("own" is created by the broker at entity creation, you can't add, modify or delete "own" permissions)

A permission targeting types and scopes gives right to entities having a matching type **AND** a matching scope 

### Permission provision

To be able to create, update or delete a permission you need to be administrator of the target of the permission.
You can't combine multiple admin permission when provisioning permissions. (ie: if you have an admin permission on type A and one on type B you can't create a permission on type [A,B])

#### Special business rules
- modifying or creating a permission with the "own" action is forbidden
- assigning the "admin" action to everyone is forbidden (ie: assignee = null)
- creating a permission with the same assignee and target as an existing permission result in a conflict


#### Create a permission

-  POST /auth/permissions

```json
{
  "id" : "urn:permission:my:id",
  "type" : "Permission",
  "target" : {
    "id" : "my:entity:id"
  },
  "assignee" : "subjectID", 
  "action":  "write",
  "@context": "https://easy-global-market.github.io/ngsild-api-data-models/authorization/jsonld-contexts/authorization-compound.jsonld"
}
```

#### Update a permission

-  PATCH /auth/permissions/{id}

Note: modifying a permission will make you the new assigner of this permission. You can’t put someone else than you as an assigner.

#### Delete a permission

-  DELETE /auth/permissions/{id}

### Permission consumption

You can only access permissions that are assigned to you or permissions targeting entities you have at least admin right on

#### Retrieve a permission

-  GET /auth/permissions/{id}

```json
{
  "id" : "urn:permission:my:id",
  "type" : "Permission",
  "target" : {
    "id" : "my:entity:id"
  },
  "assignee" : "subjectID", 
  "action":  "write",
  "@context": "https://easy-global-market.github.io/ngsild-api-data-models/authorization/jsonld-contexts/authorization-compound.jsonld"
}
```

You can ask to retrieve the entity and the assignee information in the same request by adding `details=true` in the query parameters.
In addition you can filter what property of the target entity you want to retrieve by adding `detailsPick=attr1`

The result will look like this:

```json
{
  "action" : "read",
  "assignee" : {
    "subjectId":"55e64faf-4bda-41cc-98b0-195874cefd29",
    "subjectType":"GROUP",
    "subjectInfo":{"name":"Stellio Team"}
  },
  "assigner" : {
    "subjectId":"91e64bcf-c6da-zt1d-ar70-164f74ce5d75",
    "subjectType":"USER",
    "subjectInfo": {
      "username": "jeanne@dupont.io",
      "givenName": "Jeanne",
      "familyName": "Dupont"
    }
  },
  "target" : {
    "id" : "my:id",
    "type" : "BeeHive",
    "attr1": {
      "type": "Property",
      "value": "some value 1"
    }
  }
}
```

#### Query the permissions that you can administer

-  GET /auth/permissions

You can filter the requested permissions with the following query parameters:

 - targetId=unr:id:1,urn:id:2 to get the permissions targeting entities with id urn:id:1 and urn:id:2

 - assignee=my:assignee to get the permissions assigned to “my:assignee”

 - assigner=my:assigner to get the permissions created by “my:assigner”

 - action=read to get the permissions giving the right to read

 - targetType=MyType to get the permissions targeting entities matching the corresponding type (note: the field support complex entity type selection as defined in section 4.17 of the specification)

You can ask to retrieve the entity and the assignee information in the same request by adding `details=true` in the query parameters.
In addition you can filter what property of the target entity you want to retrieve by adding `detailsPick=attr1`

Other parameter:
 - sysAttrs=true  include createdAt and modifiedAt properties

This endpoint supports the usual pagination parameters. They are functionally identical to the query entities operation :
 - count
 - limit
 - offset

#### Query the permissions assigned to you

-  GET /auth/permissions/assigned

This endpoint supports the same parameters as the previous endpoint (GET /auth/permissions).

## Get groups the currently authenticated user belongs to 

This endpoint allows an user to get the groups it belongs to.

It is available under `/ngsi-ld/v1/auth/subjects/groups` and can be called with a `GET` request.

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

## Get users

This endpoint allows an user with `stellio-admin` role to get a list of all users

It is available under `/ngsi-ld/v1/auth/subjects/users` and can be called with a `GET` request.

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

## Delete entities owned by Stellio User if said user is deleted

Stellio allows the deletion of all entities owned by a user, if said user is deleted. 

This feature can be activated by setting `search.on-owner-delete-cascade-entities` to `true` in Search Service  `application.properties` file.

If Stellio is running from `docker-compose`, `search.on-owner-delete-cascade-entities` must be set as an environment variable first in `.env` and set to `true`:

```
SEARCH_ON_OWNER_DELETE_CASCADE_ENTITIES = true
```

Then it can be added in the environment section of `search-service` :

```
SEARCH_ON_OWNER_DELETE_CASCADE_ENTITIES = ${ON_OWNER_DELETE_CASCADE_ENTITIES}
```