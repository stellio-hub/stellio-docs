# Upgrading to 2.6.0

This note describes the necessary steps to upgrade to Stellio 2.6.0

## Multitenancy

Starting from version 2.6.0, Stellio supports NGSI-LD Tenants. Please read the [multitenancy page](../user/multinenancy.md) for an explanation of the design and the way to configure and use them.

## Renaming of properties

In the subscription service, two properties have been renamed:

- `application.entity.service-url` has been renamed to `subscription.entity-service-url`
- `application.stellio-url` has been renamed to `subscription.stellio-url`
