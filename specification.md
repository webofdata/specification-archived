
# Web of Data (WoD) Specification

Author  : Graham Moore  
Contact : gramoore (at) outlook (dot) com  
Version : 0.9  
This Document : [Specification 0.8](http://github.com/webofdata/specification/specification.md)  
Issues : <http://github.com/webofdata/specification/issues>

## Overview

Like HTML and HTTP did for machine to human communication, Web of Data (WoD) aims to provide the building blocks to allow any device or application to easily communicate, access, modify, navigate and share data at web scale. Enabling low friction, meaningful,standardised, machine-to-machine communication.

Web of Data defines a protocol, data model, and data representation format for publishing, sharing and connecting data on the web. It is intended to be easy to implement and work at a level of generality that is widely applicable.

The WoD data model has at it's core the notion of an entity. An entity has identity in the form of an IRI. An entity also has properties whose 'names' or 'keys' are also IRIs. The use of IRIs to identify things and name properties provides a powerful, web scale approach to naming. 

The values of these properties can either be literals, based on XML schema data types, or complex types such as arrays, or another entity. Finally, there is a reference type. The reference type has a value that is a URI. This URI references another entity, allowing the creation of a graph of entities that can span the web.

The protocol is designed to facilitate the sharing, updating, creating and publishing of collections of entities, the retrieval of a given data entity, and the navigation of connected entities. It is expected that collections of entities can exists in databases, existing applications and dedicated WoD data stores. 

All kinds of applications can implement and support the protocol in meaningful ways, allowing a vast array of client applications to easily consume and use data from around the web.

The WoD representation format uses JSON with a few well defined keys that support the use of IRIs as identifiers and property names.

Finally, WoD defines a protocol for synchronising datasets. This allows clients to collect and synchronise data from around the web; to offer new data services and provide guaranteed quality of service to application and API users.

## Background

Several attempts have been made to define a standard and generic data model and related APIs; most notably OData and the RDF stack from W3C. While technically sound they both have significant barriers to adoption - The RDF data model is too shredded and lacks basic constructs such as list and map; OData has a data model that doesn't embrace URIs for identity or naming. The query semantics and models in both is unappealing to users, and too much work for implementors.

WebOfData builds on the experience of these and other standards to propose a universal data management and data sharing API, coupled with a data model that works at web scale.

A key concept that underpins WebOfData is the notion of a 'subject'. A subject is any thing, about which, anything may be stated or asserted, by anyone. A subject is an abstract concept, it can be a person, a document on a file system, a row in a database, a book.

Computer systems are all about building representations of these abstractions. Every application creates a model where the data structures are proxies, or stand-ins for their 'real life' counterpart.  

Standards such as RDF and Topic Maps realised that to create a global space in which anyone can talk about any thing, there needs to be an agreed way to bind the data object in a computer system to the subject it represents. They also realised that there can be many partial representations of the same subject. These partial representations may or may not be known at any given time.

Using URIs for identifying subjects provides a globally unique and authorative mechanism to unambiguously name the subjects of discourse and the vocabularies used to describe them.

WebOfData uses the term 'entity' to be the data object that is a representation of a subject. The binding between these two things is the use of an IRI as the identifier for a given subject and entity. Note that many entities can have the same identifier. If they do, then they can considered as partial representations of the same subject.  

The WoD APIs are refinement of things like OData, Linked Data Fragments and SDShare.

## Introduction

WebOfData defines a data model, a JSON serialistion for the data model, a data access protocol (Query), a data sharing protocol (Synchronisation) and a data management protocol (CRUD). Compliant implementations can choose which of the APIs they support, but must adhere to the rules and semantics of the data model and serialisation definition.

Software that implements these protocols and adheres to the serialisation representations are called WoD nodes. The potential set of services that can connect these WoD nodes together is what makes the WoD protocol exciting for the web at large and organisations looking to get control of their data.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## Data Model Definition

The WebOfData data model can be thought of as an Entity-Graph model. Entities are complex objects of keys with values. Where keys are IRIs and values can be literals, lists of values, an entity or list of entities. In addition, a reference value type can point to (reference) another entity. This allows entities to be connected together in a graph at global scale.

The value of a reference type is an IRI and not some local internal identifier. The power of this is that it can be resolved globally. It can be used to lookup the related entity from one or more API endpoints. WoD does not define where a client can resolve a reference value, but does define the capabilities that a WoD query endpoint should support.

Given that each entity is only a partial representation of a subject the data model also defines how to merge two entities. (The algorithm can be applied recursively to merge multiple representations.)

As well as entities there are two levels of container types. The store and the dataset. A store consists of zero or more datasets, and each dataset is comprised of 0 or more entities.

### Core Terms

<dl>
  <dt>Subject</dt>
  <dd>A concept, an idea, a thing; any thing, about which anything can be said.</dd>

  <dt>Subject Identifier</dt>
  <dd>A URI used to identify a subject.</dd>

  <dt>Entity</dt>
  <dd>A data object construct that can be created in software. It is the proxy for the subject. The identity of the entity is a URI that defines which subject it is a proxy for.</dd>

  <dt>Subject Reference</dt>
  <dd>A URI used to identify a subject.</dd>

  <dt>Store</dt>
  <dd>A store is a collection of datasets.</dd>

  <dt>Dataset</dt>
  <dd>A dataset is a collection of entities. At most one entity with a given subject identifier can be present in a dataset.</dd>
  
</dl>

### Entities

Entities are the core of the data model. The following definition describes the structure of an entity.

```
entity               := { id, properties, references, metadata }

child-entity         := { id, properties, references } | { properties, references }

id                   := xsd:uri

properties           := key-val-pair | key-val-pair, properties

metadata             := key-val-pair | key-val-pair, metadata

references           := key-ref-pair | key-ref-pair, references

key-val-pair         := xsd:uri , value | list(value)

key-ref-pair         := xsd:uri , xsd:uri | list(xsd:uri)

value                := child-entity | xsd:string |
                        xsd:int | xsd:datetime | xsd:uri | xsd:double | xsd:float
```

One of the key things to note is that a root-entity must have a id. Descendent entities are not required to have an id. If a child entity has an id it is implied that the entity has been embedded into the structure, but also has identity in its own right. Only the identity of the root conveys the subject identity of the entity.

Another key thing to note is that when querying for incoming references (e.g. those that point to an entity) only references of the root entity should be considered. 

We will come to serialisation later, but to give a feel of what the model above looks like when serialised in JSON the following example is provided.

``` json
{
  "@id" : "ns1:gra",
  "@props" : {
    "foaf:name" : "Graham Moore"
  },
  "@refs" : {
    "foaf:friendof" : "people:kal"
    "wod:type" : "schemaorg:Person",
  },
  "@meta" : {
    "deleted" : false,
    "etag" : "00001",
    "modified" : "939538559359"
  }
}
```

### The WebOfData Namespace

Some built in subject identifiers are of the form `wod:xxxx`. The wod expansion is defined as:

```
http://data.webofdata.io/
```

### Ontological Commitments

The WoD data model can be used to describe both the class and instance level of data. E.g. an Entity can represent the class person and instances of that class. It can also define property types and constraints. As well as the set of identifiers and related interpretation we define a few basic identifiers and define their meaning.

#### The Type Property Type

The `wod:type` property is used to indicate that a given entity is an instance of the referenced class. An entity can be an instance of 0 or more classes. A class is just an entity.

The following example shows how to define an individual entity to be an instance (or have type) of a class entity.

``` json
// we defined the class entity first
{
  "@id" : "schema:Person"
}

// and then the instance using the wod:type property
{
  "@id" : "people:gra",
  "@refs" : {
    "wod:type" : "schema:Person"
  }
}
```

#### The Class Class

As a means to indicate which entities can be used as types for others it is useful to mark them. The class entity is the way to do this.

We can extend the original example above to include a type reference to the Class entity.

``` json
{
  "@id" : "schema:Person",
  "@refs" : {
    // we add the wod:type property that references the wod:Class entity
    "wod:type" : "wod:Class"
  }
}

// and then the instance using the type property
{
  "@id" : "people:gra",
  "@refs" : {
    "wod:type" : "schema:Person"
  }
}
```

#### The subClassOf Property

The other pattern we want to support is the abilty to define a class hierarchy. 

``` json
// we create a new base class
{
  "@id" : "schema:Thing",
  "@refs" : {
    "wod:type" : "wod:Class"
  }
}

{
  "@id" : "schema:Person",
  "@refs" : {
    "wod:type" : "wod:Class",
    // and use the subClassOf property indicate the super class of this class
    "wod:subClassOf" : "schema:Thing"
  }
}

// and then the instance using the type property
{
  "@id" : "people:gra",
  "@refs" : {
    "wod:type" : "schema:Person"
  }
}
```

#### The Property Class

In the same way that it is useful to be able to indicate which entities are classes it is also useful to indicate which entities are used as property types. The following example shows the intended usage.

``` json
// the age property type can be defined as follows
{
  "@id" : "schema:age",
  "@refs" : {
    "wod:type" : "wod:PropertyClass"
  }
}
```

#### Entailments

Entailments just means `things that follow given something else`. So in terms of the WoD data model there are some inferred properties based on using the above built in properties.

The following entailment MUST be respected:

- wod:subClassOf is Transitive. This means that if `a wod:type B` and `B wod:subClassOf C`, then `a wod:type C` is true.

### DataSets

A dataset is a collection of entities. It provides a container where entities can be managed (created, deleted, updated, located).

Datasets can contain at most one entity with any given subject identifier. This means that the subject identifier is the unique key for an entity in a dataset. However, an entity with the same subject identifier can exist in other datasets.

Splitting the store into datasets allows for more than one representation of the same subject (an entity with the same subject identifier as another entity) to exist in a single store.

By default when resolving a request for a subject identifier against a store then the lookup is across all datasets. In this case it is possible for more than one entity with the same subject identifier to be located. When this occurs the entity representations should be merged as described below. A client can request that lookup only occurs in the context of one dataset. In this case, due to the fact that within a dataset there is exaclty zero or one entity with a given subject identifier, at most one entity will be located.

Formally, and in terms of the entity definition:

```
dataset        := { id, dataset-entity, entities }
dataset-entity := entity
entities       := [] | entity, entities
```

### Store

A store is a collection of datasets. The store is defined as follows:

```
store        := { id, store-entity, datasets }
store-entity := entity
datasets     := [] | dataset, datasets
```

### Merging Entities

A client can (potentially) retrieve a partial subject representation for the same subject from many WoD service endpoints. An application or server may decide that even though the subject identifiers are different that the entities represent the same subject. In addition, entities are allowed to express a belief that they are the same as one or more other subjects using the "wod:sameas" reference property. In all of these cases two or more entities are merged to create a more unified representation. We say 'more unified' as there is by definition never a complete representation of a subject.

The rules for merging two entity representations of the same subject are defined below. If more than two entities need to be merged, then the algorithm should be applied on the first two entities, then on the resulting merged entity and the next entity to be merged; and so on.

To merge two subject representations A and B the following algorithm should be applied:

  - Let A and B be the two entity representations to be merged.
  - Create a new empty entity representation, C.
  - The "id" property of C becomes the value of the "id" property of A.
  - For each property (key and value) in A if there is no corresponding property (same key) in B add it to C.
  - For each property in A if there is a corresponding property in B (same key) then merge the values:
    - If either of property values are a list then they are merged by flattening them into one list: e.g. [a, b, c][d, e] becomes [a,b,c,d,e], and [ {}, [], "d"], "x" becomes: [ {}, [], "d", "x" ].
    - For non list values a new list is created and the values from the property in A and B are added. e.g. {}, "x" becomes: [ {}, "x" ]
    - Duplicates are removed.
    - Add the merged value onto C
  - All properties on B that are not present on A are added to C.

#### SameAs Resolution

The above merging semantics define the formalism for merging two entities with the same subject identifier. In addition, the sameas resolution rule defines how a WoD server can recursively merge a set of entities that declare themselves to be the same.

This behaviour is optional and a server MAY offer this capability. Even in the case when it does offer this capability the client MUST specifically request it.

Given a subject identifier, find the entity that corresponds to that URI. Locate the 'wod:sameas' reference-property and for each 'subject-ref' value in the array look up and merge that entity according to the rules above.

Recursively apply this rule to each entity referenced by a wod:sameas value, but only process each entity once.

## Data Model JSON Serialisation

JSON is used as the serialisation format for the Data Model. It defines how an instance of the entity data model can be serialised as JSON. The serialisation does not use any extensions to the JSON language, a valid WoD Data Model JSON document is a valid JSON document, but it is not true that any JSON document is a valid WoD JSON document.

### Core Rules

The `id` property of the entity is serialised with the key `@id`.

While full URIs can be used in WoD JSON documents as keys, values and id property values, it is optimal for machines and easier for humans if full URIs are replaced with CURIEs. Context definitions are used to define default namespaces and CURIE expansions.

The following rules define how JSON data must be written in order to be considered valid WoD JSON.

#### The @id Property Rule

The `@id` property of the entity is serialised as key of `@id`. The value of this property defines the identifier for the entity. This property MUST be present on the root JSON object. The value of this property can be an full URI or a CURIE. If it is neither of these two then it is interpreted in relation to the default context (see context resolution).

The following examples show the different serialisations that are all semantically equivalent:

``` json
  // Explicit full URI:
  {
    "@id" : "http://data.webofdata.io/people/gra"
    ...
  }
```

or

``` json
  // Use of a context with prefix definition:
  [
    {
      "@id" : "@context",
      "namespaces" : { "people" : "http://data.webofdata.io/people/" }
    },
    {
      "@id" : "people:gra"
    }
  ]
```

or 

``` json
  // Use of a context with a default namespace:

  [
    {
      "@id" : "@context",
      "namespaces"  : { "_" : "http://data.example.org/people/" }
    },
    {
      "@id" : "gra"
    }
  ]
```

The @id property can also be used on any descendant object. When used on a descendant object it gives identity to that object. It does not affect or override the identity of the root object.

This provides a way to embed or contain other subjects by value and still convey that they have identity in their own right. The following example demonstrates this usage.

``` json
  {
    "@id" : "http://data.example.org/people/gra",
    "@props" : {
      "education" : [
        {
          "@props" : {
            "@id" : "http://data.schools.org/stbarts",
            "name" : "st bartholomews"
          }
        },
        {
          "@id" : "http://data.universities.org/southampton",
          "@props" : {
            "name" : "Southampton University"
          }
        }
      ]
    }
  }
```

#### Entity Properties

The data properties of an entity are serialised in as JSON object. The key of the JSON object is '@props'. All key value pairs in this JSON object are considered to be data properties of the entity. The keys are all either full URIs,CURIES or a simple string. If they are just a string then they are resolved to be a full URI against the default namespace.  

The following example shows an Entity with a simple data section.

``` json
[
  {
    "@id" : "@context",
    "namespaces"  : {
      "_" : "http://data.webofdata.io/people/",
      "foaf" : "http://foaf.org/"
    }
  },
  {
    "@id" : "gra",
    "@props" : {
      "name" : "gra",
      "foaf:name" : "Graham Moore"
    }
  }
]
```

#### Entity References

Entities can reference other subjects by their subject identifier. All subject reference are collected together in the 'refs' dictionary of the entity. All values in this dictionary must be CURIES or URIS. Simple strings that are resolved against the context to a a full URI. The following examples show how to subject references are serialised.

``` json
[
  {
    "@id" : "@context",
    "namespaces"  : {
        "_" : "http://data.webofdata.io/people/",
        "foaf" : "http://foaf.org/"
    }
  },

  {
    "@id" : "gra",
    "@props" : {
      "name" : "gra",
      "foaf:name" : "Graham Moore"
    },
    "@refs" : {
      "education" : "http://data.webofdata.io/universities/soton;"
    }
  },

  {
    "@id" : "http://data.webofdata.io/universities/soton",
    "@props" : {
      "address" : "southampton"
    }
  }
]
```

#### Property Naming Rule

As well as being able to talk about 'subjects' using subject identifiers, we also need to be able to agree on the properties being used. In WoD all property names are URIs. They can be expressed as full IRIs, Curies or simple values. If they are expressed as simple values they are resolved against the default namespace defined in a context.

An example with properties as IRIs. Both the 'foaf:name' and 'http://data.example.org/vocab/lives-in' are properties that will resolve to full IRIs.

``` json
[
  {
    "@id" : "@context",
    "namespaces" : {
      "foaf" : "http://foaf.org/schema"
    }
  },
  {
    "@id" : "http://data.example.org/people/gra",
    "@props" : {
      "foaf:name" : "Graham Moore",
      "http://data.example.org/vocab/fullname" : "Graham Moore"
    }
  }
]
```

#### Property Value Rule

In general, property value datatypes are conferred by the JSON datatype. However, JSON data types are limited. The set of types supported by WoD is the same as those defined by XML Schema (https://www.w3.org/TR/xmlschema-2/#built-in-datatypes).

A client can explicitly indicate the datatype of a property value by providing a string value using the following pattern:

```
  'xsd' : 'typename' : VALUE
```

The `xsd` namespace is built in and resolves to `http://www.w3.org/2001/XMLSchema#`

For example:

``` json
  {
    "@props" : {
      "name" : "xsd:string:Graham Moore"
    }
  }
```

is equivalent to:

``` json
  {
    "@props" : {
      "name" : "Graham Moore"
    }
  }
```

#### Context Rules

All property names, '@id' property values, and reference values are full URIs in the model. However, to simplify documents and avoid repeating long URI strings these values can be specified either as CURIEs or as simple values that are resolved against a base namespace defined in a context.

If a context is provided is should appear as the first entity in a JSON array. When defined in this way the first object in the array is an entity where the value of the `@id` property MUST be `@context`. 

The value of the `@context` property is a JSON object with a single key called `namespaces`.

The following example shows a context definition defined as the first entity in the array.

``` json
  [
    {
      "@id" : "@context",
      "namespaces" : {
        "foaf" : "http://xmlns.com/foaf/0.1/"
      }
    },

    {
      "@id" : "...",
      "@props" : {
        "foaf:name" : "bob"
      }
    },

    {
      "@id" : "..."
    }
  ]
```

The "namespaces" key has a value that is a JSON object. The keys in this object correspond to the first part of a CURIE and the value (which must be a string) is the expansion.

As well as CURIEs the "namespaces" object can also contain the definition of a default namespace. The default namespace is used when no CURIE prefix is used and the value is not a full URI.

The default instance expansion is defined using the key of "_".

The following example shows the use of the default namespace and two semantically equivalent serialisations.

``` json
[
  {
    "@id" : "@context",
    "namespaces" : { "foaf" : "http://foaf.org/schema/"}
  },

  {
    "@id" : "http://data.example.org/people/gra",
    "@props" : {
      "foaf:name" : "Graham Moore"
    }
  }
]

is the same as:

[
  {
    "@id" : "@context",
    "namespaces" : { "_" : "http://foaf.org/schema/"}
  },

  {
    "@id" : "http://data.example.org/people/gra",
    "@props" : {
      "name" : "Graham Moore"
    }
  }
]

In both examples the 'name' property is expanded to 'http://foaf.org/schema/name'

```

## Web of Data Protocol

The WoD protocol consists of a service definition and the related semantics for each operation. The operations are described in prose and swagger. The protocol is split into three distinct areas; data management, data synchronisation, and data query.

The WoD protocol intentionally covers a wide scope of data publishing, data management and data query use cases. It is perfectly acceptable for implementations to support small sections of this API and still be compliant and useful members of the web of data. The only requirement is that implementations must be consistent, e.g. if it supports 'create-dataset' operation then it must also support 'get-datasets', but another service can offer 'get-datasets' without offering 'create-datasets'. This is to be expected when some implementations are layers over existing read-only data sources. All operations are clearly marked to indicate if they are required. For operations that are not mandatory they can return a suitable 'not implemented' response.

## Common Definitions

The following swagger data type definitions are used and referenced throughout the protocol description. They are listed here just once.

``` yaml
definitions:

  ServiceInfo:
    type: "object"
    properties:
      "name":
        "type": "string"
      "baseurl":
        "type": "string"
      "entity":
        "$ref": "#/definitions/Entity"

  Store:
    type: "object"
    properties:
      "name":
        "type": "string"
      "entity":
        "$ref": "#/definitions/Entity"

  Dataset:
    type: "object"
    properties:
      "name":
        "type": "string"
      "entity":
        "$ref": "#/definitions/Entity"

  Entity:
    type: "object"
    properties:
      "@id":
        type: "string"
      "@deleted":
        type: "boolean"
        default: false
      "@etag":
        type: "string"
```

## Data Management

The data management part of the protocol defines operations for the creation and deletion of stores and datasets, and the updating of entities within a dataset.

### Operation Get Service Info

This operation returns information about the service endpoint itself. The response MUST contain a service entity for this endpoint. The entity returned as part of the ServiceInfo response is the subject identifier for the service endpoint.

``` yaml
  /info:
    get:
      tags:
        - "Management"
      summary: "Returns information about this service instance"
      description: "Returns information about this service instance, including an entity which conveys on this service instance a unique identifier"
      operationId: "get-service-info"
      produces:
        - "application/json"
      responses:
        200:
          description: "successful operation"
          schema:
            $ref: "#/definitions/ServiceInfo"
```

The following example shows an example request and response:

```
> GET /info HTTP/1.1
> Host: api.webofdata.io
> Accept: */*
> 
< HTTP/1.1 200 OK
< Content-Length: 71
< 
{  
   "name"   : "node1",
   "entity" : { "@id" : "http://data.webofdata.io/services/demonode" }
}
```

### Operation Get Stores

This operation returns a list of stores being managed by the WoD node. The list of stores can be filtered by providing the id of the store as a query parameter. Note that stores have a name that forms part of the URI, but also has a metadata entity that represents the store. This entity has an id in the form of a URI and it is this value that can be used to filter the list of stores.


``` yaml
  /stores:
    get:
      tags:
        - "Management"
      summary: Returns a list of stores
      description: "returns a list of stores"
      operationId: "get-stores"
      produces:
        - "application/json"
      parameters:
        - name: "id"
          in: "query"
          description: "if present tries to find the store with that id (note: not the store name)"
          required: false
          type: string
      responses:
        200:
          description: "successful operation"
          schema:
            type: "array"
            items:
              $ref: "#/definitions/Store"
```

The following example shows a request that returns three stores.


Request:

```
> GET /stores HTTP/1.1
> Host: api.webofdata.io
>
< HTTP/1.1 200 OK
<
```

Response body:

``` json
[
  {
    "name" : "store1",
    "entity" : {
      "@id" : "http://data.webofdata.io/publishing/store1"
    }
  },
  {
    "name" : "store2",
    "entity" : {
        "@id" : "http://data.webofdata.io/publishing/store2"
    }
  },
  {
    "name" : "store3",
    "entity" : {
        "@id" : "http://data.webofdata.io/publishing/store3"
    }
  }  
]
```

The following example shows a request that returns just one store. Only one store is returned as the query parameter is used to locate the store whose entity id matches the value provided.

Request:

```
> GET /stores?id=http://data.webofdata.io/publishing/store1 HTTP/1.1
> Host: api.webofdata.io
>
< HTTP/1.1 200 OK
<
```

Response body:

``` json
[
  {
    "name" : "store1",
    "entity" : {
      "@id" : "http://data.webofdata.io/publishing/store1"
    }
  }  
]
```

### Operation Create Store

The optional create store operation expects a JSON object to be sent in the body of the request. The JSON object may contain a 'name' property and MUST contains an 'entity' property. The value of the 'entity' property MUST be a JSON object that is valid according to the WoD JSON specification. If the 'name' property is present it MUST contain a string value.

The server rejects the request if the store name is already in use, or if the id of the entity is the same as any existing store.

If no name is provided the server MUST generate one.

The operation is defined as follows:

``` yaml
/stores:
    post:
      tags:
        - "Management"
      summary: Create a new store
      description: "Creates a new store with the name provided if and only if this name is not already in use. If no name is provided then the server MUST generate one"
      operationId: "create-store"
      produces:
        - "application/json"
      parameters:
        - name: "body"
          in: "body"
          description: "Store to create"
          required: true
          schema:
            $ref: "#/definitions/Store"
      responses:
        201:
          description: "successful operation - store created"
          schema:
            $ref: "#/definitions/Store"
        400:
          description: "Occurs when the store passed in the body is invalid or missing"
```

The following example shows the create store request:

```
> POST /stores HTTP/1.1
> Host: api.webofdata.io
{
  "name" : "store1",
  "entity" : {
    "@id" : "http://data.webofdata.io/publishing/store1"
  }
}

< HTTP/1.1 201 CREATED
< Location: api.webofdata.io/stores/store1
<
{
  "name" : "store1",
  "entity" : {
    "@id" : "http://data.webofdata.io/publishing/store1"
  }
}  
```

The following example shows a valid create store request where the server generates the store name.

```
> POST /stores HTTP/1.1
> Host: api.webofdata.io
{
  "entity" : {
    "@id" : "http://data.webofdata.io/publishing/store1"
  }
}  

< HTTP/1.1 201 CREATED
< Location: api.webofdata.io/stores/somename
<
{
  "name" : "somename",
  "entity" : {
    "@id" : "http://data.webofdata.io/publishing/store1"
  }
}  
```

### Operation Get Store

The 'get-store' operation retrieves the store entity using the local store name.

``` yaml
  /stores/{store-name}:
    get:
      tags:
        - "Management"
      summary: get store by its local store name
      description: returns a single store object or 404 not found
      operationId: get-store
      produces:
        - "application/json"
      parameters:
        - name: "store-name"
          in: "path"
          description: "unique store name for this service endpoint"
          required: true
          type: "string"
      responses:
        200:
          description: "successful operation"
          schema:
            $ref: "#/definitions/Store"
        404:
          description: "store not found"
```

The following example shows the use of this operation:

```
> GET /stores/store1 HTTP/1.1
> Host: api.webofdata.io
>

< HTTP/1.1 200 OK
<
[
  {
    "name" : "store1",
    "entity" : {
      "@id" : "http://data.webofdata.io/publishing/store1"
    }
  }  
]
```

### Operation Delete Store

The optional 'delete-store' operation requests that the implementing WoD node deletes the store  all datasets, and any metadata associated with it. This includes the entity for the store itself.

``` yaml
  /stores/{store-name}:
    delete:
      tags:
        - "Management"
      summary: delete the named store, and all datasets
      description: deletes the named store or returns a 404 not found if no store of the name provided exists
      operationId: delete-store
      parameters:
        - name: "store-name"
          in: "path"
          description: "store name"
          required: true
          type: "string"
      responses:
        200:
          description: "successful operation - store deleted."
        404:
          description: "Store not found"
```

The following example shows how to request that a store is deleted.

```
> DELETE /stores/store1 HTTP/1.1
> Host: api.webofdata.io
>
< HTTP/1.1 200 OK
<
```

### Operation Update Store Entity

The `update-store-entity` operation allows the entity for a store to be updated by using a POST request. The body of the request contains the new representation for the store entity.

``` yaml
  /stores/{store-name}:
    put:
      tags:
        - "Management"
      summary: updates the store entity
      description: updates the entity that represents the store
      operationId: update-store
      parameters:
        - name: "store-name"
          in: "path"
          description: "store name"
          required: true
          type: "string"
        - name: "body"
          in: "body"
          description: "New representation of the store entity"
          required: true
          schema:
            $ref: "#/definitions/Entity"
      responses:
        200:
          description: "successful operation"
          schema:
            $ref: "#/definitions/Store"
        404:
          description: "store not found"

```

The following example shows how to update the entity for a store. Note in this example that the server renames the namespace prefix. A server is free to rename or map prefixes as it sees fit. What must hold true is the fully expanded URI.

```
> PUT /stores/store1 HTTP/1.1
> Host: api.webofdata.io
>
{
  "@id"       : "http://data.webofdata.io/publishing/newidentifier",
  "http://data.webofdata.io/schema/name" : "This store is very important"
}  

< HTTP/1.1 200 OK
<
{
  "name" : "store1",
  "entity" : {
    "@id"      : "http://data.webofdata.io/publishing/newidentifier",
    "http://data.webofdata.io/schema/name" : "This store is very important"
  }
}
```

### Operation Create Dataset

A dataset is part of a store and acts as a container for entities. The optional `create-dataset` operation creates a new dataset by sending a POST request. The entity in a dataset is the subject for that dataset and MUST be a valid WoD entity.

``` yaml
  /stores/{store-name}/datasets:
    post:
      tags:
        - "Management"
      summary: Create a new dataset
      description: "Creates a new dataset with the name provided if and only if this name is not already taken in this named store. If no name is provided the server MUST generate one."
      operationId: "create-dataset"
      produces:
        - "application/json"
      parameters:
        - name: "store-name"
          in: "path"
          description: "unique store name for this service endpoint"
          required: true
          type: "string"
        - name: "body"
          in: "body"
          description: "Dataset to create"
          required: true
          schema:
            $ref: "#/definitions/Dataset"
      responses:
        201:
          description: "successful operation - dataset created"
          schema:
            $ref: "#/definitions/Dataset"
        404:
          description: "Store not found"
        400:
          description: "Occurs when the dataset passed in the body is invalid or missing"

```

The following example shows how to create a dataset with a POST request.

```
> POST /stores/store1/datasets HTTP/1.1
> Host: https://wod-api.example.com
>
{ 
  "name" : "dataset1", 
  "entity" : {
    "@id" : "http://data.webofdata.io/publishing/store1/people"      
  }
} 

<
< HTTP/1.1 201 CREATED
< Location: /stores/store1/datasets/dataset1
<
{ 
  "name" : "dataset1", 
  "entity" : {
    "@id" : "http://data.webofdata.io/publishing/store1/people"      
  }
}  

```

### Operation Get Datasets

The operation `get-datasets` returns a list of the datasets contained in the named store. 

``` yaml
/stores/{store-name}/datasets:
    get:
      tags:
        - "Management"
      summary: "Gets the datasets in the named store"
      description: "Returns a list of dataset objects or 404 if the store does not exist"
      operationId: "get-datasets"
      produces:
        - "application/json"
      parameters:
        - name: "store-name"
          in: "path"
          description: "Unique store name for this service endpoint"
          required: true
          type: "string"
      responses:
        200:
          description: "successful operation"
          schema:
            type: "array"
            items:
              $ref: "#/definitions/Dataset"
        404:
          description: "Store not found"

```

The following example shows how to retrieve the datasets of a store.

```
> GET /stores/store1/datasets HTTP/1.1
> Host: api.webofdata.io
>

< HTTP/1.1 200 OK
[
  {
    "name" : "dataset1",
    "entity" : {
      "@id" : "http://data.webofdata.io/publishing/store1/people"
    }
  },
  {
    "name" : "dataset2", 
    "entity" : {
      "@id" : "http://data.webofdata.io/publishing/store1/products"
    }
  }
]  
```

### Operation Get Dataset

Get a specific dataset by name. 

``` yaml
  /stores/{store-name}/datasets/{dataset-name}:
    get:
      tags:
        - "Management"
      summary: "Gets a list of datasets in the named store"
      description: "Returns a single dataset object or 404 if the store or dataset does not exist"
      operationId: "get-dataset"
      produces:
        - "application/json"
      parameters:
        - name: "store-name"
          in: "path"
          description: "store name"
          required: true
          type: "string"
        - name: "dataset-name"
          in: "path"
          description: "dataset name"
          required: true
          type: "string"
      responses:
        200:
          description: "successful operation"
          schema:
            $ref: "#/definitions/Dataset"
        404:
          description: "Invalid store name or dataset name"

```

The following example shows how to retrieve a named dataset.

```
> GET /stores/store1/datasets/dataset1 HTTP/1.1
> Host: api.webofdata.io
>
<

< HTTP/1.1 200 OK
{
  "name" : "dataset1",
  "entity" : {
    "@id" : "http://data.webofdata.io/publishing/store1/people"
  }
}
```

### Operation Update Dataset

Updates the entity for the specified dataset.

``` yaml
  /stores/{store-name}/datasets/{dataset-name}:  
    put:
      tags:
        - "Management"
      summary: Update dataset
      description: "Update the entity for the dataset"
      operationId: "update-dataset"
      produces:
        - "application/json"
      parameters:
      - name: "store-name"
        in: "path"
        description: "store name"
        required: true
        type: "string"
      - name: "dataset-name"
        in: "path"
        description: "dataset name"
        required: true
        type: "string"
      - name: "body"
        in: "body"
        description: "New dataset entity data"
        required: true
        schema:
          $ref: "#/definitions/Entity"
      responses:
        200:
          description: "successful operation - dataset updated"
          schema:
            $ref: "#/definitions/Dataset"
        400:
          description: "Occurs when the dataset passed in the body is invalid or missing"
        404:
          description: "Store or dataset not found"
```

The following example shows how to update the entity of a dataset with a PUT request.

```
> PUT /stores/store1/datasets/dataset1 HTTP/1.1
> Host: api.webofdata.io
>
{ 
    "@context": {
        "namespaces" : {
            "test" : "http://example.org/"
        }
    },
    "@id"       : "http://data.webofdata.io/publishing/store1/people",
    "test:name" : "This dataset is very important"
}

< HTTP/1.1 200 OK
<
{ 
  "name" : "dataset1", 
  "entity" : {
    "@context": {
        "namespaces" : {
            "ns1" : "http://example.org/"
        }
    },
    "@id"       : "http://data.webofdata.io/publishing/store1/people",
    "ns1:name" : "This dataset is very important"      
  }
}   
 
```

### Operation Delete Dataset

A request to delete a dataset. MUST delete the dataset entity and all contained entities.

``` yaml
  /stores/{store-name}/datasets/{dataset-name}:  
    delete:
      tags:
        - "Management"
      summary: Delete dataset
      description: "Delete the dataset"
      operationId: "delete-dataset"
      parameters:
      - name: "store-name"
        in: "path"
        description: "store name"
        required: true
        type: "string"
      - name: "dataset-name"
        in: "path"
        description: "dataset name"
        required: true
        type: "string"
      responses:
        200:
          description: "successful operation - dataset deleted"
        404:
          description: "Store or dataset not found"
```

The following example shows how to delete a dataset.

```
> DELETE /stores/store1/datasets/dataset1 HTTP/1.1
> Host: api.webofdata.io
>

< HTTP/1.1 200 OK
<
```

### Operation Get Entities

A dataset contains entities. The complete set of entities (optionally including those marked as deleted) in a dataset can be retrieved with the 'get-entities' operation. Responses to this request are free to stream all entities or return pages of entities and a link to the next page.

This endpoint can also be used to retrieve a specific entity from the dataset. A specific entity can be retrieved by providing an identifier as a query parameter whose value is the URI identifier of the entity.

As responses to this request can be large and hard for the server to determine the size of the response in advance a server MAY choose to return pages of data or can stream all the data using a multi-part response.

If a response does not contain data for all entities then the last entity in the array must be a special entity that contains a continuation token. This entity must have the id of '@continuation', and also a key 'next'. The value of 'next' is a string value that can be passed to subsequent calls to 'get-entites' via the 'token' query parameter.

```
{
    "@id" : "@continuation",
    "next" : "some value that allows the server to return more data"
}
```

``` yaml

  /stores/{store-name}/datasets/{dataset-name}/entities:
    get:
      tags:
        - "Management"
      summary: "Get entities in the dataset"
      description: "Returns all the entities in the specified dataset. Can also be used to locate a specific entity using the id query parameter."
      operationId: "get-entities"
      produces:
        - "application/json"
      parameters:
        - name: "store-name"
          in: "path"
          description: "store name"
          required: true
          type: "string"
        - name: "dataset-name"
          in: "path"
          description: "dataset name"
          required: true
          type: "string"
        - name: "id"
          in: "query"
          type: "string"
          description: "If provided it must be a full uri and it will limit the number of entities to either 0 or 1, depending if an entity with that id exists in the dataset"
        - name: "nextdata"
          in: "query"
          type: "string"
          description: "A token used to retrieve more data. Tokens are provided as a result of calling get-entities-partitions or as a value in the continuation entity of a response to this operation."
      responses:
        200:
          description: "successful operation"
          schema:
            type: "array"
            items:
              $ref: "#/definitions/Entity"
        404:
          description: "Store or dataset not found"

```

The following example shows how to use the 'get-entities' operation.

```
> GET /stores/store1/datasets/dataset1/entities HTTP/1.1
> Host: api.webofdata.io
>

< HTTP/1.1 200 OK
[
  {
    "@id" : "@context",
    "namespaces" : {
      "products" : "http://examples.org/products/"
    }
  },
  {
    "@id" : "products:1",
    "@props" : {
        "products:name" : "product 1"
    }
  },
  {
    "@id" : "products:2",
    "@props" : {
      "products:name" : "product 2"
    }
  }
]  
```

### Operation Store Entities

The store entities operation lets a client send a collection of entity objects that the server MUST either add, update or delete in the dataset specified.

If the entity specified exists then the current representation must be replaced with the new one. If the entity does not exist a new one is added. If the entity provided is marked with the '@deleted' property (with a value of 'true') then the entity must be marked as deleted in the dataset. If the entity exists in the dataset and is identical to the one being provided the server MAY choose not to update it and not to mark it as having changed.

If entities are deleted the server must keep track of this so that the entity can appear in a get-changes response. When the entity appears in this response it MUST be marked as deleted.

If an entity contains a property called '@etag' then the server can use the value of this property to decide the specified entity has been modified since this representation was retreived. If this etag value does not match that of the one in the dataset then the update is rejected.

If the etag is omitted or the 'x-wod-ignore-etags' header has a value of 'true' then the server should ignore any etag value conflicts and apply the updates.

``` yaml
  /stores/{store-name}/datasets/{dataset-name}/entities:
    post:
      tags:
        - "Management"
      summary: "Add, replaces or deletes entities in the dataset"
      description: "Adds, replaces or deletes all the entities in the specified dataset with those provided in the body."
      operationId: "store-entities"
      produces:
      - "application/json"
      parameters:
      - name: "store-name"
        in: "path"
        description: "store name"
        required: true
        type: "string"
      - name: "dataset-name"
        in: "path"
        description: "dataset name"
        required: true
        type: "string"
      - name: "entities"
        in: "body"
        schema:
          type: "array"
          items:
            $ref: "#/definitions/Entity"
      responses:
        200:
          description: "successful operation"

```

The following example shows how to upload several entities to be stored in a dataset.

```
> POST /stores/store1/datasets/dataset1/entities HTTP/1.1
> Host: api.webofdata.io
>
[
  {
    "@id" : "@context",
    "namespaces" : {
      "products" : "http://examples.org/products/"
    }
  },
  {
    "@id" : "products:1",
    "@props" : {
        "products:name" : "product 1"
    }
  },
  {
    "@id" : "products:2",
    "@props" : {
      "products:name" : "product 2"
    }
  }
] 

< HTTP/1.1 200 OK
<
```


### Operation Delete Entities

The delete entities operation lets a client request that all entities in a dataset are deleted. The dataset itself is not deleted, neither is the dataset entity object. A request for changes to this dataset after this operation has completed MAY either return all entities marked as '@deleted' = true, or the x-wod-full-sync header would be set to true and no data would be returned.

``` yaml
  /stores/{store-name}/datasets/{dataset-name}/entities:
    delete:
        tags:
          - "Management"
        summary: "Delete all entities in the dataset"
        description: "Deletes all entities in the dataset but not the dataset itself"
        operationId: "delete-entities"
        produces:
          - "application/json"
        parameters:
          - name: "store-name"
            in: "path"
            description: "store name"
            required: true
            type: "string"
          - name: "dataset-name"
            in: "path"
            description: "dataset name"
            required: true
            type: "string"
        responses:
          200:
            description: "successful operation"
```

The following example shows how to delete the entities in a dataset.

```
> DELETE /stores/store1/datasets/entities HTTP/1.1
> Host: api.webofdata.io
>

< HTTP/1.1 200 OK
<
```

### Operation Get Entities Partitions

To allow the entities of a dataset to be fetched in parallel, a dataset MAY offer the 'get-partitions' endpoint. This endpoint returns a list of tokens that can then be used to consume the dataset in parallel. The mechanism for retrieving a single partition is identical to that of making a call to the 'get-entities' operation except that the 'nextdata' query parameter is either a value from the result of calling 'get-entities-partitions' or the x-wod-next-data header in a response from 'get-entitites'.

ED: FIX ME - use continuation and not x-wod-next-data-header

``` yaml
 /stores/{store-name}/datasets/{dataset-name}/entities/partitions:
    get:
      tags:
        - "Management"
      summary: "Get partitions for reading the dataset in parallel"
      description: "Returns a list of tokens to be used with the x-wod-next-data query parameter of the get-entities operation."
      operationId: "get-partitions"
      produces:
      - "application/json"
      parameters:
      - name: "store-name"
        in: "path"
        description: "store name"
        required: true
        type: "string"
      - name: "dataset-name"
        in: "path"
        description: "dataset name"
        required: true
        type: "string"
      - name: "partitions"
        in: "query"
        type: "string"
        description: "If provided it indicates to the server the desired number of partitions. However, the server is free to decide the partition count."
      responses:
        200:
          description: "successful operation"
          schema:
            $ref: "#/definitions/EntityPartitions"
        404:
          description: "Store or dataset not found"

```

The following example shows how to retrieve partition tokens to allow a client to read the dataset entities in parallel.

```

> GET /stores/store1/datasets/entities/partitions HTTP/1.1
> Host: api.webofdata.io
>
< HTTP/1.1 200 OK
<
[
    "partition:0:0",
    "partition:1:0",
] 

```

And then a client can make calls such to retrieve the data from one partition. The response may contain an x-wod-next-data header. The value of this header should be used to retrieve subsequent data from this partition as the value of the `nextdata` query parameter. 

```

> GET /stores/store1/datasets/entities?nextdata=partition:0:0 HTTP/1.1
> Host: api.webofdata.io
>
< HTTP/1.1 200 OK
[
  { 
    "@id" : "@context", 
    "namespaces" : {
      "products" : "http://examples.org/products/"   
    }
  },
  {
    "@id" : "products:1",
    "producst:name" : "product 1"      
  }
]  

```

## Data Synchronisation

The data synchronisation part of the protocol allows a client to keep a local copy of a dataset in-sync with a remote copy. It defines the service endpoint that a WoD server MUST offer and the semantics a compliant client MUST implement.

The goal of the synchronisation protocol is to provide clean and simple semantics that can be implemented easily and quickly to promote the sharing of datasets. These mechanisms enable complete datasets to be shared easily, automatically and incrementally.

### Operation: Get Changes

The 'get-changes' operation exposes a list of the entities that have changed in the specified dataset. Complete entity representation MUST be delivered back to the client.

This operation exposes the changes that occur to a dataset. Even though an entity may have changed many times the server is not required to return all these unique states. The server is only required to return the latest representation of the entity.

The last entity in the array must be a special entity that contains a continuation token. This entity must have the id of '@continuation', and also the key 'next'. The value of 'next' is a string value that can be passed to subsequent calls to 'get-changes' via the 'token' query parameter.

```
{
    "@id" : "@continuation",
    "next" : "token used to fetch next changes"
}
```


``` yaml
/stores/{store-name}/datasets/{dataset-name}/changes:
    get:
      tags:
        - "Synchronisation"
      summary: "Get dataset changes"
      description: "Returns all the changes in the specified dataset."
      operationId: "get-changes"
      produces:
      - "application/json"
      parameters:
      - name: "store-name"
        in: "path"
        description: "store name."
        required: true
        type: "string"
      - name: "dataset-name"
        in: "path"
        description: "dataset name."
        required: true
        type: "string"
      - name: "nextdata"
        in: "query"
          description: "A token that the service interprets in order to only return changes that have occurred since that point. These or tokens are provided by a call to get-changes-partitions or as the value in the continuation entity in responses to this operation."
        type: "string"
      responses:
        200:
          description: "successful operation"
          headers:
            x-wod-change-count:
              description: if present indicates the number of changes that server has at this point in time in the specified dataset.
            x-wod-full-sync:
              description: if present indicates that a full sync is required. This means that all local data should be deleted and the new data arriving put in its place.
              type: boolean
              default: false
          schema:
            type: "array"
            items:
              $ref: "#/definitions/Entity"
        400:
          description: "Invalid nextdata token"
        404:
          description: "Store or dataset not found"

```

The following example describes an interaction between a client and a server. The client requests changes, updates a local store or system with the new entity representations, stores the wod:next-data value, and then, later on, makes a get request that uses the stored wod:next-data value. The actual value of the wod:next-data value is not defined, and up to the server.

```
> GET /stores/store1/datasets/dataset1/changes HTTP/1.1
> Host: api.webofdata.io
>

< HTTP/1.1 200 OK
[
  {
    "@id" : "@context",
    "namespaces" : {
      "products" : "http://examples.org/products/"
    }
  },
  {
    "@id" : "products:1",
    "@props" : {
        "products:name" : "product 1"
    }
  },
  {
    "@id" : "products:2",
    "@props" : {
      "products:name" : "product 2"
    }
  },
  {
    "@id" : "@continuation",
    "next" : "offset:2"
  }
]  

```

A client would store the value of `next`, in this case `offset:2` and use it in a subsequent call. Such as:

```
> GET /stores/store1/datasets/dataset1/changes?nextdata=offset:2 HTTP/1.1
> Host: api.webofdata.io
>

< HTTP/1.1 200 OK
[
  {
    "@id" : "@context",
    "namespaces" : {
      "products" : "http://examples.org/products/"
    }
  },
  {
    "@id" : "products:3",
    "@props" : {
      "products:name" : "product 3"
    }
  },
  {
    "@id" : "@contiuation",
    "next" : "offset:3"
  }
]
```

#### X-WOD-FULL-SYNC HEADER

The response can include a `X-WOD-FULL-SYNC` header. The header can have one of the following values:

  - false (default)

    Meaning that the entities in the list can replace (update or mark as deleted) the subject representations the client already has stored.

  - true

    Meaning that the client should delete all local data and replace it with what it receives. This value is true when a client first requests changes, and can also occur when the dataset on the server is deleted and re-populated. It MUST only appear in the first response, even if the data to re-populate is made available over many requests.
    
### Deleted Subject Representations

The server can include deleted entities in the response. They have the following representation:

``` json
  [
    {
      "@id"      : "...",
      "@deleted" : true
    }
  ]
```

The "@id" property and the "@deleted" property MUST be included. The server MAY provide the rest of the deleted entity data.

### Client Semantics

A client is assumed to be storing, and keeping in sync, a local copy of a remote dataset. The actual form of the local storage is left to the client to decide. It could be updating another WoD service endpoint, or writing into a database.

If the client has not yet synchronised the dataset then it must first use the 'get-changes' operation to obtain the initial data. 

The response to this request will be an array of entities where the last entity has the id `@continuation` which has a data property of `next`. The value of this property MUST be used as the value of the `nextdata` query paramter in a subsequent request.

The client stores the entities provided in the array into a local dataset. For each entity the client deletes the local representation and replaces it with the new one provided. If the entity provided is marked as deleted then the client must delete the local copy. Clients should not update local entities unless there has been an actual change. If there is no difference between the local representation and the one provided in the entity then the client is free to not modify the local representation.

The client MUST store the value of `next` property only after it has committed the changes indicated by the entities in the response.

A client can then keep making calls and then storing the value in the `next` property until no more data is available.

At subsequent further intervals a client should use the stored `next` value and a requst to 'get-changes' to determine if there are any changes to the dataset.

Clients are free to use their own schedule for calling 'get-changes'.

When a client receives the `X-WOD-FULL-SYNC` header and it has a value of `true` then the client MUST delete all content from the local dataset and replace it with what comes from the server.

### Operation Get Change Partitions

To allow the changes to a dataset to be processed in parallel a dataset MAY offer the `get-changes-partitions` endpoint. This endpoint returns a list of tokens that can then be used to consume the changes to a dataset in parallel. 

A client calling this operation receives N tokens. Each token can be used as an initial `nextdata` parameter to `get-changes`.  

The endpoint is defined as follows:

``` yaml
  /stores/{store-name}/datasets/{dataset-name}/changes/partitions:
    get:
      tags:
        - "Synchronisation"
      summary: "Get partitions for reading the changes to a dataset in parallel"
      description: "Returns a list of tokens to be used with the x-wod-next-data query parameter of the get-changes operation."
      operationId: "get-changes-partitions"
      produces:
      - "application/json"
      parameters:
      - name: "store-name"
        in: "path"
        description: "store name"
        required: true
        type: "string"
      - name: "dataset-name"
        in: "path"
        description: "dataset name"
        required: true
        type: "string"
      - name: "count"
        in: "query"
        type: "string"
        description: "If provided it indicates to the server the desired number of partitions. However, the server is free to decide the partition count."
      responses:
        200:
          description: "successful operation"
          schema:
            type: "array"
            items:
              type: "string"
        404:
          description: "Store or dataset not found"

```

An example request showing a server returning a list of tokens. Note that the server is free to define the content of each token and a client should not try to deconstruct the token.  

```
> GET /stores/store1/datasets/changes/partitions HTTP/1.1
> Host: api.webofdata.io
>

< HTTP/1.1 200 OK
<
[
    "partition:0:0",
    "partition:1:0",
] 
```

These tokens can then be used to initiate calls to `get-changes` such as:

```
> GET /stores/store1/datasets/dataset1/changes?nextdata=partition:0:0 HTTP/1.1
> Host: api.webofdata.io
>

< HTTP/1.1 200 OK
[
  { 
    "@id" : "@context",
    "namespaces" : {
      "products" : "http://examples.org/products/"
    }
  },
  {
    "@id" : "products:1",
    "@props" : {
        "products:name" : "product 1"
    }
  },
  {
    "@id" : "products:2",
    "@props" : {
      "products:name" : "product 2"
    }
  },
  {
    "@id" : "@continuation",
    "next" : "partition:0:2"
  }
]  
```

When processing the changes of a dataset in parallel it is possible for any one of the client requests to a specific partition to receive the full-sync header with a value of true. In this case the client must stop all partitions that are processing changes for that dataset, delete the local dataset and then restart processing by requesting new partition tokens and then calling 'get-changes' with the new tokens.

## Query Operations

The WebOfData Query operations are designed to facilitate the retrieval of the representation of a given entity via its subject identifier, and the retrieval of those entities that reference (are connected to), a given entity.

The protocol is intended to work both in controlled (secure, closed networks) and open environments (on the web). It separates the use of URIs as identifiers from the use of URLs as references to resolvable resource representations.

Some WoD nodes are exposing datasets that require understanding by humans. These are often bootstrapping vocabularies or schema terms that are the basis for other applications. They are also starting points for further traversal in the web of data. The query interface provides a simple means to locate subject entities by name or identity.

## Query Operation

The query endpoint can take a variety of query parameters that allow a client to search for subjects or connected subjects. The datasets in which the search is performed can also be restricted by using the dataset query parameter. The endpoint and the allowed query parameters is defined as follows:

``` yaml
  /stores/{store-name}/query:
    get:
      tags:
        - "Query"
      summary: Queries across the datasets of the store using the query parameters provided.
      operationId: query-store
      parameters:
        - name: "store-name"
          in: "path"
          description: "unique store name for this service endpoint"
          required: true
          type: "string"
        - name: "subject"
          in: "query"
          required: false
          type: "string"
        - name: "connected"
          in: "query"
          required: false
          type: "string"
          description: "If provided indicates that the query should find all subjects connected to the identified subject. Value is either * or a URI."
        - name: "incoming"
          in: "query"
          required: false
          type: "boolean"
          description: "Used in combination with connected to indicate if the subject is the source or the target of the connections to be found. Default value is false."
        - name: "dataset"
          in: "query"
          required: false
          type: "string"
          description: "Repeatable parameter that indicates which datasets should be queried. If no values are specified then all datasets are used."
        - name: "nextdata"
          in: "query"
          required: false
          type: "string"
          description: "Can not be used with any other query parameters. The value of this query parameter should be values returned as the x-wod-next-data header from a prior query. The server is responsible for encoding into the token enough information to provide the next set of results."
        - name: "take"
          in: "query"
          required: false
          type: "integer"
          description: "A request to the server for the maximum number of items to returned. The server MAY ignore this."
      responses:
        200:
          description: "successful operation"
          headers:
            x-wod-next-data:
              description: "if present indicates to the client that there is more data to be retrieved. Use the value of this header as the value of the 'nextdata' query parameter to retrieve more data."
              type: "string"
          schema:
            type: "array"
            items:
              $ref: "#/definitions/Entity"
        404:
          description: "Store or dataset not found"

```


### Query for Entity by SubjectIdentifier

To query for a single entity by its subject identifier use the 'subject' query parameter. The following example shows how to locate a single entity in any dataset in the specified store. If the same entity exists in more than one dataset then the entities MUST be merged according to the merging semantics.

```
> GET /query?subject=http://data.webofdata.io/people/gra HTTP/1.1
> Host: api.webofdata.io
>

< HTTP/1.1 200 OK
{
    "@id" : "http://data.webofdata.io/people/gra",
    "@props" : {
      "http://data.webofdata.io/people/name" : "graham moore"
    }
}
```

### Query for Connected Subjects

To traverse a graph of connected subjects the 'connected' query parameter can be used. The value of this property can either be '*' to indicate all relationships or a URI that identifies the property type to use in traversing the graph. 

The starting point for the traversal is specified using the 'subject' query parameter. This identifies the starting point for the traversal. The value of this parameter MUST be a URI. 

The result of this query can be many results (e.g. when navigating from a type to it's instances or from a category to it's members.) The server is free to return a subset of the full results along with a token that can be used to fetch more data.

Any entities with the same subject identifier should be merged. 

The following example shows how to get all entities related (via outgoing connections) to the entity identified with a subject identifier.

```
> GET /query?subject=http://data.webofdata.io/people/gra&connected=* HTTP/1.1
> Host: api.webofdata.io
>
< HTTP/1.1 200 OK
{
    "@id" : "http://data.webofdata.io/schools/stbarts",
    "http://data.webofdata.io/schools/name" : "st barts"      
}
```

To traverse in towards the subject rather than away from it use the 'connected' query parameter to identify the property and the 'incoming' query parameter with a value of 'true' to indicate the subject should be the target of any connections and not the source.  

```
GET /query?connected-to=&lt;uri&gt;?by=&lt;uri&gt; =&gt; returns a list of subject representations that are connected with the given subject by incoming references of the specified property name.
```

Note that when querying the graph only relationships defined in the root of the entity are considered. The following entity would *not* match any outgoing or incoming queries as the only reference type is not at the root entity.

```
{ 
    "@id" : "@context", 
    "namespaces" : {
      "products" : "http://examples.org/products/"   
    }
  },
  {
    "@id" : "products:1",
    "products:name" : "product 1",
    "products:manager: {
        "products:company" : "&lt;http://data.webofdata.io/examples/bigone>"
    }      
  }
}
```

If the number of results to be returned is more than the server wishes to provide in one response or more than the client would like to receive then the server can return a partial result along with a token that allows the client to fetch more results. 

A client can request a number of results to return, but the server is free to ignore this. Servers typically ignore this request when the number is higher than the maximum number of results that the server is prepared to return.

The following example shows a client server interchange with the client requesting a small number of results and the server returning the requested number along with a continuation entity containing a 'wod:next-data' property whose value can then be used with the 'nextdata' query parameter to retreive more results. 

The 'nextdata' query token can not be used with any other query parameters.

The following example shows an initial request for related entities and a result 'pagesize' of one, and then a response, along with the 'x-wod-next-data' header for more data.

```
> GET /query?subject=http://data.webofdata.io/people/gra&connected=*&take=1 HTTP/1.1
> Host: api.webofdata.io
>
< HTTP/1.1 200 OK
[
  {
    "@id" : "@context",
    ...
  }
  {
    "@id" : "http://data.webofdata.io/schools/stbarts",
    "http://data.webofdata.io/schools/name" : "st barts"      
  },
  {
    "@id" : "@continuation",
    "@data" : {
      "wod:next-data" : "http://data.webofdata.io/people/gra::connected=*::take=1::from=1"
    }
  }
]

```

Then a subsequent request using the token provided. Note that tokens are opaque and a client should not try and generate or modify tokens. 

```
> GET /query?token=http://data.webofdata.io/people/gra::connected=*::take=1::from=1 HTTP/1.1
> Host: api.webofdata.io
>
< HTTP/1.1 200 OK
{
    "@id" : "http://data.webofdata.io/unis/southampton",
    "http://data.webofdata.io/schools/name" : "Southampton"      
}
```


### Query For Subject By Title and / or Description

As well as machine to machine communication WoD provides a way for humans to find subjects of interest by searching for them by name or description. This is most common when trying to find a starting point subject, or well known subject.

WoD defines two property types, 'wod:title' and 'wod:description'. The following search operation uses the values in these two properties to locate entities. The 'text' query parameter can only be used in conjunction with the 'dataset' query parameter.

The dataset query parameter can be repeated 0 or more times to restrict the dataset in which the search is performed. If it is not provided then the search occurs across all datasets.

A WoD node should search in all fields of all entities that are either wod:title or wod:description. Hits in wod:title should be prioritised over those in wod:description. The quality of the search results is left up to the implementation. 

The following example shows how
```
> GET /query?text=southampton HTTP/1.1
> Host: api.webofdata.io
>
< HTTP/1.1 200 OK
{
    "@id" : "http://data.webofdata.io/unis/southampton",
    "http://data.webofdata.io/schools/name" : "Southampton"      
}
```

### Linked Data URL Resolution

Follow-your-nose URL semantics works by a web endpoint capturing requests for subject representations and re-writing and re-routing them to the WoD server query endpoint. The following example shows how this should work:

```
http://data.webofdata.io/people/gra gets re-written as 

https://api.webofdata.io/query?subject=http://data.webofdata.io/people/gra
```

An example NGINX config and docker image are provided at https://github.com/webofdata/linked-data.

## Conformance

It is not possible to define conformance for a data model, conformance can only be defined in terms of observable actions and responses. 

The automated conformance test suite can be run against any WoD node that claims to implement the protocol and serialisation parts of this specification. 

The conformance client can be found online at http://github.com/webofdata/conformance. It includes source code and instructions for running the conformance test suite.

## References

<dl>
  <dt>HTTP
  <dd><a href="http://www.ietf.org/rfc/rfc2616.txt">RFC 2626</a>, HTTP
    1.1. September 2004.</li>
  <dt>Atom
  <dd><a href="http://www.ietf.org/rfc/rfc4287.txt">RFC  4287</a>,
    The Atom Syndication Format. December 2005.</li>
  <dt>RDF
  <dd><a href="http://www.w3.org/TR/rdf-concepts/">Resource
      Description Framework (RDF): Concepts and Abstract
      Syntax</a>, W3C Recommendation, February 2004.</li>
  <dt>NTriples
  <dd><a href="http://www.w3.org/TR/rdf-testcases/#ntriples"
         >RDF Test Cases</a>, W3C Recommendation, 10 February 2004.</li>
  <dt>RDF/XML
  <dd><a href="http://www.w3.org/TR/REC-rdf-syntax/">RDF/XML
      Syntax Specification (Revised)</a>, W3C Recommendation,
    10 February 2004.
  <dt>Atom Paging
  <dd><a href="http://www.ietf.org/rfc/rfc5005.txt"
         >RFC 5005</a>: Feed Paging and Archiving, IETF Standard,
    September 2007</li>
  <dt>RFC 2119
  <dd><a href="http://www.ietf.org/rfc/rfc2119.txt" >RFC 2119</a>: Key
         words for use in RFCs to Indicate Requirement Levels,
         IETF, March 1997.</li>
</dl>
