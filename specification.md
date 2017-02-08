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

WebOfData is a web scale data sharing protocol and data representation format. It is designed to be simple, robust and empowering. WebOfData is split into three main sections: 

<ul>
    <li>The Semantic Data Representation
    <li>The Data Access Protocol
    <li>The Data Sharing Protocol
    <li>The Data Management Protocol (Experimental)
  </ul>
</p>  

JSON is the defacto standard for representing data on the web. However, for building a robust web of data and for semantically agreeing on what is being said, it is underpowered. 'Semantics' at the most basic level is when two or more parties can agree upon the meaning of something. In the web of data the most concrete way to do this is to publish subjects with globally unique identifiers. A subject can be anything, about which, any thing can be said. Any two parties can agree on what is being said by using the same identifiers.

While efforts such as JSON-ld have tried to merge the worlds of JSON and Semantic Web technologies they have been required to bring along too much RDF data model baggage. The RDF data model is powerful but at odds with the kinds of structures used by developers and applications.

The data representation in WebOfData is called Semantic JSON. Semantic JSON aims to provide a simple approach to unifying JSON with the use of URIs for the identity of things and the identity of property types. Using URIs for identifying subjects provides a globally unique and authorative scheme to name what we are talking about.

As well as a syntax for representating subjects WebOfData also defines a multi-purpose protocol. The protocol can be used for navigating a global web of connected data and also for allowing clients to consume complete data sets in a scalable way.

The protocol has been informed by two trends. The first is that SPARQL endpoints exposing data has been tried and found wanting in terms of scale and reliability. The protocol considers it's query capability as closer to the Linked Data Fragments concepts, but imposes further restrictions to create a more navigational rather than query experience. To compensate for reduced query capabilities WebOfData encourages the replication of datasets from servers to clients using the dataset sharing protocol. The data sharing protocol is a refinement of the SDShare protocol.

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
<li>To unambiguously know which property is the 'identity' property for this representation.
<li>For the identity of the thing we are talking about to be an IRI.
<li>For references to other concepts to be declared simply and unambiguously
<li>For contained objects to also express their identity
</ul>

The Semantic JSON version looks like this:

<pre>

    {
      ## Use of __id to denote the identity property
      "__id"      : "http://data.example.org/people/gra",

      ## Use of full IRIs as property names
      "http://data.example.org/schema/person/age" : 23,

      ## Use of '@' to indicate references to other subjects 
      "@http://data.example.org/schema/person/skills" : 
        [ "http://data.example.org/skills/csharp", 
          "http://data.example.org/skills/data" ],

      "education" : [ 
          { 
            ## Contained objects can also have global identifier property
            "__id" : http://data.example.org/people/gra"
            "name" : "southampton university"
          } 
        ]
    }

</pre>

And the concise Semantic JSON looks like this:

<pre>

    {
      ## Defintion of a context to simplify the adoption of URIs and 
      ## improve human readability of the JSON
      ## Also allows for easy reuse of existing JSON.
      ## Context definition doesnt have to be colocated with every document.
      ## They can be part of the request header or as a seperate data structure
      ## when dealing with a list of entities.
      "__context" : {
        "_:"   : "http://data.example.org/people/",
        "__:"  : "http://data.example.org/schema/person/",
        "skills" : "http://data.example.org/skills/"
        "universities" : "http://data.example.org/universities/"
      },

      "__id"      : "gra",

      "age" : 23,

      "@skills" : 
        [ "skills:csharp", 
          "skills:data" ],

      "education" : [ 
          { 
            "__id" : "universities:soton"
            "name" : "southampton university"
          } 
        ]
    }

</pre>

### Conceptual Model

This specification does not dictate how and where semantic JSON is used. However, the primary use case is that a client will make a request to a server and the response will be a semantic JSON document.

A semantic JSON document can be one of the following:

<ul>
  <li>A single subject representation.</li>
  <li>An array containing zero or one context definitions followed by N subject representations</li>
</ul> 

### Representation Rules

Semantic JSON consists of a number of rules that define how JSON data must be written in order to be considered valid Semantic JSON.

#### The __id Property

The '__id' property. The value of this property defines the identifier for the subject. This property MUST be present on the JSON object. The value of this property can be an IRI, or a curie. If it is neither of these two then it is interpreted in relation to the default context (see context resolution).

Examples of usage are as follows:

<pre>
  {
    "__id" : "http://data.example.org/people/gra" 
    ...
  } 
</pre>

or

<pre>

  {
    "__id" : "people:gra" 
    ...
  } 
</pre>

or 

<pre>
  {
    "__id" : "gra" 
    ...
  } 
</pre>

The __id property can also be used on any descendant object. When used on a descendant object it gives identity to that object. It does not affect or override the identity of the root object.

This provides a way to embed or contain other subjects by value and still convey that they have identity in their own right.

