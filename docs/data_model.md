# <a name="top"></a>Data model

* [Introduction](#introduction)
* [NGSI-LD data Model](#NGSI-LD-Data-Model)
* [Semantic Supports in `Stellio`](#Semantic-Supports-in-Stellio)
* [NGSI-LD Mapping in `Stellio`](#NGSI-LD-Mapping-in-Stellio)
* [NGSI-LD Mapping Example in `Stellio`](#NGSI-LD-Mapping-Example-in-Stellio)
* [Neo4J and TimescaleDB databases](#Neo4J-and-TimescaleDB-databases)


## Introduction
Stellio context broker supports exchanging data in the NGSI-LD format. For more details about NGSI-LD data format and data model please check the [NGSI-LD How To]( https://fiware-datamodels.readthedocs.io/en/latest/ngsi-ld_howto/index.html) pages.

In this section, we briefly remind main basics of the NGSI-LD data models. The semantic supports in `Stellio` context broker are then highlighted. `Stellio` mapping approach and use case example are then explained. Finally, the technologies used by `Stellio` for supporting semantics and storing data in graph formats are presented.

[Top](#top)
## NGSI-LD Data Model

An NGSI-LD entity is composed of:
- **id** (mandatory): The identifier of the Entity (URI format).
- **type** (mandatory): The type of the Entity
- **Properties** (optional)
- **Relationships** (optional)
- **@context** (mandatory): JSON-LD specification for linked data.

A property in NGSI-LD is composed of
- **type** (mandatory): `Property` or `GeoProperty`
- **value** (mandatory)
- **unitCode** (optional): The unit of the value in [CEFACT](https://www.unece.org/cefact/codesfortrade/codes_index.html) format.
- **Properties** (optional)
- **Relationships** (optional)

A relationship in NGSI-LD is composed of
- **type** (mandatory): `Relationship`
- **object** (mandatory): The URI of the target Entity.
- **Properties** (optional)
- **Relationships** (optional)

Temporal Properties `observedAt`, `createdAt` and `observedAt` follow the [ISO 8086](https://www.iso.org/iso-8601-date-and-time-format.html) format.

[Top](#top)
## Semantic Supports in Stellio
Storing NGSI-LD entities with their relationships and properties in semantic databases (triple stores) is a complex task especially with time series IoT data. In fact, the [RDF reification](https://www.w3.org/DesignIssues/Reify.html) is the default way to model property graphs, by allowing to attach properties to relationships, which is not directly supported by triple stores. For this purpose, most of the actual implementations of the NGSI-LD context brokers have switched to syntactic databases where the linked data aspect in JSON-LD is only serving as definition for the NGSI-LD attributes.

To enhance the exploitation of the semantic Web technologies such as reasoning, multi-typing, data validation... `Stellio` adopts an innovative mapping format which simplify the process of storing NGSI-LD entities in an efficient graph database, where the aspect of linked data is ensured.

An entity in `Stellio` is stored as a set of linked nodes via labelled edges. Any updates on attributes of this entity  will update the values of the nodes and produces a new storing event in time series databases for the former values. In a such process, the context is stored in a graph database, latest values of properties and relationships are also stored in the graph database. The former values of each updated attributes are transmitted to a time series database.

Queries then over the `Stellio` context broker are processed according to their nature. Queries about entities (get the actual values of the entity) are processed in the Graph database since they are stored there. Queries about the temporal evolution of an entity (historical data) are processed in the time series database since they are stored there.

[Top](#top)
## NGSI-LD Mapping in Stellio

The graph database in `Stellio` follows the basics of the semantic Web specifications. It is mainly based a triplets format where a triplet consists of a subject (node), an object(node) and a relation (vertex) relating the subject and the object. 

The proposed mapping approach for an [NGSI-LD entity](#NGSI-LD-Data-Model) is as follow:

- Each entity is mapped to a subject node with the same id and type. `createdAt`, `modifiedAt` and `location` attributes of the NGSI-LD entity are mapped to literal properties of this node.

- Each property is mapped to an object node. The vertex relating the entity node to its property node is labelled `has_value`. The Property name, `value`, `createdAt`, `modifiedAt` and `unitCode` are mapped to literal properties of this node. In the case of a property of property in NGSI-LD, each property will be modelled as a node and a new `has_value` vertex relation will be created between them. 

- Each relationship is mapped to an object node. `createdAt` and `modifiedAt` attribute are mapped to literal properties of this node. The vertex relating the entity node to its relationship is labelled `has_object`. As the object of a relationship in NGSI-LD is the URI of the related Entity, two vertexes labelled `has_object` and the name of the Relationship are created from the Relationship node the tagged entity node. In the case of:
    - Relationship of Relationship: each sub-relationship is mapped to a subject node and new `has_object` vertex will link these Relationships. The sub-relationship is related to the tagged Entity via two vertexes labelled `has_object` and the name of the sub-Relationship.

    - Property of a Relationship: each sub-property of a relationship will be be mapped to a subject node and a new `has_value` vertex will be link between the relationship to its  sub-property nodes. 

    - Relationship of a Property: each sub-relationship of a property is mapped to a subject node and linked via the vertex labelled `has_object`. The sub-relationship is related to the tagged Entity via two vertexes labelled `has_object` and the name of the Relationship.

[Top](#top)
## NGSI-LD Mapping Example in Stellio
Below  an example of an NGSI-LD Vehicle Entity issued from the [NGSI-LD specification](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.03.01_60/gs_CIM009v010301p.pdf)

```json
{
    "id": "urn:ngsi-ld:Vehicle:01",
    "type": "Vehicle",
    "createdAt":"2020-08-07T12:00:00",
    "color": {
        "type": "Property",
        "value": "red",
    },
    "isParkedIn": {
        "type": "Relationship",
        "object": "urn:ngsi-ld:Parking:01",
        "createdAt":"2020-08-07T12:00:00",
        "startTime": {
            "type": "Property",
            "value": "2020-08-07T12:30:00",
        }
    },
        "@context": [
        "https://schema.lab.fiware.org/ld/context.jsonld"
    ]
}
```
The graphical presentation of the Vehicle Entity is depicted in the Figure Below:

![](figure/vehicle-ngsild.jpg)

Following the mapping approach proposed in `Stellio`:
- A (vehicle) node is generated for the `Vehicle` Entity,
- A (property) node is generated for the color Property,
- A (relationship) node is generated for the `isParkedIn` Relationship,
- A (sub-property) node is created for the `startTime` sub-property,

The graphical presentation of the Vehicle Entity in the Graph database is depicted in the Figure Below:

![](figure/vehicle-graph.jpg)

[Top](#top)
## Neo4J and TimescaleDB databases
As graph database, `Stellio` exploits the potential of the [Neo4j](https://neo4j.com/) database. Developed in Java, Neo4j one of the most popular graph database management system. Neo4j is easily accessible from external softwares written in other different languages via the expressive and efficient Cypher query language developed initially for Neo4j needs. 

Neo4j is also offering a collection of the latest innovations in graph technology that exploits stored data such as the [NeoSemantics](https://neo4j.com/labs/neosemantics/) library which offers various tool for managing linked data and supporting ontologies and semantic inferences.

This library is integrated in `Stellio` in order to exploit the references and semantic definitions from context field attached to NGSI-LD entities. Semantic Reasoning and Semantic Data validation are ongoing task in `Stellio` using this library.

The [TimescaleDB](https://www.timescale.com/) is integrated in `Stellio` which is a leading open-source relational database for time-series data.
