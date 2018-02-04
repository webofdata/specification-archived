
# Web of Data (WoD) Specification

Author  : Graham Moore  
Contact : gramoore (at) outlook (dot) com  
Version : 0.8  
This Document : [Specification 0.8](http://github.com/webofdata/specification/specification.md)  
Issues : <http://github.com/webofdata/specification/issues>

## Overview

Like HTML and HTTP did for machine to human communication, Web of Data (WoD) aims to provide the building blocks to allow any device or application to easily communicate, access, modify, navigate and share data at web scale. Enabling low friction, meaningful,standardised, machine-to-machine communication.

Web of Data defines a protocol, data model, and data representation format for publishing, sharing and connecting data on the web. It is intended to be easy to implement and work at a level of generality that is widely applicable.

The WoD data model has at it's core the notion of an entity. An entity has identity in the form of a URI. An entity also has properties whose 'names' or 'keys' are also URIs. The use of URIs to identify things and name properties provides a powerful, web scale approach to naming. The values of these properties can either be literals, based on XML schema data types, or complex types such as arrays, or another entity. Finally, there is a special reference type. This reference type has a value that is a URI. This URI references another entity to create a graph of entities that can span the web. 

The protocol is designed to facilitate the sharing, updating, creating and publishing of collections of entities, the retrieval of a given data entity, and the navigation of connected entities. It is expected that collections of entities can exists in databases, existing applications and dedicated WoD data stores. All kinds of applications can implement and support the protocol in meaningful ways, allowing a vast array of client applications to easily consume and use data from around the web. 

The WoD representation format uses JSON with a few well defined keys that support the use of URIs as identifiers and property names. 

Finally, WoD defines a protocol for synchronising datasets. This allows clients to collect and keep in sync core data from around the web to offer new services and provide guaranteed quality of service.

## Background

Several attempts have been made to define a standard and generic data model and related APIs; most notably OData and the RDF stack from W3C. While technically sound they both have significant barriers to adoption - The RDF data model is too shredded and lacks basic constructs such as list and map; OData has a data model that doesn't embrace URIs for identity or naming. The query semantics and models in both is unappealing to users, and too much work for implementors.  

WebOfData builds on the experience of these and other standards to propose a universal data management and data sharing API, coupled with a data model that works at web scale.

A key concept that underpins WebOfData is the notion of a 'subject'. A subject is any thing, about which, anything may be stated or asserted, by anyone. A subject is an abstract concept, it can be a person, a document on a file system, a row in a database, a book. 

Computer systems are all about building representations of these abstractions. Every application creates a model where the data structures are proxies, or stand-ins for their 'real life' counterpart.  

Standards such as RDF and Topic Maps realised that to create a global space in which anyone can talk about any thing, there needs to be an agreed way to bind the data object in a computer system to the subject it represents. They also realised that there can be many partial representations of the same subject. These partial representations may or may not be known at any given time. 

Using URIs for identifying subjects provides a globally unique and authorative mechanism to unambiguously name the subjects of discourse and the vocabularies used to describe them.

WebOfData uses the term 'entity' to be the data object that is a representation of a subject. The binding between these two things is the use of a URI as the identifier for a given subject and entity. Note that many entities can have the same identifier. If they do, then they can considered as partial representations of the same subject.  

The WoD APIs are refinement of things like OData, Linked Data Fragments and SDShare.

## Introduction

WebOfData defines a data model, a JSON serialistion for the data model, a data access protocol (Query), a data sharing protocol (Synchronisation) and a data management protocol (CRUD). Compliant implementations can choose which of the APIs they support, but must adhere to the rules and semantics of the data model and serialisation definition. 

We talk about the of implementions of these protocols and adherence to the serialisation representations to be WoD nodes. A WoD node implementation can 

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## Data Model Definition

The WebOfData data model can be thought of an Entity-Graph model. Entities are complex objects of keys with values. Where keys are URIs and values can be literals, lists or another entity. In addition there is a reference value type that points to (references) another entity. This allows entities to be connected together in a graph at global scale. 

The value of a reference type is a URI and not some local internal identifier. The power of this is that it can be resolved globally. It can be used to lookup the related entity from one or more API endpoints. WoD does not define where a client can resolve an reference value, but does define the capabilities that a WoD data access endpoint should support.

Given that each entity is only a partial representation of a subject the data model also defines how to merge two entities. (The algorithm can be applied recursively to merge multiple representations.)

As well as entities there are two levels of container types. The store and the dataset. A store consists of zero or more datasets, and each dataset is comprised of 0 or more entities. 

### Terms: 

<dl>
  <dt>Subject</dt>
  <dd>A concept, an idea, a thing; any thing, about which anything can be said. Note that a subject is not part of the data model, but a logically related concept.</dd>

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

<pre>
entity             := { id, members }

child-entity       := { id, members } | { members }

id                 := xsd:uri

members            := pair | pair, members

pair               := xsd:uri : value 

value              := entity | child-entity | subject-ref | xsd:string |
                      xsd:int | xsd:datetime | xsd:uri | xsd:double |       
                      xsd:float
                      
subject-ref        := xsd:uri

</pre>

One of the key things to note is that a root-entity must have a id. Descendent entities are not required to have an id. If a child entity has an id it is implied that the entity has been embedded into the structure, but also has identity in its own right. Only the identity of the root conveys the subject identity of the entity.

### DataSets

A dataset is a collection of entities. It provides a container where entities can be managed (created, deleted, updated, located). 

Datasets can contain as most 1 entity with any given subject identifier. This means that the subject identifier is the unique key for an entity in a dataset. But, an entity with the same subject identifier can exist in other datasets.

Splitting the store into datasets allows for more than one representation of the same subject (an entity with the same subject identifier as another entity) to exist in a single store. 

When resolving a request for a subject identifier against a store then by default the lookup is across all datasets. In this case it is possible for more than one entity with the same subject identifier to be located. When this occurs the entity representations should be merged as described below. A client can request that lookup only occurs in the context of one dataset. In this case, due to the fact that within a dataset there is exaclty 0 or 1 entity with a given subject identifier, at most 1 entity will be located. 

Formally, and in terms of the entity definition:

<pre>

dataset  := { id, entities }
entities := [] | entity, entities

</pre>

### Entity Store

An entity store is a collection of datasets. The store is defined as follows:

<pre>

store    := { id, datasets }
datasets := [] | dataset, datasets

</pre>

### Merging Entities

A client can (potentially) retrieve a partial subject representation for the same subject from many WoD service endpoints. An application or server may decide that even though the subject identifiers are different that the entities represent the same subject, and entities are allowed to express a belief that they are the same as one or more other subjects using the "wod:sameas" property. In all of these cases two or more entities are merged to create a more unified representation.  

The rules for merging two entity representations of the same subject are defined below. If more than two entities need to be merged, then the algorithm should be applied on the first two entities, then on the resulting merged entity and the next entity to be merged; and so on.
 
To merge two subject representations A and B the following algorithm should be applied:

  - Let A and B be the two representations to be merged.
  - Create a new empty representation, C.
  - The "id" property of C becomes the value of the "id" property of A.
  - For each property in A if there is no corresponding property in B add it to C.
  - For each property in A if there is a corresponding property in B then merge the values:
    - If either of A or B properties are a list then they are merged by flattening them into one list: e.g. [a, b, c][d, e] becomes [a,b,c,d,e], and [ {}, [], "d"], "x" becomes: [ {}, [], "d", "x" ]. 
    - For non list values a new list is created and the values from A and B are added. e.g. {}, "x" becomes: [ {}, "x" ]
    - Duplicates are removed.
  - All properties on B that are not present on A are added to C.
 

#### SameAs Resolution

The above merging semantics define the formalism for merging two entities with the same subject identifier. In addition, the sameas resolution rule defines how a WoD server can recursively merge a set of entities that declare themselves to be the same.

This behaviour is optional in that a server MAY offer this capability, and even in the case when it does then the client MUST specifically request it.

Given a subject identifier, find the entity that corresponds to that URI. Locate the 'wod:sameas' property and for each subject-ref value in the array look up and merge that entity according to the rules above.

Recursively apply this rule to each entity references by a wod:sameas value, but only process each entity once.

## Data Model JSON Serialisation

JSON is used as the serialisation format for the Data Model. It defines how an instance of the data model can be serialised as JSON. The serialisation does not use any extensions to the JSON language, a valid WoD Data Model JSON document is a valid JSON document, but it is not true that any JSON document is a valid WoD JSON document.

### Core Rules

The <b>id</b> property of the entity is serialised with the key <b>@id</b>.

The <b>subject-ref</b> URI values are serialised with <b><</b> as the first character and <b>></b> as the last character. The enclosing value is the string representation of a CURIE or URI.

While full URIs can be used in WoD Json documents as keys, values and id property values, it is optimal for machines and easier for humans if full URIs are replaced with CURIEs. Contexts definitions are used to define default namespaces, CURIEs. The datatypes of property values can also be defined in a context.

The following rules define how JSON data must be written in order to be considered valid WoD JSON.

#### The @id Property Rule

The <b>@d</b> property of the entity is serialised as key of <b>@id</b>. The value of this property defines the identifier for the entity. This property MUST be present on the root JSON object. The value of this property can be an full URI or a CURIE. If it is neither of these two then it is interpreted in relation to the default context (see context resolution).

The following examples show the different serialisations that are all semantically equivalent:

<pre>
  {
    "@id" : "http://data.webofdata.io/people/gra" 
    ...
  } 
</pre>

or

<pre>
  {
    "@context" : { "people" : "http://data.webofdata.io/people/" } ,
    "@id" : "people:gra",
    ... 
  } 
</pre>

or 

<pre>
  {
    "@context" : { "_" : "http://data.example.org/people/" } ,
    "@id" : "gra",
    ...
  } 
</pre>

The @id property can also be used on any descendant object. When used on a descendant object it gives identity to that object. It does not affect or override the identity of the root object.

This provides a way to embed or contain other subjects by value and still convey that they have identity in their own right. The following example demonstrates this usage.

<pre>
  {
    "@id" : "http://data.example.org/people/gra",
    "education" : [
      {
        "@id" : "http://data.schools.org/stbarts",
        "name" : "st bartholomews"
      },
      {
        "@id" : "http://data.universities.org/southampton",
        "name" : "Southampton University"   
      }
    ]
  }
</pre>

#### Subject Reference Rule

Entities can reference other subjects by their subject identifier. To represent a subject reference the '<' character MUST come at the start of the property value and '>' at the end. The value between '<' and '>' can be a full URI or a CURIE. If it is neither of these two then it is interpreted in relation to the default context (see context resolution).
 
<p>The following examples show how to use subject references: </p>

<pre>
{
  "@id" : "http://data.example.org/people/gra",
  "lives-in" : "<http://data.cities.org/oslo>"
}
</pre>

and: 

<pre>
{
  "@id" : "http://data.example.org/people/gra",
  "lives-in" : [ "<http://data.cities.org/oslo>", "<http://data.cities.org/oxford>" ]
}
</pre>

<pre>
{
  "@id" : "http://data.example.org/people/gra",
  "skills" : [
    { 
      "name"      : "csharp", 
      "category" : "<http://data.categories.org/programming>" 
    }
  ] 
}
</pre>

#### Property Naming Rule

As well as being able to talk about 'subjects' using subject identifiers, we also need to be able to agree on the properties being used. In WoD all property names are URIs. They can be expressed as full IRIs, Curies or simple values. If they are expressed as simple values they are resolved against the default namespace defined in a context.

An example with properties as IRIs. Both the 'foaf:name' and 'http://data.example.org/vocab/lives-in' are properties that will resolve to full IRIs.

<pre>
{
  "@id" : "http://data.example.org/people/gra",
  "@context" : { "foaf" : "http://foaf.org/schema" } ,
  "http://data.example.org/vocab/lives-in" : "<http://data.cities.org/oslo>",
  "foaf:name" : "Graham Moore"
}
</pre>


#### Context Rules

All property names, '@id' property values, and subject references values are full URIs. However, to simplify documents and avoid repeating long URI strings these values can be specified either as CURIEs or as simple values that are resolved against a context.

When a CURIE or simple value is used then they must be resolved against some context. A document collection or a single document can describe the context in which CURIEs and simple values are expanded.

A context is declared using the special '@context' property. The value of the '@context' property is a JSON object with two keys. The allowed keys are "namespaces" and "datatypes". 

The "namespaces" key has a value that is a JSON object. The keys in this object correspond to the first part of a CURIE and the value (which must be a string) is the expansion.

The "datatypes" key is used to define the datatypes of values based on their key. In general the datatypes are infered from the JSON data type. However, JSON data types are limited and this mechanism allows for a greater range of basic types, via XML Schema, and also for extension types to be introduced.

If a datatype is defined for a key then the value of that property MUST be either a string or a list of strings. 

<pre>
  {
    "@context" : {
        "namespaces" {
          "foaf" : "http://xmlns.com/foaf/0.1/"
        }, 
        "datatypes" : {
            "foaf:name"   : "xsd:string",
            "foaf:height" : "xsd:double",
            "foaf:dob"    : "xsd:datetime"
        }
    }
    
    "@id" : "http://data.webofdata.io/people/gra",
    "foaf:name" : "Graham Moore",
    "foaf:height" : "1.87",
    "foaf:age" : 23,
    "foaf:dob" : "1995-06-20"
  }
</pre>

As well as CURIEs the "namespaces" object can also contain the definition of a default namespace. The default namespace is used when no CURIE prefix is used and the value is not a full URI.

The default instance expansion is defined using the key of "_".

The following example shows the use of the default namespace and two semantically equivalent serialisations.

<pre>

{
  "@id" : "http://data.example.org/people/gra",
  "@context" : { "foaf" : "http://foaf.org/schema/"} ,
  "foaf:name" : "Graham Moore"
}

is the same as:

{
  "@id" : "http://data.example.org/people/gra",
  "@context" : { "_" : "http://foaf.org/schema/"} ,
  "name" : "Graham Moore"
}

In both examples the 'name' property is expanded to 'http://foaf.org/schema/name'

</pre>


Contexts can be defined as part of each entity serialisation, or when serialising a list of entities as the first object. It is allowed to define a context as the first entity of a list and as part of any following root entity. If there is both an array level context and an object level context, then any keys that are defined in the object take precedence.

If the context is defined as the first object in an array then the value of the "@id" property MUST be "@context". 

The following example shows a context definition defined as the first entity in the array.

  <pre>
  [
    {
      "@id" : "@context,
      "namespaces" : {
        "foaf" : "http://xmlns.com/foaf/0.1/"
      }
    },

    {
      "@id" : "...",
      "foaf:name" : "bob"
    },

    {
      "@id" : "..."
    }
  ]
</pre>

The context can also be defined as part of the entity serialisation, e.g:

<pre>
  {
    "@context" : {
      "foaf" : "http://xmlns.com/foaf/0.1/" 
    },
    "@id" : "http://data.example.org/people/gra",
    "http://data.example.org/vocab/lives-in" : "<http://data.cities.org/oslo>",
    "foaf:name" : "Graham Moore"
  }

  Expands to:

  {
    "@id" : "http://data.example.org/people/gra",
    "http://data.example.org/vocab/lives-in" : "<http://data.cities.org/oslo>",
    "http://xmlns.com/foaf/0.1/name" : "Graham Moore"
  }
</pre>

If a context is defined as the first object in an array, and in an object then the contexts are merged with the values in the object context taking precedence.

<pre>
  Given:
    [
        {
        "@id" : 
        "namespaces" :{
                "_"      : "http://data.example.org/people/"
                "people" : "http://data.example.org/people/"
            }
        },

        {
        "@id" : "gra",
        "@context" : {
            "namespaces" : {
                "people" : "http://data.example.org/person/"  
            }
        }
        "people:name" : "Graham Moore"   
        }
    ]

Resolves to:

    {
        "@id" : "http://data.example.org/people/gra",  # from default namespace
        "http://data.example.org/person/name" : "Graham Moore" # from people expansion definition in the entity  
    }
</pre>

Datatype Support in property expansions. In addition to the simple form of:

<pre>
    "&lt;key&gt;" : "&lt;namespace-prefix&gt;"
</pre>

The following form is allowed:

<pre>
    &lt;key&gt; : {
      "@id"   : "&lt;namespace-prefix&gt;",
      "@type" : "xsd:string"  
    }
</pre>

## Web of Data Protocol

The WoD protocol consists of a service definition and the related semantics for each operation. The operations are described in prose and then formalised using swagger. The protocol is split into three distinct areas; data management, data query, and data synchronisation. 

## Common Definitions

The following type definitions are used and referenced throughout the protocol description. They are listed here just once.

<pre>

definitions:

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
      "@context":
        "$ref": "#/definitions/Context"
      "@etag":
        type: "string"

  Context:
    type: "object"
    properties:
      namespaces:
        type: "object"
      datatypes:
        type: "object"

  Transaction:
    type: "object"
    properties:
      id:
        type: "string"
      operations:
        type: array
        items:
          $ref: "#/definitions/TransactionOperation"

  TransactionOperation:
    type: "object"
    properties:
      dataset:
        type: "string"
        description: "The name of the dataset to update entities in"
      body:
        type: array
        items:
          $ref: "#/definitions/Entity"

  TransactionInfo:
    type: "object"
    properties:
      id:
        type: "string"
      status:
        type: "string"

</pre>

## Data Management

The data management part of the protocol defines operations for the creation and deletion of stores and datasets, and the updating of entities within a dataset. It also defines operations for creating and monitoring transactions that span datasets.


### Operation Get Service 

This operation returns information about the service endpoint itself. 

### Operation Get Stores

This operation returns a list of stores being managed by the WoD node. The list of stores can be filtered by providing the id of the store as a query parameter. Note that stores have a name that forms part of the URI, but also has a metadata entity that represents the store. This entity has an id in the form of a URI and it is this value that can be used to filter the list of stores.

<pre>

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

</pre> 

The following example shows a request that returns three stores.

<pre>

HOST https://wod-api.example.com
GET /stores

[
  { "name" : "store1", 
    "entity" : {
      "@id" : "http://data.webofdata.io/publishing/store1"      
    }
  },
  { "name" : "store2", 
    "entity" : {
        "@id" : "http://data.webofdata.io/publishing/store2"      
    }
  },
  { "name" : "store3", 
    "entity" : {
        "@id" : "http://data.webofdata.io/publishing/store3"      
    }
  }  
]

</pre>

The following example shows a request that returns just one store. Only one store is returned as the query parameter is used to locate the store whose entity id matches the value provided.

<pre>

HOST https://wod-api.example.com
GET /stores?id=http://data.webofdata.io/publishing/store1

[
  { "name" : "store1", 
    "entity" : {
      "@id" : "http://data.webofdata.io/publishing/store1"      
    }
  }  
]

</pre>

### Operation Create Store

The create store operation expects a JSON object to be sent in the body of the request. The JSON object may contain a 'name' property and MUST contains an 'entity' property. The value of the 'entity' property MUST be a JSON object that is valid according to the WoD JSON specification. If the 'name' property is present it MUST contain a string value. 

The server rejects the request if the store name is already in use, or if the id of the entity is the same as any existing store.

If no name is provided the server MUST generate one.

The operation is defined as follows:

<pre>

/stores:
    post:
      tags:
        - "Management"
      summary: Create a new store
      description: "Creates a new store with the name provided, if and only f this name is not already taken. If no name is provided then the server MUST generate one"
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
            $ref: "#/definitions/Entity"
        400:
          description: "Occurs when the store passed in the body is invalid or missing"

</pre>

The following example shows the create store request:

<pre>

HOST https://wod-api.example.com
POST /stores

{ 
  "name" : "store1", 
  "entity" : {
    "@id" : "http://data.webofdata.io/publishing/store1"      
  }
}  

Response =>

201 CREATED
{ 
  "name" : "store1", 
  "entity" : {
    "@id" : "http://data.webofdata.io/publishing/store1"      
  }
}  

</pre>

The following example shows a valid create store request where the server generates the store name.

<pre>

HOST https://wod-api.example.com
POST /stores

{ 
  "entity" : {
    "@id" : "http://data.webofdata.io/publishing/store1"      
  }
}  

Response =>

201 CREATED
{ 
  "name" : "someguid", 
  "entity" : {
    "@id" : "http://data.webofdata.io/publishing/store1"      
  }
}  

</pre>

### Operation Get Store

The 'get-store' operation retreives the store entity using the local store name.

<pre>

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

</pre>

The following example shows the use of this operation:

<pre>

HOST https://wod-api.example.com
GET /stores/store1

[
  { "name" : "store1", 
    "entity" : {
      "@id" : "http://data.webofdata.io/publishing/store1"      
    }
  }  
]

</pre>

### Operation Delete Store

The 'delete-store' operation requests that the implementing WoD node deletes the store and all datasets and metadata associated with it. This include the entity for the store itself.

<pre>

  /stores/{store-name}:
    delete:
      tags:
        - "Management"
      summary: delete the named store, and all datasets
      description: deletes the store or returns a 400 bad request if no store of the name provided exists
      operationId: delete-store
      parameters:
        - name: "store-name"
          in: "path"
          description: "store name"
          required: true
          type: "string"

</pre>

The following example shows how to request that a store is deleted.

<pre>

==== Request

HOST https://wod-api.example.com
DELETE /stores/store1

==== Response

200 OK

</pre>


### Operation Update Store Entity

The entity for a store can be updated by using a POST request. The body of the request contains the new representation for the store entity.

<pre>

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
            $ref: "#/definitions/Store"
      responses:
        200:
          description: "successful operation"
          schema:
            $ref: "#/definitions/Store"      

</pre>

The following example shows how to update the entity for a store. Note in this example that the server renames the namespace prefix. A server is free to rename or map prefixes as it sees fit. What must hold true is the fully expanded URI. 

<pre>

HOST https://wod-api.example.com
PUT /stores/store1

{ 
  "name" : "store1", 
  "entity" : {
    "@context": {
        "namespaces" : {
            "test" : "http://example.org/"
        }
    },
    "@id"       : "http://data.webofdata.io/publishing/newidentifier",
    "test:name" : "This store is very important"      
  }
}  

Response =>

200 OK
{ 
  "name" : "store1", 
  "entity" : {
    "@context": {
        "namespaces" : {
            "ns1" : "http://example.org/"
        }
    },
    "@id"      : "http://data.webofdata.io/publishing/newidentifier",
    "ns1:name" : "This store is very important"      
  }
}  
 
</pre>


### Operation Create Dataset

A dataset is part of a store, and acts as a container for entities. A dataset can be created by sending a POST request. 

<pre>

  /stores/{store-name}/datasets:
    post:
      tags:
        - "Management"
      summary: Create a new dataset
      description: "Creates a new dataset with the name provided iff this name is not already taken. If no name is provided the server MUST generate one."
      operationId: "create-dataset"
      produces:
        - "application/json"
      parameters:
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
        400:
          description: "Occurs when the dataset passed in the body is invalid or missing"

</pre>

The following example shows how to create a dataset with a POST request.

<pre>

HOST https://wod-api.example.com
POST /stores/store1/datasets

{ 
  "name" : "dataset1", 
  "entity" : {
    "@id" : "http://data.webofdata.io/publishing/store1/people"      
  }
}  

Response =>

201 CREATED
{ 
  "name" : "dataset1", 
  "entity" : {
    "@id" : "http://data.webofdata.io/publishing/store1/people"      
  }
}  

</pre>

### Operation Get Datasets

The operation ´get-datasets' returns a list of the datasets contained within the named store. 

<pre>

/stores/{store-name}/datasets:
    get:
      tags:
        - "Management"
      summary: gets a list of datasets in the named store
      description: returns a single store object or 404 if the store does not exist
      operationId: get-datasets
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
            type: "array"
            items:
              $ref: "#/definitions/Dataset"
        400:
          description: "Invalid store name"

</pre>

The following example shows how to retreive the datasets of a store.

<pre>

HOST https://wod-api.example.com
GET /stores/store1/datasets

Response =>

200 OK
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

</pre>

### Operation Get Dataset

Get the entity for a given dataset.

<pre>

  /stores/{store-name}/datasets/{dataset-name}:
    get:
      tags:
        - "Management"
      summary: gets a list of datasets in the named store
      description: returns a single store object or 404 if the store does not exist
      operationId: get-datasets
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

</pre>

The following example shows how to retrieve a dataset entity.

<pre>

HOST https://wod-api.example.com
GET /stores/store1/datasets/dataset1

Response =>

200 OK
{ 
  "name" : "dataset1", 
  "entity" : {
    "@id" : "http://data.webofdata.io/publishing/store1/people"      
  }
}

</pre>

### Operation Update Dataset

Updates the entity for the specified dataset.

<pre>

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
          $ref: "#/definitions/Dataset"
      responses:
        200:
          description: "successful operation - dataset updated"
          schema:
            $ref: "#/definitions/Dataset"
        400:
          description: "Occurs when the dataset passed in the body is invalid or missing"

</pre>

The following example shows how to update the entity of a dataset with a PUT request.

<pre>

HOST https://wod-api.example.com
PUT /stores/store1/datasets/dataset1

{ 
  "name" : "dataset1", 
  "entity" : {
    "@context": {
        "namespaces" : {
            "test" : "http://example.org/"
        }
    },
    "@id"       : "http://data.webofdata.io/publishing/store1/people",
    "test:name" : "This dataset is very important"      
  }
}  

Response =>

200 OK
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
 
</pre>

### Operation Merge Dataset

Starts a merges operation from a named dataset (dataset B) into the one addressed (dataset A). The semantics of this operation requires:
  - that all entities that are present in B and not marked as deleted replace those in A that already exist with the same id, or are added to A. 
  - that all entities marked as deleted in B are marked as deleted in A.
  - that all entities that exist in A but are not in B are marked as deleted.

While the operation is in progress any writes to A should be stored and only applied after the merge has completed. Any writes to B MUST be rejected.

Support for this operation MAY be provided by a server. If the server doesn't offer the merge service then it should return a operation not supported response when a client attempts to POST the merge operation resource.

<pre>

  /stores/{store-name}/datasets/{dataset-name}/merge: 
    post: 
     tags:
        - "Management"
      summary: Merge dataset
      description: "Merge all entities from specified dataset into this one, delete any in this one that are not in the other."
      operationId: "merge-dataset"
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
        description: "Merge operation parameters"
        required: true
        schema:
          $ref: "#/definitions/MergeOperation"
      responses:
        200:
          description: "successful operation - dataset updated"
          schema:
            $ref: "#/definitions/Dataset"
        400:
          description: "Occurs when the dataset passed in the body is invalid or missing"

</pre>

The following example shows how to update the entity of a dataset with a PUT request.

<pre>

HOST https://wod-api.example.com
PUT /stores/store1/datasets/dataset1

{ 
  "name" : "dataset1", 
  "entity" : {
    "@context": {
        "namespaces" : {
            "test" : "http://example.org/"
        }
    },
    "@id"       : "http://data.webofdata.io/publishing/store1/people",
    "test:name" : "This dataset is very important"      
  }
}  

Response =>

200 OK
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
 
</pre>


### Operation Delete Dataset

A request to delete a dataset MUST delete the dataset entity and all contained entities.

<pre>

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
</pre>


The following example shows how to delete a dataset.

<pre>

HOST https://wod-api.example.com
DELETE /stores/store1/datasets/dataset1

=> response
200 OK

</pre>


### Operation Get Entities

A dataset contains entities. The complete set of entities in a dataset can be retrieved with the 'get-entities' operation. Responses to this request are free to stream all entities or return pages of entities and a link to the next page. 

This endpoint can also be used to retrieve a specific entity from the dataset. A specific entity can be retrieved by providing an identifier as a query parameter whose value is the URI identifier of the entity.

<pre>

  /stores/{store-name}/datasets/{dataset-name}/entities:
    get:
      tags:
        - "Management"
      summary: "Get entities in the dataset"
      description: "Returns the entities in the specified dataset. Can also be used to locate a specific entity using the id query parameter."
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
      - name: "nextdata"
        in: "query"
        description: "To be used with values provided by the x-wod-next-data header value. If omitted it indicates that the client wishes to retrieve all data." 
      - name: "id"
        in: "query"
        description: "If provided it must be either a full uri or curie and it will limit the number of entities to either 0 or 1, depending if an entity with that id exists in the dataset. Cannot be used with the 'nextdata' parameter."
        type: "string"
      responses:
        200:
          description: "successful operation"
          headers:
            x-wod-next-data:
              description: "if present indicates to the client that there is more data to be retreived. Use the value of this header as the value of the 'nextdata' query parameter to retreive more data."
              type: "string"
          schema:
            type: "array"
            items:
              $ref: "#/definitions/Entity"

</pre>

The following example shows how to use the 'get-entities' operation.

<pre>

HOST https://wod-api.example.com
GET /stores/store1/datasets/entities

Response =>

200 OK
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
  },
  {
    "@id" : "products:2",
    "producst:name" : "product 2"      
  }
]  

