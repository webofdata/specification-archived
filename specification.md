# WebOfData Specification

Author  : Graham Moore  
Contact : gramoore (at) outlook (dot) com  
Version : 0.1  
This Document : [Specification 0.1](http://github.com/webofdata/specification/specification.md)  
Issues : <http://github.com/webofdata/specification/issues>

## Abstract

WebOfData defines a web protocol and data representation format for publishing, sharing and connecting data on the web. The protocol is designed to facilitate the sharing and publishing of datasets, the retrieval of the representation of a given subject, and the retrieval of connected subject representations. 

The representation format unifies JSON with URIs to provide a means for identifying subjects and property types when interchanging data representation of subjects. Using URIs for identifying subjects provides a globally unique and authorative mechanism to unambiguously name the subjects of discourse and the vocabularies used to describe them. 

The data access and management protocols provide a means to manage datasets; to access and query them in a globally scalable way. The protocols also provide mechanisms to synchronise datasets between WebOfData nodes.

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

## Introduction

WebOfData is a web scale data sharing protocol and data representation format. It is designed to be simple, robust and empowering. WebOfData is split into four main sections: 

<ul>
    <li>Semantic Json Data Representation
    <li>Data Access Protocol
    <li>Data Sharing Protocol
    <li>Data Management Protocol
</ul>

JSON is the defacto standard for representing data on the web. However, for building a robust web of data and for semantically agreeing on what is being said, it is underpowered. 'Semantics' at the most basic level is when two or more parties can agree upon the meaning of something. In the web of data the most concrete way to do this is to publish subjects with globally unique identifiers. A subject can be anything, about which, any thing can be said. Any two parties can agree on what is being said by using the same identifiers.


While efforts such as JSON-ld have tried to merge the worlds of JSON and Semantic Web technologies they have been required to bring along too much RDF data model baggage. The RDF data model is powerful but at odds with the kinds of structures used by developers and applications.


The data representation in the WebOfData is called Semantic JSON. Semantic JSON aims to provide a simple approach to unifying JSON with the use of URIs for the identity of things and the identity of property types. Using URIs for identifying subjects provides a globally unique and authorative scheme to name what we are talking about.


As well as a syntax for representating subjects WebOfData also defines a multi-purpose query, synchronisation and update protocol. The protocol can be used for navigating a global web of connected data and also for allowing clients to consume complete data sets in a scalable way.


The protocol has been informed by two trends. The first is that SPARQL endpoints exposing data has been tried and found wanting in terms of web scale and reliability. The WebOfData protocol considers it's query capability as closer to the Linked Data Fragments concepts, but imposes further restrictions to create a more navigational rather than query experience. To compensate for reduced query capabilities WebOfData encourages the replication of datasets from servers to clients using the dataset sharing protocol. The data sharing protocol is a refinement of the SDShare protocol and OData change sets.


The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## Semantic JSON

WebOfData introduces the following core concepts: 

<dl>
  <dt>Subject
  <dd>A concept, an idea, a thing; any thing, about which anything can be said.</li>

  <dt>Subject Identifier
  <dd>A unique identifier for a subject.</li>

  <dt>Subject Representation
  <dd>A Semantic JSON document for a subject</li>
</dl>

### Example (Informative)

Before describing the Semantic JSON representation here are few examples to demonstrate the concept. The idea behind Semantic JSON is that a single JSON document is a (partial) representation of some subject. That subject has identity in the form of an IRI. The property names of the JSON object should also be expressed as IRIs. This allows for both things and property names (vocabularies) to be discussed, reused and agreed upon.

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
      ## Use of _si to denote the subject identity property
      "_si"      : "http://data.example.org/people/gra",

      ## Use of full IRIs as property names
      "http://data.example.org/schema/person/age" : 23,

      ## Use of '@' to indicate references to other subjects 
      "@http://data.example.org/schema/person/skills" : 
        [ "http://data.example.org/skills/csharp", 
          "http://data.example.org/skills/data" ],

      "education" : [ 
          { 
            ## Contained objects can also have an identifier property
            "_si" : http://data.example.org/unis/soton",
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
      ## when dealing with a array of Semantic JSON objects.
      "__context" : {
        # The default instance context
        "_:"   : "http://data.example.org/people/",
        # The defualt schema context
        "__:"  : "http://data.example.org/schema/person/",
        "skills" : "http://data.example.org/skills/"
        "universities" : "http://data.example.org/universities/"
      },

      "_si"      : "gra",

      "age" : 23,

      "@skills" : 
        [ "skills:csharp", 
          "skills:data" ],

      "education" : [ 
          { 
            "_si" : "universities:soton"
            "name" : "southampton university"
          } 
        ]
    }

</pre>

### Representation Rules

Semantic JSON consists of a number of rules that define how JSON data must be written in order to be considered valid Semantic JSON.

#### The _si Property

The '_si' property. The value of this property defines the identifier for the subject. This property MUST be present on the JSON object. The value of this property can be an IRI, or a curie. If it is neither of these two then it is interpreted in relation to the default context (see context resolution).

Examples of usage are as follows:

<pre>
  {
    "_si" : "http://data.example.org/people/gra" 
    ...
  } 
</pre>

or

<pre>

  {
    "_si" : "people:gra" 
    ...
  } 
</pre>

or 

<pre>
  {
    "_si" : "gra" 
    ...
  } 
</pre>

The _si property can also be used on any descendant object. When used on a descendant object it gives identity to that object. It does not affect or override the identity of the root object.

This provides a way to embed or contain other subjects by value and still convey that they have identity in their own right.

<pre>
  {
    "_si" : "http://data.br.no/people/gra",
    "education" : [
      {
        "_si" : "http://data.schools.org/stbarts",
        "name" : "st bartholomews"
      },
      {
        "_si" : "http://data.universities.org/southampton",
        "name" : "Southampton University"   
      }
    ]
  }
</pre>

#### Subject References</h3>

Subjects can reference other subjects using their subject identifier. To represent a subject reference the '@' character MUST come at the start of the property name. The value of any subject reference properties MUST either be a string containing an IRI or curie, or an array of strings where each string is an IRI or curie.

<p>The following examples show how to use '@': </p>

<pre>
{
  "_si" : "http://data.example.org/people/gra",
  "@lives-in" : "http://data.cities.org/oslo"
}
</pre>

and: 

<pre>
{
  "_si" : "http://data.example.org/people/gra",
  "@lives-in" : [ "http://data.cities.org/oslo", "http://data.cities.org/oxford" ]
}
</pre>

The use of '@' is permitted anywhere in the JSON object hierarchy. For example: 

<pre>
{
  "_si" : "http://data.example.org/people/gra",
  "skills" : [
    { 
      "name"      : "csharp", 
      "@category" : "http://data.categories.org/programming" 
    }
  ] 
}
</pre>

#### Properties as Subjects

As well as being able to talk about 'things' using subject identifiers, we also need to be able to agree on the properties being used to communicate. In Semantic JSON all property names are IRIs. They can be expressed as full IRIs, Curies or simple values. 

An example with properties as IRIs. Both the 'foaf:name' and 'http://data.example.org/vocab/lives-in' are properties that will resolve to full IRIs.

<pre>
{
  "_si" : "http://data.example.org/people/gra",
  "@http://data.example.org/vocab/lives-in" : "http://data.cities.org/oslo"
  "foaf:name" : "Graham Moore"
}
</pre>


#### Contexts

All property names, '_si' property values, and values of '@' properties are full URIs. However, to simplify documents and avoid repeating long IRIs strings these values can be specified either as Curies or as simple values.

When a Curie or simple value is used then they must be resolved against some context. A document collection or a single document can describe the context in which curies and simple values are expanded.

A context is declared using the special '__context' property. The value of the '__context' property is a JSON object where each key is a Curie prefix and the value of each key is the expansion value. e.g. :

<pre>
  {
    "__context" : {
      "foaf" : "http://xmlns.com/foaf/0.1/"
    }
  }
</pre> 

As well as Curies a context can also contain default namespaces that are used as the prefix to simple values. There are two defaults that can be specified. A default for property names and a default prefix for subject identifier values and subject reference values.

There are two prefix defaults as the schema vocabulary often uses a different namespace to instance values.

The default instance namespace is defined used the '_:' property, and the default property namespace is defined using the '__:' property.

In an array, a context is declared with the first entity in the array being an object containing the property '__context' or '__context-href'. The href version allows the curie expansion definitions to be stored on the server.

The following example shows a context definition defined as the first entity in the array.

  <pre>
  [
    {
      "__context" : {
        "foaf" : "http://xmlns.com/foaf/0.1/"
      }
    },

    {
      "_si" : "...",
      "foaf:name" : "bob"
    },

    {
      "_si" : "..."
    }
  ]
</pre>

The context can also be defined as part of the subject representation, e.g:

<pre>
  {
    "_si" : "http://data.example.org/people/gra",
    "__context" : {
      "foaf" : "http://xmlns.com/foaf/0.1/" 
    },
    "@http://data.example.org/vocab/lives-in" : "http://data.cities.org/oslo",
    "foaf:name" : "Graham Moore"
  }

  Resolves to:

  {
    "_si" : "http://data.example.org/people/gra",
    "@http://data.example.org/vocab/lives-in" : "http://data.cities.org/oslo",
    "http://xmlns.com/foaf/0.1/name" : "Graham Moore"
  }
</pre>

As an alternative to defining the context in the array or as part of a single object the context can be a remote resource. A remote context is indicated by the use of the '__context-href' property. A client processing the Semantic JSON is expected to fetch the resource. The resource should be a single JSON document.

<pre>
  {
    "__context-href" :  "http://example-server.org/context/people"
  }
</pre>

The following example shows the use of the '__context-href' property as part of an subject representation:

<pre>
  {
    "_si" : "http://data.example.org/people/gra",
    "__context-href" : "http://example-server.org/context/people",
    "@http://data.example.org/vocab/lives-in" : "http://data.cities.org/oslo",
    "foaf:name" : "Graham Moore"
  }
</pre>

The default namespace provides a way to prefix any property names that are not curies and not full IRIs.

<pre>
  {
    "_si" : "http://data.example.org/people/gra",
    "__context" : {
      "__:" : "http://data.example.org/schema/"
      "foaf" : "http://data.foaf.org/schema/" 
    },
    "@http://data.example.org/vocab/lives-in" : "http://data.cities.org/oslo",
    "name" : "Graham Moore"
  }

  Which resolves to:

  {
    "_si" : "http://data.example.org/people/gra",
    "@http://data.example.org/vocab/lives-in" : "http://data.cities.org/oslo",
    "http://data.example.org/schema/name" : "Graham Moore"
  }
</pre>

As well as a schema default we also need a instance default. So given:

<pre>
  {
    "_si" : "gra",
    "__context" : {
      "__schema-default" : "http://data.example.org/schema/"
      "__instance-default" : "http://data.example.org/people/"
      "foaf" : "http://data.foaf.org/schema/" 
    },
    "@http://data.example.org/vocab/lives-in" : "http://data.cities.org/oslo",
    "name" : "Graham Moore"
  }

  Resolves to:

  {
    "_si" : "http://data.example.org/people/gra",
    "@http://data.example.org/vocab/lives-in" : "http://data.cities.org/oslo",
    "http://data.example.org/schema/name" : "Graham Moore"
  }
</pre>

If a context is defined in an array and in an object the contexts are merged with the values in the object context taking precedence.

<pre>
  Given:
    [
        {
        "__context" : {
            "people" : "http://data.example.org/people/"
        }
        },

        {
        "_si" : "gra",
        "__context" : {
            "people" : "http://data.example.org/person/"  
        }
        "people:name" : "Graham Moore"   
        }
    ]

Resolves to:

    {
        "_si" : "gra",
        "http://data.example.org/person/name" : "Graham Moore"   
    }
</pre>

#### Merging Subject Representations

A client can (potentially) retreive a partial subject representation from many WOD service endpoints. The rules for merging two representations of the same subject are as follows:

To merge two subject representations A and B the following algorithm should be applied:

  - Let A and B be the two representations to be merged.
  - Create a new empty representation, C.
  - The "_si" of C is the _si of A (or B as they are the same)
  - For each property in A if there is no corresponding property in B add it to C.
  - For each property in A if there is a corresponding property in B then merge the values:
    - If either of A or B properties are a list then they are merged by flattening them into one list: e.g. [a, b, c][d, e] becomes [a,b,c,d,e], and [ {}, [], "d"], "x" becomes: [ {}, [], "d", "x" ]. 
    - For non lists values a new list is created and the values from A and B are added. e.g. {}, "x" becomes: [ {}, "x" ]
    - Duplicates are removed.
  - All properties on B that are not present on A are added to C.

Merging of two or more subject representations can result in a new materialised representation or can appear to be a merged representation. This is up to the client to decide. These rules are defined to give some consistency to the merging process. 

#### Normal JSON Properties

Semantic JSON allows all other JSON without restriction. It is also possible to use data type encoding standards such as transit, without issue.

## DataSet Sharing Protocol

The DataSet Sharing Protocol (DSP) is a simple protocol to allow a client to keep a local copy of a dataset in-sync with a remote copy. It defines the set of endpoints that a DSP server MUST offer and the semantics that a compliant client MUST implement.

The goal of DSP is to provide clean and simple semantics that can be implemented easily and quickly to promote the sharing of datasets. The WebOfData needs mechanisms that enable complete datasets to be shared easily, automatically and incrementally.

Allowing applications to collect and download datasets incrementally allows them to run with no dependencies on remote services. This is turn allows them to provide a guarenteed quality of service to their users.

### Definitions

<dl>
  <dt>DataSet
  <dd>A collection of Subjects.</dd>

  <dt>DataSet Representation
  <dd>A Semantic JSON representation of all or part of a DataSet.</dd>
</dl>

### Protocol Specification

A compliant DSP server exposes a single endpoint that MUST serve a service description document. This document contains typed links. The type of each link is described below. These typed links form a hypertext that can be processed by a compliant client. It is important that clients follow the semantics and link types and do not craft URIs by hand.

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
      "_deleted" : true
    }
    ...
  ]
