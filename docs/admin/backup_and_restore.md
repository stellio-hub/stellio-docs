# Backup a Stellio instance

Below is a basic script that can be used to backup the Stellio databases. Feel free to take it as a basis (don't forget to review the environement variables at the top of the script) and adapt it to fit your needs.

```shell
#!/usr/bin/env bash

# Number of days of history kept for backups
# 1st argument passed on the command line, 30 either
DAYS_HISTORY=${1:-30}
# Where the backups will be put
BACKUP_DIR=$HOME/backup/stellio
# Where docker-compose file for Stellio is installed
STELLIO_COMPOSE_DIR=$HOME/stellio-launcher

source $STELLIO_COMPOSE_DIR/.env

mkdir -p $BACKUP_DIR
now=$(date +%Y-%m-%d)
echo "Backups will be suffixed with date $now and placed into $HOME/backup/stellio"

echo
echo "Performing a backup of PostgreSQL databases (search and subscription)"

# Following instructions from https://docs.timescale.com/latest/using-timescaledb/backup#pg_dump-pg_restore
# It display warnings ... that can be ignored
# Cf https://github.com/timescale/timescaledb/issues/1581
docker exec postgres /bin/bash -c "export PGPASSWORD=$POSTGRES_PASSWORD && /usr/local/bin/pg_dump -Fc -U postgres stellio_search" | gzip -9 > /tmp/postgres_search_$now.gz
docker exec postgres /bin/bash -c "export PGPASSWORD=$POSTGRES_PASSWORD && /usr/local/bin/pg_dump -Fc -U postgres stellio_subscription" | gzip -9 > /tmp/postgres_subscription_$now.gz

mv /tmp/postgres_search_$now.gz $BACKUP_DIR/.
mv /tmp/postgres_subscription_$now.gz $BACKUP_DIR/.

echo
echo "Stopping Stellio containers (to backup Neo4j and Grafana)"

cd $STELLIO_COMPOSE_DIR
/usr/local/bin/docker-compose -f docker-compose.yml stop

echo
echo "Performing a backup of Neo4j"

# It needs to be writable by the user inside the neo4j container run just after
# TODO it should be improved, this is currently a brute force solution
chmod 777 $BACKUP_DIR

# Be sure to change the neo4j version if you are not yet on 4.4
# Also check the name of the volume for neo4j, you can find it with the "docker volume ls" command
neo4j_data_volume=$(docker volume inspect --format '{{ .Mountpoint }}' stellio-launcher_neo4j-storage)
docker run --rm --publish=7474:7474 --publish=7687:7687 --volume=$neo4j_data_volume:/data --volume=$BACKUP_DIR:/backups neo4j:4.4 neo4j-admin dump --database=stellio --to=/backups/neo4j_$now.dump

echo
echo "Performing a backup of Kafka"

docker run --rm --volumes-from kafka -v $BACKUP_DIR:/backup ubuntu tar cvf /backup/kafka_$now.tar /var/lib/kafka/data
gzip -f $BACKUP_DIR/kafka_$now.tar

echo
echo "Performing a backup of Zookeeper"

docker run --rm --volumes-from zookeeper -v $BACKUP_DIR:/backup ubuntu tar cvf /backup/zookeeper_$now.tar /var/lib/zookeeper/data
gzip -f $BACKUP_DIR/zookeeper_$now.tar

echo
echo "Restarting Stellio containers"

/usr/local/bin/docker-compose -f docker-compose.yml start

echo
echo "Deleting backups older than $DAYS_HISTORY days"

find $BACKUP_DIR -mtime +$DAYS_HISTORY -delete

echo
echo "Backup terminated!"
```

You can call it in a cron job like this:

```shell
30 1 * * * /path/to/backup_databases.sh # called every night at 1:30am
```

# Restore a Stellio instance

## Set some environment variables

WARNING : backup date has to be set up manually in other places below, review them carefully!

```shell
backup_date=2021-05-13
export BACKUP_DIR=$HOME/backup/stellio
export STELLIO_COMPOSE_DIR=$HOME/stellio-launcher
```

## Step 1 - Restore PG

* Launch Postgres container and connect into it

```shell
cd $STELLIO_COMPOSE_DIR
docker-compose up -d postgres

docker cp $BACKUP_DIR/postgres_search_$backup_date.gz stellio-postgres:/tmp/.
docker cp $BACKUP_DIR/postgres_subscription_$backup_date.gz stellio-postgres:/tmp/.

docker exec -it stellio-postgres bash
```

* Once in the container, restore the databases

```shell
backup_date=2021-05-13 # need to be set again in the container
gunzip /tmp/postgres_search_$backup_date.gz
gunzip /tmp/postgres_subscription_$backup_date.gz

# then following https://docs.timescale.com/timescaledb/latest/how-to-guides/backup-and-restore/pg-dump-and-restore/#entire-database
su - postgres
psql

\c stellio_search
CREATE EXTENSION IF NOT EXISTS timescaledb;
SELECT timescaledb_pre_restore();
\! pg_restore -Fc -d stellio_search /tmp/postgres_search_2021-05-13 -- change the date!
SELECT timescaledb_post_restore();

\c stellio_subscription
CREATE EXTENSION IF NOT EXISTS timescaledb;
SELECT timescaledb_pre_restore();
\! pg_restore -Fc -d stellio_subscription /tmp/postgres_subscription_2021-05-13 -- change the date!
SELECT timescaledb_post_restore();

exit # from psql
exit # from postgres user and be root again

rm -f /tmp/postgres_search_*
rm -f /tmp/postgres_subscription_*

exit # from the container
```

* Stop the Postgres container

```shell
docker-compose stop postgres
```

## Step 2 - Restore neo4j

```shell
cd $STELLIO_COMPOSE_DIR
# Start and stop neo4j to create the container and volume if they do not yet exist
docker-compose up -d neo4j
docker-compose logs -f neo4j # wait for neo4j to finish starting
docker-compose stop neo4j

docker run --interactive --tty --rm --publish=7474:7474 --publish=7687:7687 --volumes-from neo4j --volume=$BACKUP_DIR:/backups neo4j:4.4 neo4j-admin load --from=/backups/neo4j_$backup_date.dump --database=stellio --force
```

## Step 3 - Restore Kafka and Zookeeper

```shell
cd $STELLIO_COMPOSE_DIR
# Start and stop kafka and zookeeper to create the containers and volumes if they do not yet exist
docker-compose up -d zookeeper kafka
docker-compose logs -f zookeeper kafka # wait for zookeeper and kafka to finish starting
docker-compose stop zookeeper kafka

docker run --rm --volumes-from zookeeper -v $BACKUP_DIR:/backup ubuntu tar -C /var/lib/zookeeper/data -xzf /backup/zookeeper_$backup_date.tar.gz --strip-components 4
docker run --rm --volumes-from kafka -v $BACKUP_DIR:/backup ubuntu tar -C /var/lib/kafka/data -xzf /backup/kafka_$backup_date.tar.gz --strip-components 4
```

## Step 4 - Restart Stellio

```shell
cd $STELLIO_COMPOSE_DIR
docker-compose up -d && docker-compose logs -f --tail=100
```