</pre>

### Operation Store Entities

The store entities operation lets a client send a collection of entity objects that the server MUST either add, update or delete in the dataset specified.

If the entity specified exists then the current representation must be replaced with the new one. If the entity does not exist a new one is added. If the entity provided is marked with the '@deleted' property (with a value of 'true') then the entity must be deleted from the dataset. If the entity exists in the dataset and is identical to the one being provided the server MAY choose not to update it and not to mark it as having changed.

If entities are deleted the server must keep track of this so that the entity can appear in a get-changes response. When the entity appears in this response it MUST be marked as deleted.

<pre>

  /stores/{store-name}/datasets/{dataset-name}/entities:
    post:
      tags:
        - "Management"
      summary: "Add or replace entities in the dataset"
      description: "Adds or replaces all the entities in the specified dataset with those provided in the body."
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

</pre>

The following example shows how to upload several entities to be stored in a dataset.

<pre>

</pre>


### Operation Delete Entities

The delete entities operation lets a client request that all entities in a dataset are deleted. The dataset itself is not deleted, neither is the dataset entity object. A request for changes to this dataset would return all entities, and they would be marked as '@deleted' = true.

<pre>
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
</pre>

### Operation Get Entities Partitions

To allow the entities of a dataset to be fetched in parallel a dataset MAY offer the 'get-partitions' endpoint. This endpoint returns a list of tokens that can then be used to consume the dataset in parallel. The mechanism for retrieving a single partition is identical to that of making a call to the 'get-entities' operation except that the 'nextdata' query parameter is either a value from the result of calling 'get-emtities-partitions'  or the x-wod-next-data header in a response from 'get-entitites'.




