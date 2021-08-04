# Upgrading to 1.0.0

This note describes the necessary steps to upgrade to Stellio 1.0.0

## Upgrade to Timescale 2.3 / PostgreSQL 13

* Prepare the environment variables

```
export POSTGRES_PASSWORD=postgres_user_password
```

* Backup the search and subscription databases

```
docker exec postgres /bin/bash -c "export PGPASSWORD=$POSTGRES_PASSWORD && /usr/local/bin/pg_dump -Fc -U postgres stellio_search" | gzip -9 > /tmp/postgres_search.gz
docker exec postgres /bin/bash -c "export PGPASSWORD=$POSTGRES_PASSWORD && /usr/local/bin/pg_dump -Fc -U postgres stellio_subscription" | gzip -9 > /tmp/postgres_subscription.gz
```

* Stop the current running instance

```
docker-compose stop postgres
docker rm stellio-postgres
```

* Get and run the Timescale 2.3.0-pg11 image

```
docker pull timescale/timescaledb:2.3.0-pg11
docker run -v stellio-postgres-storage:/var/lib/postgresql/data -d --name timescaledb -p 5432:5432 timescale/timescaledb:2.3.0-pg11
```

* Upgrade Timescale extension in the two databases

```
docker exec -it postgres psql -U postgres -X -c "ALTER EXTENSION timescaledb UPDATE;"
docker exec -it postgres psql -U postgres -X -c "ALTER EXTENSION timescaledb UPDATE;" template1
docker exec -it postgres psql -U postgres -X -c "ALTER EXTENSION timescaledb UPDATE;" stellio_search
docker exec -it postgres psql -U postgres -X -c "ALTER EXTENSION timescaledb UPDATE;" stellio_subscription
```

* Backup again the two databases

```
docker exec postgres /bin/bash -c "export PGPASSWORD=$POSTGRES_PASSWORD && /usr/local/bin/pg_dump -Fc -U postgres stellio_search" | gzip -9 > /tmp/postgres_search_before_pg13.gz
docker exec postgres /bin/bash -c "export PGPASSWORD=$POSTGRES_PASSWORD && /usr/local/bin/pg_dump -Fc -U postgres stellio_subscription" | gzip -9 > /tmp/postgres_subscription_before_pg13.gz
```

* Get and run the Timescale 2.3.0-pg13 image (without any previous data)

```
docker volume rm stellio-postgres-storage
docker volume create stellio-postgres-storage
docker run -v stellio-postgres-storage:/var/lib/postgresql/data -d --name timescaledb -p 5432:5432 stellio/stellio-timescale-postgis:2.3.0-pg13
```

* Restore the backups made at step 4

Copy the dumps and connect to the Timescale container:

```
docker cp /tmp/postgres_search_before_pg13.gz stellio-postgres:/tmp/.
docker cp /tmp/postgres_subscription_before_pg13.gz stellio-postgres:/tmp/.
docker exec -it stellio-postgres bash
```

In the container, restore the dumps:

```
gunzip /tmp/postgres_search_before_pg13.gz
gunzip /tmp/postgres_subscription_before_pg13.gz

su - postgres
psql

\c stellio_search
CREATE EXTENSION IF NOT EXISTS timescaledb;
SELECT timescaledb_pre_restore();
\! pg_restore -Fc -d stellio_search /tmp/postgres_search_before_pg13
SELECT timescaledb_post_restore();

\c stellio_subscription
CREATE EXTENSION IF NOT EXISTS timescaledb;
SELECT timescaledb_pre_restore();
\! pg_restore -Fc -d stellio_subscription /tmp/postgres_subscription_before_pg13
SELECT timescaledb_post_restore();
```

Exit the container and eventually remove the dumps.

References:

- [Updating TimescaleDB to 2.0](https://docs.timescale.com/timescaledb/latest/how-to-guides/update-timescaledb/update-timescaledb-2)
- [Updating TimescaleDB versions](https://docs.timescale.com/timescaledb/latest/how-to-guides/update-timescaledb)
- [Updating a TimescaleDB Docker installation](https://docs.timescale.com/timescaledb/latest/how-to-guides/update-timescaledb/updating-docker)
- [Logical backups with pg_dump and pg_restore](https://docs.timescale.com/timescaledb/latest/how-to-guides/backup-and-restore/pg-dump-and-restore/#entire-database)

## API breaking changes

### Rename `page` to `offset` in pagination queries

### Migrate to NGSI-LD core @context v1.3
