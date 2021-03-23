# Authentication and authorization

## Specific access policy

Stellio supports providing a specific property, called `specificAccessPolicy` (defined in [EGM's cross-context]("https://github.com/easy-global-market/ngsild-api-data-models/blob/master/shared-jsonld-contexts/egm.jsonld#L34")), to allow any authenticated user to read or update the entity.

This property can be set at entity creation time and it can later be updated by any user who has admin rights on the entity. In case an unauthorized user tries to modify it, a 403 HTTP error is returned.

It currently supports the following two values (more may be added in the future):

- `AUTH_READ`: any authenticated user can read the entity
- `AUTH_WRITE`: any authenticated user can update the entity (it of course implies the `AUTH_READ` right)