### Operation Get Transactions


### Operation Create Transaction






## Data Synchronisation

The data synchronisation part of the protocol allows a client to keep a local copy of a dataset in-sync with a remote copy. It defines the service endpoint that a WoD server MUST offer and the semantics a compliant client MUST implement.

The goal of the synchronisation protocol is to provide clean and simple semantics that can be implemented easily and quickly to promote the sharing of datasets. These mechanisms enable complete datasets to be shared easily, automatically and incrementally.

Allowing applications to collect and download datasets incrementally allows them to run with no dependencies on remote services. This is turn allows them to provide a guarenteed quality of service to their users, and to offer query capabilities that are not possible in a federated, distributed or client server model.

### Operation: Get Changes

The 'get-changes' operation exposes a list of the entities that have changed in the specified dataset. Complete entity representation MUST be delivered back to the client.

This operation exposes the changes that occur to a dataset. Even though an entity may have changed many times the server is not required to return all these unique states. The server is only required to return the latest representation of the entity.

<pre>

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
        description: "If provided it is used by the server to compute which are the next entities to return. If omitted indicates that the client wishes to receive all changes."
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
            x-wod-next-data:
              description: A token that should be the value of the 'nextdata' query parameter the next time the client requests data.
              type: string
              required: true
          schema:
            type: "array"
            items:
              $ref: "#/definitions/Entity"
        400:
          description: "Invalid nextdata token"
        404:
          description: "Store or dataset not found"