</pre>

The "_si" property and the "_deleted" property MUST be included. The server MAY provide the rest of the deleted subject representation.

### Client Semantics

A client is assumed to be storing, and keeping in sync, a local copy of a remote dataset. The remote dataset is exposed using the data sharing protocol.

If the client has an empty local dataset and wants to fetch all the data from the remote dataset then it should call to the subjects endpoint and follow the URL value contained in the `X-WOD-DSP-NEXT-PAGE` header until there are no more headers of this type contained in the reponse. 

As the client processes each page of subjects it should add the subject representations it receives into its local dataset. 

The client is free to store the value of the `X-WOD-DSP-NEXT-DATA` header value link it at any point, but it MUST store it after processing the last page.

At subsequent intervals a client should use the stored `X-WOD-DSP-NEXT-DATA` link to determine if there are any changes to the dataset. If there are changes then the client should replace the current local copy of the subject representation with the one provided by the server. It should again resolve the `X-WOD-DSP-NEXT-PAGE` link until there are no more subject representations. 

Clients are free to use their own schedule for following the `X-WOD-DSP-NEXT-DATA` link.

When a client recieves the `X-WOD-DSP-DATASET-RESET` header and it has a value of `true` then the client MUST delete all content from the local dataset and replace it with what comes from the server.

