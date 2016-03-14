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

- `boolean .equals(Term other)` returns true if and only if the strings returned by .toCanonical() are equal
- `String .toCanonical()` returns a canonical string representation of the term.
  For IRIs, BlankNodes and Literals the [N-Triples canonical form](https://www.w3.org/TR/n-triples/#canonical-ntriples) must be used.
  Variables must return the variable name prefixed with a question mark (example: `?a`).

### IRI extends Term

**Properties:**

- `String .termType` contains the constant `"iri"`.
- `String .iri` the IRI as a string (example: `http://example.org/resource`)

### BlankNode extends Term

**Properties:**

- `String .termType` contains the constant `"bnode"`.
- `String .id` blank node identifier as a string, without any serialization specific prefixes, e.g. when parsing, if the data was sourced from Turtle, remove _:, if it was sourced from RDF/XML, do not change the blank node name (example: `blank3`)

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
- `String .name` the name of the variable without leading `?` (example: `a`)

### DefaultGraph extends Term

An instance of `DefaultGraph` represents the default graph.
It's only allowed to assign a `DefaultGraph` to the `.graph` property of a `Quad`. 

**Properties:**

- `String .termType` contains the constant `"defaultGraph"`.

### Triple
Triple is an alias of Quad.
Triples always have `.graph` set to DefaultGraph.

### Quad extends Triple

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
- `boolean .equals(Triple other)` returns true if and only if the argument is a) of the same type b) has all components equal
  Quads that contain a non-default graph must add the graph as defined in the [N-Quads specification](https://www.w3.org/TR/n-quads/).
  For that use case the definition of [N-Triples canonical form](https://www.w3.org/TR/n-triples/#canonical-ntriples) is extended:
  the whitespace following subject, predicate, object and graph MUST be a single space, (U+0020).

### DataFactory

**Methods:**

- `IRI .iri(String iri)` returns a new instance of IRI.
- `BlankNode .blankNode()` returns a new instance of BlankNode.
- `Literal .literal(String value, String language, String datatype)` returns a new instance of Literal.
- `Variable .variable(String name)` returns a new instance of Variable. This method is optional.
- `DefaultGraph .defaultGraph()` returns an instance of DefaultGraph.
- `Triple .triple(Term subject, Term predicate, Term object, [Term graph])` returns a new instance of Triple.
- `Quad .quad(Term subject, Term predicate, Term object, [Term graph])` returns a new instance of Quad.