<pre>
  {
    "__id" : "http://data.br.no/people/gra",
    "education" : [
      {
        "__id" : "http://data.schools.org/stbarts",
        "name" : "st bartholomews"
      },
      {
        "__id" : "http://data.universities.org/southampton",
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
  "__id" : "http://data.example.org/people/gra",
  "@lives-in" : "http://data.cities.org/oslo"
}
</pre>

and: 

<pre>
{
  "__id" : "http://data.example.org/people/gra",
  "@lives-in" : [ "http://data.cities.org/oslo", "http://data.cities.org/oxford" ]
}
</pre>

The use of '@' is permitted anywhere in the JSON object hierarchy. For example: 

<pre>
{
  "__id" : "http://data.example.org/people/gra",
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
  "__id" : "http://data.example.org/people/gra",
  "@http://data.example.org/vocab/lives-in" : "http://data.cities.org/oslo"
  "foaf:name" : "Graham Moore"
}
</pre>


#### Contexts

All property names, '__id' property values and values of '@' properties are full URIs. However, to simplify documents and avoid repeating long URIs these values can be specified either as Curies or as simple values.

When a Curie or simple value is used then they must be resolved against some context. A document collection or a single document can describe the context in which curies and simple values are expanded.

A context is declared using the special '__context' property. The value of the '__context' property is a JSON object where each key is a Curie prefix and the value of each key is the expansion value. e.g. :

<pre>
  {
    "__context" : {
      "foaf" : "http://xmlns.com/foaf/0.1/"
    }
  }
</pre> 

As well as Curies a context can also contain default namespaces that are used as the postfix to simple values. There are two defaults that can be specified. A default for property names and a prefix for values.

There are two prefixes as often the schema vocabulary uses a different namespace to instance values.

The default instance namespace is defined used the '_:' property, and the default property namespace is defined using the '__:' property.

In an array, a context is declared with the first entity in the array being an object containing the property '___context' or '__context-href'. The href version allows the curie expansion definitions to be stored on the server.

The following example shows a context definition defined as the first entity in the array.

<pre>
  [
    {
      "__context" : {
        "foaf" : "http://xmlns.com/foaf/0.1/"
      }
    },

    {
      "__id" : "...",
      "foaf:name" : "bob"
    },

    {
      "__id" : "..."
    }
  ]
</pre>

The context can also be defined as part of the subject representation, e.g:

<pre>
  {
    "__id" : "http://data.example.org/people/gra",
    "__context" : {
      "foaf" : "http://xmlns.com/foaf/0.1/" 
    },
    "@http://data.example.org/vocab/lives-in" : "http://data.cities.org/oslo",
    "foaf:name" : "Graham Moore"
  }

  Resolves to:

  {
    "__id" : "http://data.example.org/people/gra",
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
    "__id" : "http://data.example.org/people/gra",
    "__context-href" : "http://example-server.org/context/people",
    "@http://data.example.org/vocab/lives-in" : "http://data.cities.org/oslo",
    "foaf:name" : "Graham Moore"
  }
</pre>

The default namespace provides a way to prefix any property names that are not curies and not full IRIs.

<pre>
  {
    "__id" : "http://data.example.org/people/gra",
    "__context" : {
      "__:" : "http://data.example.org/schema/"
      "foaf" : "http://data.foaf.org/schema/" 
    },
    "@http://data.example.org/vocab/lives-in" : "http://data.cities.org/oslo",
    "name" : "Graham Moore"
  }

  Which resolves to:

  {
    "__id" : "http://data.example.org/people/gra",
    "@http://data.example.org/vocab/lives-in" : "http://data.cities.org/oslo",
    "http://data.example.org/schema/name" : "Graham Moore"
  }
</pre>

As well as a schema default we also need a instance default. So given:

<pre>
  {
    "__id" : "gra",
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
    "__id" : "http://data.example.org/people/gra",
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
        "__id" : "gra",
        "__context" : {
            "people" : "http://data.example.org/person/"  
        }
        "people:name" : "Graham Moore"   
        }
    ]

Resolves to:

    {
        "__id" : "gra",
        "http://data.example.org/person/name" : "Graham Moore"   
    }
</pre>
</div>

#### Normal JSON Properties

Semantic JSON allows all other JSON without restriction. It is also possible to use data type encoding standards such as transit without issue.

## DataSet Sharing Protocol

The DataSet Sharing Protocol (DSP) is a simple protocol to allow a client to keep a local copy of a dataset in-sync with a remote copy. It defines the set of endpoints that a DSP server MUST offer and the semantics that a compliant client MUST implement.

The goal of DSP is to provide clean and simple semantics that can be implemented easily and quickly to promote the sharing of datasets. The web of data needs mechanisms that enable complete datasets to be shared easily, automatically and incrementally.

Allowing applications to collect and download datasets incrementally allows them to run with no dependencies on remote services. This is turn allows them to provide a guarenteed quality of service to their users.

### Definitions

<dl>
  <dt>DataSet
  <dd>A collection of Subjects.</dd>

  <dt>DataSet Representation
  <dd>A Semantic JSON representation of all or part of a DataSet.</dd>
</dl>

### Protocol Specification

A compliant DSP server offers a number of endpoints that expose the datasets that the server is hosting. For each of these datasets there are additional endpoints:

<pre>
GET /datasets => a list of datasets.

A JSON document with the following structure:

[
  { "dataset" : "dataset1", "href" : "datasets/dataset1" },
  { "dataset" : "dataset2", "href" : "datasets/dataset2" }
]

GET /datasets/{dataset-1} => returns metadata about the dataset

GET /datasets/dataset1 => returns metadata about the dataset

{
  "_id"            : "dataset1",
  "href"           : "/datasets/dataset1",
  "subjects"       : "/datasets/datasets1/subjects",
  "subjectcount"   : 3000,
  "lastmodified"   : "2016-12-01T00:00:00Z",
  "formats"        : ["json", "csv"]
}

</pre>

<pre>
GET /datasets/dataset1/subjects&amp;format=csv, json
</pre>

When a client uses this endpoint they receive a json array that contains the subject representations in the named dataset.

As part of the JSON response the first JSON object gives metadata about what data is going to be returned. The only property that has significant semantic meaning is the type value. 'Incremental' implies that the subject representations in the list can replace the subject representation the client already has stored. 'all' indicates that all the data the client has should be deleted and replaced with the subject representations provided. e.g.:

<pre>
  [
    {
      "id" : "request-id",
      "href" : "request-url",
      "type" : "incremental | all"
    },
</pre>

The server can include deleted subject representations in the response. They look like:

<pre>
  [
    {
      "__id" : "",
      "__deleted" : true
    }
  ]
</pre>

The last JSON object is always a reference to retrieve the next set of changes. It is expected that a client continues to call this until no more subjects are returned.

<pre>
[
  ...

  {
    "___next" : ""
  }
]
</pre>

</div>

### Client Semantics

A client is assumed to be storing, and keeping in sync, a local copy of a remote dataset. The remote dataset is exposed using the data sharing protocol.

If the client has an empty local dataset and wants to fetch all the data from the remote dataset then it should call to the subjects endpoint and follow all '__next' references until the response contains 0 subject representations. The client should store the last '__next' link it received.

The client should add the subject representations it receives into its local dataset. Each subject has a unique identifier in the context of the dataset.

At subsequent intervals a client should use the stored '__next' link to determine if there are any changes to the dataset. If there are changes then the client should replace the current local copy of the subject representation with the one provided by the server. It should again resolve the '__next' link until there are no more subject representations. Clients are free to use their own schedule for following the '__next' link.

Subject deleted markers mean that the client should remove the subject representation with that id from the dataset.

## Web Of Data Query Protocol

The Web of Data Query Protocol (WOD-QP). WOD-QP is designed to facilitate the retrieval of the representation of a given subject, and the retrieval of those subjects that reference, or are connected to, a given subject.

The protocol is intended to work both in controlled (secure,closed networks) and open environments (on the web). It seperates the use of URIs as identifiers from the use of URLs as reference to a resolvable resource representation.

### Protocol

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

TODO: Add some examples here. Add support for paging

### Subject Representation

All data returned follows the Semantic JSON format. The mime type application/json+semantic is used to indicate this.

### Linked Data URL Resolution Semantics

Follow-your-nose semantics works by a web endpoint capturing requests for subject representations and re-writing and re-routing them to the WOD-QP server. e.g:

<pre>
http://data.example.org/people/gra gets re-written as 

http://swdp-endpoint.example.org/query=http://data.example.org/people/gra
</pre>

## Web Data Of Data - Management Protocol

This protocol defines how clients can update a WebOfData node with new datasets and the contents of those datasets. Access to these endpoints should be tightly controlled.

### Protocol Definition

TODO: This is still a bit of a todo but here is the outline:  

<pre>

DataSet Management:
-------------------

GET /datasets

POST /datasets { dataset body }

PUT /datasets/{dataset-id} 

DELETE /datasets/{dataset-id}

GET /datasets/{dataset-id}

Subjects Management
-------------------

GET /datasest/{dataset-id}/subjects &lt; basically the same as WOD-SP.

PUT /datasets/{dataset-id}/subjects?id=&lt;uri&gt; { subject-representation }

DELETE /datasets/{dataset-id}/subjects?id=&lt;uri&gt;

POST /datasets/{dataset-id}/subjects [ { subject-representation }, { subject-representation } ]

DELETE /datasets/{dataset-id}/subjects &lt; Clears all subjects but leaves the dataset.



