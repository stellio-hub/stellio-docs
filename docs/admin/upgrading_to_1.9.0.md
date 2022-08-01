# Upgrading to 1.9.0

This note describes changes made by upgrade to Stellio 1.9.0

## New endpoints for rights management

Stellio 1.9.0 brings new endpoints to visualize the rights an user has on entities but also to know the groups which he belongs to.

### Get authorized entites for the currently authenticated user 

The first endpoint allows to retrieve the rights for all the entities of the authenticated person.

To do this, the query should be a `GET` with as endpoint `/ngsi-ld/v1/entityAccessControl/entities`.

The following request parameters are supported: 

    -  `q`: to restrict returned entities to the ones with a specific right. Only `rCanRead` and `rCanWrite` and `rCanAdmin` are accepted. A list is accepted (e.g, `q=rCanRead;rCanWrite`). This request parameter does not work when user is stellio admin. 

    -  `type`: to restrict returned entities to a given entity type 

    - `options`: use `sysAttrs` value to get the system attributes.

Then there are several possible answers:

    1) If user is not admin of the  entity but it has a right on it (`rCanRead` or `rCanWrite`) the body will be under this form:  

    ```
    { 
        “id”:…, 
        “type”:…, 
        “datasetId”:…, 
        “right”: { 
            “type”: “Property”, 
            “value”: “rCanAdmin” 
        }, 
        “specificAccessPolicy”: { 
            “type”: “Property”, 
            “value”: “AUTH_READ” 
        }, 
        @context: […] 
    }
    ```
The body contains id, type, datasetId, right and specificAccessPolicy. 

    2) If user is admin of the returned entity the body will be under this form: 

    ```
    { 
        “id”:…, 
        “type”:…, 
        “datasetId”:…, 
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
        @context: […] 
    }
    ```
    The body also contains the users who have right on it.  

    3) If user use Stellio without authentification, a 204 (No content) response is returned. 


### Get groups the currently authenticated user belongs to 

This endpoint allows to retrieve the groups which we belongs to.

To do this, the query should be a `GET` with as endpoint `/ngsi-ld/v1/entityAccessControl/groups`.

Then there are several possible answers:

    1) If user is not admin : 

    ```
    { 
        “id”:…, 
        “type”:…, 
        “name”: { 
            “type”: “Property”, 
            “value”: “egm” 
        }, 
        context: […] 
    }
    ```
    The body contains id, type and name of the group. 

    2) If user is admin, all groups are returned:

    ```
    { 
        “id”:…, 
        “type”:…, 
        “name”: { 
            “type”: “Property”, 
            “value”: “egm” 
         }, 
        “isMemberOf”: { 
            “type”: “Property”, 
            “value”: “true” 
        }, 
        @context: […] 
    }
    ```
    The body contains id, type, name of the group and if user is member of this group. 

    3) If user use Stellio without authentification, a 204 (No content) response code is returned. 