</pre>

The following example describes an interaction between a client and a server. The client requests changes, updates a local store or system with the new entity representations, stores the x-wod-next-data token, and then, later on, makes a get request that uses the stored x-wod-next-data token. 

#### X-WOD-FULL-SYNC HEADER

The response can include a `X-WOD-FULL-SYNC` header. The header can have one of the following values:

  - false (default)

    Meaning that the entities in the list can replace the subject representations the client already has stored.

  - true

    Meaning that the client should delete all local data and replace it with what it receives. This value is true when a client first requests changes, and can also occur when the dataset on the server is deleted and re-populated. It MUST only appear in the first response, even if the data to re-populate is made available over many requests.
    
##### X-WOD-NEXT-DATA

The `X-WOD-NEXT-DATA` header MUST contain a token that can be passed to a subsequent call to 'get-changes' as the value of the 'nextdata' query parameter. 

### Deleted Subject Representations

The server can include deleted entities in the response. They have the following representation:

<pre>
  [
    ...
    {
      "@id"      : "...",
      "@deleted" : true     
    }
    ...
  ]
</pre>

The "@id" property and the "@deleted" property MUST be included. The server MAY provide the rest of the deleted entity data.

### Client Semantics

A client is assumed to be storing, and keeping in sync, a local copy of a remote dataset. The actual form of the local storage is left to the client to decide. It could be updating another WoD service endpoint, or writing into a database.

