# Overview

Stellio is composed of 3 business services:

-   Entity service is in charge of managing the information context, it
    is backed by a [neo4j](https://neo4j.com) database
-   Search service is in charge of handling the temporal (and
    geospatial) queries, it is backed by a
    [TimescaleDB](https://www.timescale.com/) database
-   Subscription service is in charge of managing subscriptions and
    subsequent notifications, it is backed by a
    [TimescaleDB](https://www.timescale.com/) database

It is completed with:

-   An API Gateway module that dispatches requests to downstream
    services
-   A [Kafka](https://kafka.apache.org/) streaming engine that decouples
    communication inside the broker (and allows plugging other services
    seamlessly)

The services are based on the [Spring Boot](https://spring.io/projects/spring-boot) framework, developed in
[Kotlin](https://kotlinlang.org), and built with [Gradle](https://gradle.org).

# Quick start

A quick way to start using Stellio is to use the provided `docker-compose.yml` file in the root directory 
(feel free to change the default passwords defined in the `.env` file):

```shell
docker-compose up -d && docker-compose logs -f
```

It will start all the services composing the Stellio context broker platform and expose them on the following ports:

-   API Gateway: 8080
-   Entity service: 8082
-   Search service: 8083
-   Subscription service: 8084

Please note that the environment and scripts are validated on Ubuntu 19.10+ and MacOS. Some errors may occur on other platforms.

Docker images are available on [Docker Hub](https://hub.docker.com/orgs/stellio/repositories).

# Development

## Developing on a service

Requirements:

-   Java 11 (we recommend using [sdkman!](https://sdkman.io/) to install
    and manage versions of the JDK)

To develop on a specific service, you can use the provided `docker-compose.yml` file inside each service's directory, for
instance:

```shell
cd entity-service docker-compose up -d && docker-compose logs -f
```

Then, from the root directory, launch the service:

```shell
./gradlew entity-service:bootRun
```

## Running the tests

Each service has a suite of unit and integration tests. You can run them without manually launching any external component, 
thanks to Spring Boot embedded test support and to the great [TestContainers](https://www.testcontainers.org/) library.

For instance, you can launch the test suite for entity service with the following command:

```shell
./gradlew entity-service:test
```

## Building the project

To build all the services, you can just launch:

```shell
./gradlew build
```

It will compile the source code, check the code formatting (thanks to [ktLint](https://ktlint.github.io/)) and run the 
test suite for all the services.

For each service, a self executable jar is produced in the `build/libs` directory of the service.

If you want to build only one of the services, you can launch:

```shell
./gradlew entity-service:build
```

## Working locally with Docker images

To work locally with a Docker image of a service without publishing it
to Docker Hub, you can follow the below instructions:

-   Build a tar image:

```shell
./gradlew entity-service:jibBuildTar
```

-   Load the tar image into Docker:

```shell
docker load --input entity-service/build/jib-image.tar
```

-   Run the image:

```shell
docker run stellio/stellio-entity-service:latest
```

# Usage

To start using Stellio, you can follow the [API quick guide](https://github.com/easy-global-market/ngsild-api-data-models/blob/master/API_Quick_Guide.md)
published in our NGSI-LD API & Data Model repository.

As the development environment does not make use of the authentication setup, you can ignore related information in the API quick guide.
