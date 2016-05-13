# Interface Specification: RDF Representation

## Mission Statement
This document provides a specification of a low level interface definition representing RDF data independent of a serialized format in a JavaScript environment. The task force which defines this interface was formed by RDF JavaScript library developers with the wish to make existing and future libraries interoperable. This definition strives to provide the minimal necessary interface to enable interoperability of libraries such as serializers, parsers and higher level accessors and manipulators.


**Status:** This document is a _draft_.<br>
**Purpose:** This document proposes a uniform interface to represent RDF data in low-level JavaScript libraries.

## Design elements and principles
- We define data interfaces to represent **triples**, **quads**, **IRIs**, **blank nodes**, **literals** and **variables**.
- Instances of the interfaces created with different libraries should be interoperable.
- Interfaces do _not_ specify how instances are stored in memory.
- Interfaces mandate specific pre-defined methods such as `.equals()`.
- Given the necessity of methods, plain objects (JSON) cannot be used.
- Factory functions (e.g., `quad()`) or methods (e.g., `store.createQuad()`) create instances.
  - Should allow "upgrading" a plain object into a fully functional triple
- Interfaces may have additional implementation specific properties.
  A list of these properties maintained on the [RDFJS Representation Task Force wiki](https://github.com/rdfjs/representation-task-force/wiki/Additional-properties).

## Data interfaces

![UML diagram](img/class_diagram.png)

### Term

Abstract interface.

**Properties:**
- `String .termType` contains a value that identifies the concrete interface of the term, since Term itself is not directly instantiated.
  Possible values include `"iri"`, `"bnode"`, `"literal"`, and `"variable"`.
- `String .value` is refined by each interface which extends Term

**Methods:**

- `boolean .equals(Term other)` returns true if and only if the argument is a) of the same type b) has the same contents (value and, if applicable, type or language)
- `String .toCanonical()` returns a canonical string representation of the term.
  For IRIs, BlankNodes and Literals the [N-Triples canonical form](https://www.w3.org/TR/n-triples/#canonical-ntriples) must be used.
  Variables must return the variable name prefixed with a question mark (example: `?a`).
  DefaultGraph must return an empty string.

### NamedNode extends Term

**Properties:**

- `String .termType` contains the constant `"iri"`.
- `String .value` the IRI as a string (example: `http://example.org/resource`)

### BlankNode extends Term

**Properties:**

- `String .termType` contains the constant `"bnode"`.
- `String .value` blank node name as a string, without any serialization specific prefixes, e.g. when parsing, if the data was sourced from Turtle, remove _:, if it was sourced from RDF/XML, do not change the blank node name (example: `blank3`)

### Literal extends Term

**Properties:**

- `String .termType` contains the constant `"literal"`.
- `String .value` the text value, unescaped, without language or type (example: `Brad Pitt`)
- `String .language` the language as lowercase [BCP47](http://tools.ietf.org/html/bcp47) string (examples: `en`, `en-gb`) or an empty string if the literal has no language.
- `IRI .datatype` the datatype of the literal

If the literal has a language, the datatype IRI is `http://www.w3.org/1999/02/22-rdf-syntax-ns#langString`.
Otherwise, if no datatype is explicitly specified, the datatype IRI is `http://www.w3.org/2001/XMLSchema#string`.

### Variable extends Term

**Properties:**

- `String .termType` contains the constant `"variable"`.
- `String .value` the name of the variable without leading `?` (example: `a`)

### DefaultGraph extends Term

An instance of `DefaultGraph` represents the default graph.
It's only allowed to assign a `DefaultGraph` to the `.graph` property of a `Quad`.

**Properties:**

- `String .termType` contains the constant `"defaultGraph"`.
- `String .value` contains an empty string as constant value.

### Triple

Triple is an alias of Quad.

### Quad

**Properties:**

- `Term .subject` the subject, which is an IRI, a BlankNode or Variable.
- `Term .predicate` the predicate, which is an IRI or Variable.
- `Term .object` the object, which is an IRI, a Literal, a BlankNode or Variable.
- `Term .graph` the named graph, which is an IRI, DefaultGraph, BlankNode or Variable.

- `Boolean .equals(Quad other)` returns true if and only if the argument is a) of the same type b) has all components equal