If the client has not yet synchronised the dataset then it must first use the 'get-changes' operation to obtain the initial data. 

The response to this request will be an array of entities and a header, 'X-WOD-NEXT-DATA', that contains a token that MUST be used as the value of the 'nextdata' query paramter in a subsequent request. The client stores the entities provided in the array into a local dataset. 

For each entity the client deletes the local representation and replaces it with the new one provided. If the entity provided is marked as deleted then the client must delete the local copy. Clients should not update local entities unless there has been an actual change.

The client MUST store the value of `X-WOD-NEXT-DATA` header value only after it has committed the changes indicated by the entities in the response. 

A client can then keep making calls and then storing the tpken in the 'X-WOD-NEXT-DATA' header until no more data is available. 

At subsequent further intervals a client should use the stored `X-WOD-NEXT-DATA` token and a requst to 'get-changes' to determine if there are any changes to the dataset. 

Clients are free to use their own schedule for calling 'get-changes'.

When a client receives the `X-WOD-FULL-SYNC` header and it has a value of `true` then the client MUST delete all content from the local dataset and replace it with what comes from the server.

### Operation Get Change Partitions

To allow the changes to a dataset to be processed in parallel a dataset MAY offer the `get-changes-partitions` endpoint. This endpoint returns a list of links that can then be used to consume the changes to a dataset in parallel. 