Subjects with the property "_deleted" with a value of "true" mean that the client should remove the subject representation with the corresponding '_si' from the local dataset.

A client is free to identify and name the local dataset. 

<!--
#### WebSocket Semantics

As well as the HTTP semantics described above the following websocket variant is also described. This variant is designed to better support real time push of subject changes. 

The WebSocket variant of the protocol sees a client connect to a websocket endpoint provided by the server. Something like:

<pre>
https://api.webofdata.io/dsp-socket
</pre>



If the connection is broken then a client can reconnect using the 

This should be considered a long lived connection

#### GRPC Semantics

As well as the HTTP semantics described above the following tcp socker variant is also described. This variant is designed to support real time push of subject changes. 

-->

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

DataSet Management:
-------------------

GET /datasets

POST /datasets { dataset body }

PUT /datasets/{dataset-id} { dataset body }

DELETE /datasets/{dataset-id}

GET /datasets/{dataset-id}

Subjects Management
-------------------

GET /datasest/{dataset-id}/subjects &lt; basically the same as WOD-SP.

PUT /datasets/{dataset-id}/subjects?id=&lt;uri&gt; { subject-representation }

DELETE /datasets/{dataset-id}/subjects?id=&lt;uri&gt;

POST /datasets/{dataset-id}/subjects [ { subject-representation }, { subject-representation } ]

DELETE /datasets/{dataset-id}/subjects &lt; Clears all subjects but leaves the dataset.

</pre>

We also need a SPARUL like update mechanism to allow updates to move than one dataset in a single transaction.

POST /updates => txn id
GET /updates/{txn-id}

<pre>



</pre>

