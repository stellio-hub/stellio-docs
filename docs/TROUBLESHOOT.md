# TROUBLESHOOT


## Common Stellio API mistakes 

### Sending a single entity to a entityOperations endpoint

```json
{
  "type": "https://uri.etsi.org/ngsi-ld/errors/InvalidRequest",
  "status": 400,
  "title": "Cannot deserialize value of type `java.util.ArrayList<java.util.Map<java.lang.Object,java.lang.Object>>` from Object value (token `JsonToken.START_OBJECT`)\n at [Source: REDACTED (`StreamReadFeature.INCLUDE_SOURCE_IN_LOCATION` disabled); line: 1, column: 1]",
  "instance": "/ngsi-ld/v1/entityOperations/**"
}
```
The endpoint behind `/ngsi-ld/v1/entityOperations` only accepts a list of entities. 

You can solve it by sending a list of one entity

```json
[
  {
    "id": "my:id",
    "type": "Example",
    "property1": ...
  }
]
```
### Other common mistakes

We are looking to document other common mistakes.
If you're aware of any, feel free to let us know or contribute at https://github.com/stellio-hub/stellio-docs