A client calling this operation receives N tokens. Each token can be used as an initial 'nextdata' parameter to 'get-changes'.  




## WebSocket Service Binding

As well as the HTTP semantics the following websocket variant for listening for changes is also described. This variant is designed to better support real time push of entity changes. 

The WebSocket variant of the protocol sees a client connect to a websocket endpoint provided by the server. One endpoint is provided for each dataset. When the client connects is provides the 'nextdata' parameter and then receives entities that have changed and occasional progress objects. A progress object MUST contain a property called X-WOD-NEXT-DATA whose value can be stored and used as the request parameter the next time the socket connection needs to be reopened. 

<pre>

/stores/{store-id}/dataset/{ds-id}/changes-web-socket

</pre>

The client MUST not write to the socket. The client reads from the server and receives either an entity representation (like those provided in the 'get-changes' response) or a JSON object that contains a 

## Query Operations

The WebOfData Query operations are designed to facilitate the retrieval of the representation of a given subject, and the retrieval of those subjects that reference (are connected to), a given subject.

The protocol is intended to work both in controlled (secure, closed networks) and open environments (on the web). It seperates the use of URIs as identifiers from the use of URLs as references to resolvable resource representations.

### Protocol

<pre>
GET /query?subject=&lt;uri&gt; =&gt; returns a representation of the subject.
</pre>

