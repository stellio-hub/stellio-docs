# Upgrading to 1.5.0

This note describes the necessary steps to upgrade to Stellio 1.5.0

## Upgrade Docker Compose

Stellio now uses the Compose specification version 3.9. To be able to run the Docker Compose file provided with Stellio, you need to have Docker Compose 1.29+, be sure to upgrade to this version before launching Stellio 1.5.0.

On Debian / Ubuntu systems, you can install it with the following instructions:

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## Synchronization of access rights

In order to propagate access rights defined in entity service, an action has to be launched manually by an account having the `stellio-admin` role.

Before doing this, it is recommended, but not necessary, to launch manually some Cypher scripts to clean up potentially invalid data:

* Ensure all dates on relationships nodes are in the correct format:

```
docker exec -it neo4j cypher-shell -u neo4j -p {neo4j_user_password} -d stellio "MATCH (r:Relationship) SET r.createdAt = datetime(r.createdAt) SET r.modifiedAt = datetime(r.modifiedAt);"
```

* Ensure there are no duplicate entities in the graph

```
docker exec -it neo4j cypher-shell -u neo4j -p {neo4j_user_password} -d stellio "MATCH (e:Entity) WITH e.id AS id, COLLECT(e) AS nodelist, COUNT(*) AS count WHERE count > 1 CALL apoc.refactor.mergeNodes(nodelist) YIELD node RETURN node;"
```

* Ensure all relationships have the `objectId` property correctly filled:

```
docker exec -it neo4j cypher-shell -u neo4j -p {neo4j_user_password} -d stellio "MATCH (r:Attribute:Relationship)-[]->(e:Entity) SET r.objectId = e.id;"
```

* Launch the synchronization action (to be done with an account having the `stellio-admin` role)

```
# Get an acces token first
export TOKEN=$(http --form POST https://sso.eglobalmark.com/auth/realms/{realm_name}/protocol/openid-connect/token client_id={client_id} client_secret={client_secret} grant_type=client_credentials | jq -r .access_token)

# Call the synchronization action
http POST https://{your_context_broker_domain_name}/ngsi-ld/v1/entityAccessControl/sync Authorization:"Bearer $TOKEN"
```

Check the logs of the entity and search services to be sure the synchronization has worked as expected. If not, feel free [to raise an issue](https://github.com/stellio-hub/stellio-context-broker/issues/new/choose) in the Stellio repository.