**Methods:**

- `String .toCanonical()` returns a canonical string representation of the quad.
  The [N-Triples canonical form](https://www.w3.org/TR/n-triples/#canonical-ntriples) must be used.
  Terms must be represented as defined in the `.toCanonical()` method of the Term interface.
- `boolean .equals(Quad other)` returns true if and only if the argument is a) of the same type b) has all components equal
  Quads that contain a non-default graph must add the graph as defined in the [N-Quads specification](https://www.w3.org/TR/n-quads/).
  For that use case the definition of [N-Triples canonical form](https://www.w3.org/TR/n-triples/#canonical-ntriples) is extended:
  the whitespace following subject, predicate, object and graph MUST be a single space, (U+0020).

### DataFactory

For default values of the instance properties, see the individual [interface
definitions](#data-interfaces)

**Methods:**

- `NamedNode .namedNode(String iri)` returns a new instance of NamedNode.
- `BlankNode .blankNode([String identifier])` returns a new instance of BlankNode.
  The optional identifier parameter is assigned to `.value`.
  If the label parameter is undefined a new value is generated for each call.
- Literal .literal(String value, [String languageOrDatatype]) returns a new
  instance of Literal. If languageOrDatatype is an IRI, then a NamedNode is
  constructed with that IRI, and is used for the value of `.datatype`.
  Otherwise languageOrDatatype is used for the value of `.language`.
- `Variable .variable(String name)` returns a new instance of Variable. This method is optional.
- `DefaultGraph .defaultGraph()` returns an instance of DefaultGraph.
- `Quad .triple(Term subject, Term predicate, Term object)` returns a new instance of Quad with `.graph` set to DefaultGraph.
- `Quad .quad(Term subject, Term predicate, Term object, [Term graph])` returns a new instance of Quad.

## Stream interfaces

Streams are used only in a readable manner.
This requires only a single queue per stream, which simplifies implementations and doesn't have performance drawbacks, compared to writeable streams.

### Stream extends EventEmitter

**Methods:**

- `Quad .read()`
  This method pulls a quad out of the internal buffer and returns it.
  If there is no quad available, then it will return null.

**Events:**

- `readable`
  When a quad can be read from the stream, it will emit this event.

- `end`
  This event fires when there will be no more quads to read.

- `error`
  This event fires if any error occurs.
  The error message is forwarded to the event listener.

- `data`
  This event is emitted for every quad that can be read from the stream.
  The quad is forwarded to the event listener.

- `prefix`
  This event is emitted every time a prefix map occurs in the stream.
  The prefix map is forwarded to the event listener.

### Source

A Source is an object that emits quads.
It can contain quads but also generate them on the fly.
For example, parsers and transformations which generate quads can implement the Source interface.

- `Stream .match([Term|RegExp subject], [Term|RegExp predicate], [Term|RegExp object], [Term|RegExp graph])`
  Returns a stream that processes all quads matching the pattern.

### Sink

A Sink is an object that consumes quads.
It can store the quads or do some further processing.
For example serializers and transformations which consume quads can implement the Sink interface.

- `undefined .import(Stream stream)`
  Writes all quads from the stream to the sink.

### Store extends Source, Sink

A Store is an object that usually used to persist quads.
The interface allows removing quads, beside read and write access.
The quads can be stored locally or remotely.
Access to stores LDP or SPARQL endpoints can be implemented with a Store inteface.

- `EventEmitter .remove(Stream stream)`
  Removes all streamed quads.
  The `end` and `error` events are used like described in the `Stream` interface.

- `EventEmitter .removeMatches([Term|RegExp subject], [Term|RegExp predicate], [Term|RegExp object], [Term|RegExp graph])`
  All quads matching the pattern will be removed.
  The `end` and `error` events are used like described in the `Stream` interface.

- `EventEmitter .deleteGraph(IRI|String graph)`
  Deletes the given named graph.
  The `end` and `error` events are used like described in the `Stream` interface.