<pre>
GET /query?connected-to=&lt;uri&gt; =&gt; returns a list of subject representations
</pre>

<pre>
GET /query?connected-to=&lt;uri&gt;?by=&lt;uri&gt; =&gt; returns a list of subject representations that are connected with the given subject by incoming references of the specified property name.
</pre>

TODO: Add some examples here. Add support for paging. Add swagger definitions.

### Linked Data URL Resolution

Follow-your-nose semantics works by a web endpoint capturing requests for subject representations and re-writing and re-routing them to the WoD server query endpoint. The following example shows how this should work:

<pre>
http://data.webofdata.io/people/gra gets re-written as 

http://api.webofdata.io/query?connected=http://data.webofdata.io/people/gra
</pre>

An example NGINX config and docker image are provided at https://github.com/webofdata/linked-data.

## GRPC Service Binding

As well as the HTTP and websocket bindings of the protocol this section defines the GRPC version of the service. 

It should be noted that the entity data model is not represented as a GRPC model. The GRPC binding is defined in terms of bytes that contain the JSON entity objects. 

Clients in specific language bindings are responsible for using the bytes data as JSON.

## Conformance

It is not possible to define conformance for a data model, conformance can only be defined in terms of observable actions and responses. 

The automated conformance test suite can be run against any WoD node that claims to implement the protocol and serialisation parts of this specification. 

The conformance client can be found online at http://github.com/webofdata/conformance. It includes source code, instructions for running the conformance tests.

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
