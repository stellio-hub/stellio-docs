# Upgrading to 2.2.0

This note describes the necessary steps to upgrade to Stellio 2.2.0

## Update existing subscriptions

Previous versions of Stellio were not correctly handling the JSON-LD context of the subscriptions:
- entities included in notifications were compacted using the context of the matched entities, and not the context used at the time of the subscription
- the `watchedAttributes` member of the subscriptions was not stored under its expanded form, but as received in the subscription (so typically under its compacted form)

These 2 points are fixed in Stellio 2.2.0, but existing subscriptions may have to be updated to set the correct JSON-LD context and expanded version of `watchedAttributes`. This can be done directly in the database by a similar SQL script:

```sql
update subscription
set contexts = array['https://my.context.org/business-context.jsonld']
-- applied to all or only some of the existing subscriptions
-- where id like 'urn:ngsi-ld:Subscription:BusinessContext:%';

update subscription
set watched_attributes = 'https://my.ontology.org/domain#term'
-- applied to all or only some of the existing subscriptions
-- where id like 'urn:ngsi-ld:Subscription:BusinessContext:%';
```
