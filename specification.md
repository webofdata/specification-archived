
# WebOfData Specification

Author  : Graham Moore  
Contact : gramoore (at) outlook (dot) com  
Version : 0.8  
This Document : [Specification 0.8](http://github.com/webofdata/specification/specification.md)  
Issues : <http://github.com/webofdata/specification/issues>

## Overview

Like HTML and HTTP did for machine to human communication, WebOfData aims to provide the building blocks to allow any device or application to easily communicate, access, modify, navigate and share data at web scale. Enabling low friction, meaningful,standardised, machine-to-machine communication.

WebOfData defines a protocol, data model, and data representation format for publishing, sharing and connecting data on the web. It is intended to be easy to implement and work at a level of generality that is widely applicable.

The WoD data model has at it's core the notion of an entity. An entity has identity in the form of a URI. An entity also has properties whose 'names' or 'keys' are also URIs. The use of URIs to identify things and name properties provides a powerful, web scale approach to naming. The values of these properties can either be literals, based on XML schema data types, or complex types such as arrays, or another entity. Finally, there is a special reference type. This reference type has a value that is a URI. This URI references another entity to create a graph of entities that can span the web. 

The protocol is designed to facilitate the sharing, updating, creating and publishing of collections of entities, the retrieval of a given data entity, and the navigation of connected entities. It is expected that collections of entities can exists in databases, existing applications and dedicated WoD data stores. All kinds of applicatios can implement and support the protocol in meaningful ways, allowing a vast array of client applications to easily consume and use data from around the web. 

The WoD representation format uses JSON with a few well defined keys that support the use of URIs as identifiers and property names. 

Finally, WoD defines a protocol for synchronising datasets between WebOfData implementations.

## Background

Several attempts have been made to define a standard and generic data model and related APIs; most notably OData and the RDF stack from W3C. While technically sound they both have significant barriers to adoption - The RDF data model is too shredded and lacks basic constructs such as list and map; OData has a data model that doesn't embrace URIs for identity or naming. The query semantics and models in both is unappealing to users, and too much work for implementors.  

WebOfData builds on the experience of these and other standards to propose a universal data management and data sharing API, coupled with a data model that works at web scale.

A key concept that underpins WebOfData and it is the notion of a 'subject'. A subject is any thing, about which, anything may be stated or asserted, by anyone. A subject is an abstract concept, it can be a person, a document on a file system, a row in a database, a book. 

Computer systems are all about building representations of these abstractions. Every application creates a model where the data structures are proxies, or stand-ins for their 'real life' counterpart.  

Standards such as RDF and Topic Maps realised that to create a global space in which anyone can talk about any thing, there needs to be an agreed way to bind the data object in a computer system to the subject it represents. They also realised that there can be many partial representations of the same subject. These partial representations may or may not be known at any given time. 

Using URIs for identifying subjects provides a globally unique and authorative mechanism to unambiguously name the subjects of discourse and the vocabularies used to describe them.

WebOfData uses the term 'entity' to be the data object that is a representation of a subject. The binding between these two things is the use of a URI as the identifier for a given subject and entity. Note that many entities can have the same identifier. If they do, then they can considered as partial representations of the same subject.  

The WoD APIs are refinement of things like OData, Linked Data Fragments and SDShare.

## Introduction

WebOfData defines a data model, a JSON serialistion for the data model, a data access protocol (Query), a data sharing protocol (Synchronisation) and a data management protocol (CRUD). Compliant implementations can choose which of the APIs they support, but must adhere to the rules and semantics of the data model and serialisation definition. 

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## Data Model Definition

The WebOfData data model can be thought of an Entity-Graph model. Entities are complex objects of keys with values. Where keys are URIs and values can be literals, lists or another entity. In addition there is a reference value type that points to (references) another entity. This allows entities to be connected together in a graph at global scale. 

The value of a reference type is a URI and not some local internal identifier. The power of this is that it can be resolved globally. It can be used to lookup the related entity from one or more API endpoints. WoD does not define where a client can resolve an reference value, but does define the capabilities that a WoD data access endpoint should support.

Given that each entity is only a partial representation of a subject the data model also defines how to merge two entities. (The algorithm can be applied recursively to merge multiple representations.)

As well as entities there are two levels of container types. The store and the dataset. A store consists of zero or more datasets, and each dataset is comprised of 0 or more entities. 

WebOfData data core concepts: 

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

A dataset is a collection of entities. It provides a container into which new entities can be managed (created, deleted, updated, located). 

Datasets can contain as most 1 entity with any given subject identifier. This means that the subject identifier is the unique key for an entity in a dataset. But, an entity with the same subject identifier can exist in other datasets.

Splitting the store into datasets allows for more than one representation of the same subject (an entity with the same subject identifier as another entity) to be in a single store. 

Datasets should be able to provide a list of which entities have changed since a given point. The way this point marker is defined is opaque and left to the implementation to decide. Typical examples would be a datetime value, or a sequence number.

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

A client can (potentially) retreive a partial subject representation for the same subject from many WOD service endpoints. An application or server may decide that even though the subject identifiers are different that the entities represent the same subject, and entities are allowed to express a belief that they are the same as one or more other subjects using the "wod:sameas" property. In all of these cases two or more entities are merged to create a more unified representation.  

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

This behaviour is optional in that a server MAY offer this capability, and even in the case when it does then the client must specifically request it.

Given a subject identifier, find the entity that corresponds to that URI. Locate the 'wod:sameas' property and for each subject-ref value in the array look up and merge that entity according to the rules above.

Recursively apply this rule to each entity references by a wod:sameas value, but only process each entity once.

## Data Model JSON Serialisation

JSON is used as the serialisation format for the Data Model. It defines how an instance of the data model can be serialised as JSON. The serialisation does not use any extensions to the JSON language, a valid WoD Data Model JSON document is a valid JSON document, but it is not true that any JSON document is a valid WoD JSON document.

### Core Rules

The <b>id</b> property of the entity is serialised with the key <b>@id</b>.

The <b>subject-ref</b> URI values are serialised with <b><</b> as the first character and <b>></b> as the last character. The enclosing value is the string representation of a curie or URI.

While full URIs can be used in WoD Json documents as keys, values and id property values, it is optimal for machines and easier for humans if full URIs are replaced with CURIEs. Contexts definitions are used to define default namespaces, CURIEs. The datatypes of property values can also be defined in a context.

The following rules define how JSON data must be written in order to be considered valid WoD JSON.

#### The @id Property Rule

The <b>@d</b> property of the entity is serialised as key of <b>@id</b>. The value of this property defines the identifier for the entity. This property MUST be present on the root JSON object. The value of this property can be an full URI or a CURIE. If it is neither of these two then it is interpreted in relation to the default context (see context resolution).

The following examples show the different usages and are all semantically equivalent:

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

As well as being able to talk about 'subjects' using subject identifiers, we also need to be able to agree on the properties being used. In WoD all property names are URIs. They can be expressed as full IRIs, Curies or simple values. If they are expressed as simple values they are resolved against the defualt namespace defined in a context.

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

"<key>" : "<namespace-prefix>"

The following form is allowed:

<key> : {
  "@id"   : "<namespace-prefix>",
  "@type" : "xsd:string"  
}


## DataSet Sharing Protocol

The DataSet Sharing Protocol (DSP) is a simple protocol to allow a client to keep a local copy of a dataset in-sync with a remote copy. It defines the set of endpoints that a DSP server MUST offer and the semantics that a compliant client MUST implement.

The goal of DSP is to provide clean and simple semantics that can be implement easily and quickly to promote the sharing of datasets. These mechanisms enable complete datasets to be shared easily, automatically and incrementally.

Allowing applications to collect and download datasets incrementally allows them to run with no dependencies on remote services. This is turn allows them to provide a guarenteed quality of service to their users, and to offer query cababilities that are not possible in a federated, distributed or client server model.

### Definitions

<dl>
  <dt>DataSet
  <dd>A collection of Subjects.</dd>

  <dt>DataSet Representation
  <dd>A Semantic JSON representation of all or part of a DataSet.</dd>
</dl>

### Protocol Specification

The protocol is defined using Swagger to

#### The Service Document Endpoint

This specification does not define the root URI for the document, but would recommend that a WOD server would expose:

<pre>
  /dsp 
</pre>

Such as:

<pre>
  https://api.webofdata.io/dsp 
</pre>

The service document has the following structure:

<pre>
{
  "title" : "this web of data data sync protocol",
  "datasets_href" : "datasets"
}
</pre>

The document contains a single JSON Object with the following structure:

<dl>
  <dt>title (REQUIRED)
  <dd>A label for this WebOfData data sync endpoint.</dd>

  <dt>datasets_href (REQUIRED)
  <dd>A relative or absolute URI that accepts HTTP GET and returns a resource of type `dataset-list`</dd>
</dl>

#### Dataset List Resource Type

A resource of the type `dataset-list` has the following representation and semantics.

<pre>
[
  { 
    "subjectidentifier"  : "http://data.webofdata.io/datasets/dataset1",
    "name"               : "products dataset", 
    "href"               : "datasets/dataset1", 
  },
  { 
    "subjectidentifier"  : "http://data.webofdata.io/datasets/dataset2",
    "name"               : "people dataset", 
    "href"               : "datasets/dataset2", 
  }
]
</pre>

The document contains a list of JSON Objects where each has the following structure:

<dl>
  <dt>subjectidentifier (REQUIRED)
  <dd>The id for the dataset</dd>

  <dt>name (REQUIRED)
  <dd>A label for this dataset.</dd>

  <dt>href (REQUIRED)
  <dd>A relative or absolute URI that returns a resource of type `dataset-info`.</dd>
</dl>

#### Dataset Info Resource Type

A resource of the type `dataset-info` has the following representation and semantics.

<pre>
{
  "subjectidentifier"  : "http://data.webofdata.io/datasets/dataset1",
  "name"               : "dataset1",
  "subjects_href"      : "subjects",
  "subjectcount"       : 3000,
  "lastmodified"       : "2016-12-01T00:00:00Z"
}
</pre>

The document contains a single JSON Object with the following structure:

<dl>
  <dt>subjectidentifier (REQUIRED)
  <dd>The id for the dataset</dd>

  <dt>name (REQUIRED)
  <dd>A label for this dataset.</dd>

  <dt>subjects_href (REQUIRED)
  <dd>A relative or absolute URI that returns a resource of type `modified-subjects-list`.</dd>
</dl>

#### Modified Subjects List Resource Type

The resource type `modified-subjects-list` is an feed of subject representations that are ordered by when they were created, modified, or deleted.

A resource of the type `modified-subjects-list` has the following representation, headers and semantics.

When fetching a resource of this type the following HTTP headers are applicable: 

##### X-WOD-DSP-FEED-TYPE

The response can include a `X-WOD-DSP-DATASET-RESET` header. The header can have one of the following values:

  - false (default)

    Meaning that the set of subjects in the list can replace the subject representations the client already has stored.

  - true

    Meaning that the client should delete all local data and replace it with what it receives. 

##### X-WOD-DSP-NEXT-PAGE

The `X-WOD-DSP-NEXT-PAGE` header MUST contain a URI that will return a resource of type `modified-subjects-list`.

This header is used to provide paging when the dataset being synchronised is large. The server is free to choose the page size.

##### X-WOD-DSP-NEXT-DATA

The `X-WOD-DSP-NEXT-DATA` header MUST contain a URI that will return a resource of type `modified-subjects-list`.

This header is used to provide the client with a link that will provide the next collection of updates.

### Deleted Subject Representations

The server can include deleted subject representations in the response. They look like:

<pre>
  [
    ...
    {
      "_si"      : "...",
      "_deleted" : true     ## TODO: use a proper wod namespace rather than _
    }
    ...
  ]
</pre>

The "_si" property and the "_deleted" property MUST be included. The server MAY provide the rest of the deleted subject representation.

### Client Semantics

A client is assumed to be storing, and keeping in sync, a local copy of a remote dataset. The remote dataset is exposed using the data sharing protocol.

If the client has an empty local dataset and wants to fetch all the data from the remote dataset then it should call to the subjects endpoint and follow the URL value contained in the `X-WOD-DSP-NEXT-PAGE` header until there are no more headers of this type contained in the response. 

As the client processes each page of subjects it should add the subject representations it receives into its local dataset. 

The client is free to store the value of the `X-WOD-DSP-NEXT-DATA` header value link it at any point, but it MUST store it after processing the last page.

At subsequent intervals a client should use the stored `X-WOD-DSP-NEXT-DATA` link to determine if there are any changes to the dataset. If there are changes then the client should replace the current local copy of the subject representation with the one provided by the server. It should again resolve the `X-WOD-DSP-NEXT-PAGE` link until there are no more subject representations. 

Clients are free to use their own schedule for following the `X-WOD-DSP-NEXT-DATA` link.

When a client recieves the `X-WOD-DSP-DATASET-RESET` header and it has a value of `true` then the client MUST delete all content from the local dataset and replace it with what comes from the server.

Subjects with the property "_deleted" with a value of "true" mean that the client should remove the subject representation with the corresponding '_si' from the local dataset.

A client is free to identify and name the local dataset. 

#### WebSocket Semantics

As well as the HTTP semantics described above the following websocket variant is also described. This variant is designed to better support real time push of subject changes. 

The WebSocket variant of the protocol sees a client connect to a websocket endpoint provided by the server. One endpoint is provided for each dataset. When the client connects is provides the since parameter and then receives the changes. 

<pre>

/stores/{store-id}/dataset/{ds-id}/changes-web-socket

</pre>

#### GRPC Semantics

As well as the HTTP semantics described above the following GRPC binding is defined. 




## Web Of Data Query Protocol

The WebOfData Query Protocol (WOD-QP). WOD-QP is designed to facilitate the retrieval of the representation of a given subject, and the retrieval of those subjects that reference (are connected to), a given subject.

The protocol is intended to work both in controlled (secure,closed networks) and open environments (on the web). It seperates the use of URIs as identifiers from the use of URLs as references to resolvable resource representations.

### Protocol

TODO: introduce a service document with rel-types that provide a better quality REST interface.

A server offering the WOD-QP is a web application that exposes the following endpoints:

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

### Namespace Extensions

Semantic JSON utilises CURIES as a way to provide more consise and human readable representations. A WOD server MAY choose to use CURIEs when exposing data via the "__context" property. In the cases where CURIES are used the server can disclose the CURIE prefix expansions.

A WOD server MUST expose the following endpoints:

<pre>
GET /namespaces returns a JSON object that defines all of the CURIE prefix expansions.

A server MAY choose to return an empty document if it doesn't support CURIE expansion in the query requests.
</pre>

A client can use a CURIE when querying. The curie is first resolved against the publishing namespaces and then the internal namespaces. 

### Subject Representation

All data returned follows the Semantic JSON format. The mime type application/json+semantic is used to indicate this.

### Linked Data URL Resolution Semantics

Follow-your-nose semantics works by a web endpoint capturing requests for subject representations and re-writing and re-routing them to the WOD-QP server. e.g:

<pre>
http://data.webofdata.io/people/gra gets re-written as 

http://api.webofdata.io/query?connected=http://data.webofdata.io/people/gra
</pre>

## Web Of Data - Management Protocol

This protocol defines how clients can update a WebOfData node with new datasets and the contents of those datasets. Access to these endpoints should be tightly controlled.

### Protocol Definition

TODO: This is still a bit of a todo but here is the outline:  

<pre>

Store Management
----------------

GET /stores

POST /stores

GET /stores/{store-id}

DELETE /stores/{store-id}

PUT /stores/{store-id} 


DataSet Management:
-------------------

GET /stores/{store-id}/datasets

POST /stores/{store-id}/datasets { dataset body }

PUT /stores/{store-id}/datasets/{dataset-id} { dataset body }

DELETE /stores/{store-id}/datasets/{dataset-id}

GET /stores/{store-id}/datasets/{dataset-id}

Subjects Management
-------------------

GET /stores/{store-id}/datasest/{dataset-id}/entities &lt; basically the same as WOD-SP.

PUT /stores/{store-id}/datasets/{dataset-id}/entities?id=&lt;uri&gt; { subject-representation }

DELETE /stores/{store-id}/datasets/{dataset-id}/subjects?id=&lt;uri&gt;

POST /stores/{store-id}/datasets/{dataset-id}/subjects [ { subject-representation }, { subject-representation } ]

DELETE /stores/{store-id}/datasets/{dataset-id}/subjects &lt; Clears all subjects but leaves the dataset.

</pre>

<pre>

In order to update several entities across different data sets in a transactional way the following endpoint MUST be provided:

POST /stores/{store-id}/txns 

{
  "reference"  : "some client provided info",
  "operations" : [
    {
      "method"  : "PUT",
      "resource" : "/datasets/people/person1",
      "data"     : { }
    },

  ]
}

GET /stores/{store-id}/txns/{txn-id}

GET /stores/{store-id}/txns?status=inprogress

GET /stores/{store-id}/txns?status=completed


</pre>

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


### Example (Informative)

Before describing the JSON representation a few examples are provided to demonstrate usage. A single JSON document is a serialisation of one or more entities. That entity has identity in the form of an URI and the property keys of the JSON object should also be expressed as URIs.

The following example shows the raw JSON, the fully expanded Semantic JSON equivalent, and the concise Semantic JSON equivalent:

Here is a simple description of a person:

<pre>
    {
      "id"        : "gra",
      "age"       : 23,
      "skills"    : [ "csharp", "data"],
      "education" : [ { "southampton university"} ]
    }
</pre>

To be more `semantic` (robust in our clarity of what we are talking about), we would like the following:

<ul>
<li>To unambiguously know which property is the 'subject identity' property for this representation.
<li>For the identity of the thing we are talking about to be an IRI.
<li>For references to other concepts to be declared simply and unambiguously
<li>For contained objects to also express their identity
</ul>

The Semantic JSON version looks like this:

<pre>

    {
      ## Use of @id to denote the subject identity property
      "@id"      : "http://data.example.org/people/gra",

      ## Use of full IRIs as property names
      "http://data.example.org/schema/person/age" : 23,

      ## Use of '< >' to indicate references to other subjects 
      "http://data.example.org/schema/person/skills" : 
        [ "<http://data.example.org/skills/csharp>", 
          "<http://data.example.org/skills/data>" ],

      "education" : [ 
          { 
            ## Contained objects can also have an identifier property
            "@id" : http://data.example.org/unis/soton",
            "name" : "southampton university"
          } 
        ]
    }

</pre>

And the concise Semantic JSON looks like this:

<pre>

    {
      ## The defintion of a context simplifies the adoption of URIs and 
      ## improves human readability of the JSON
      ## Also allows for easy reuse of existing JSON.
      ## Context definition doesn`t have to be colocated with every Semantic JSON object.
      ## They can be part of the request header or as a seperate JSON object
      ## when dealing with an array of Semantic JSON objects.

      "@context" : {
        
        # The default instance namespace
        "_:"   : "http://data.example.org/people/",
        
        # The default schema namespace
        "__:"  : "http://data.example.org/schema/person/",
        "skills" : "http://data.example.org/skills/"
        "universities" : "http://data.example.org/universities/"
      },

      "@id"      : "gra",

      "age" : 23,

      "skills" : 
        [ "<skills:csharp>", 
          "<skills:data>" ],

      "education" : [ 
          { 
            "_si" : "universities:soton"
            "name" : "southampton university"
          } 
        ]
    }

</pre>
