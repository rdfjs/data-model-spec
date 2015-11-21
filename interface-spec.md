# Interface Specification

**Status:** This document is a _draft_.<br>
**Purpose:** This document proposes a uniform interface to represent RDF data in low-level JavaScript libraries.

## Design elements and principles
- We define data interfaces to represent **triples**, **quads**, **IRIs**, **blank nodes**, **literals** and **variables**.
- Instances of the interfaces created with different libraries should be interoperable.
- Interfaces do _not_ specify how instances are stored in memory.
- Interfaces mandate specific pre-defined methods such as `.equals()`.
- Given the necessity of methods, plain objects (JSON) cannot be used.
- Factory functions (e.g., `triple()`) or methods (e.g., `store.createTriple()`) create instances.
  - Should allow "upgrading" a plain object into a fully functional triple

## Data interfaces

### Node

**Properties:**

- `String .value` is refined by each interface which extends Node

**Methods:**

- `.equals(Node other)` returns true if and only if the argument is a) of the same type b) has the same contents (value and, if applicable, type or language)

### IRI extends Node

**Properties:**

- `String .value` the IRI as a string (example: `http://example.org/resource`)

### BlankNode extends Node

**Properties:**

- `String .value` blank node name as a string (example: `_:blank3`)

### Literal extends Node

**Properties:**

- `String .value` the text value, unescaped, without language or type (example: `Brad Pitt`)
- `String .language` the language as two character lower case BCP47 string
- `String .datatype` the datatype IRI as string

### NodeVariable extends Node

**Properties:**

- `String .value` the name of the variable (example: `?a`)

### Triple

**Properties:**

- `Node .subject` the subject, which is an IRI, a BlankNode or NodeVariable.
- `Node .predicate` the predicate, which is an IRI or NodeVariable.
- `Node .object` the object, which is an IRI, a Literal, a BlankNode or NodeVariable.

**Methods:**

- `Boolean .equals(Triple other)` returns true if and only if the argument is a) of the same type b) has all components equal

### Quad extends Triple

**Properties:**

- `Node .graph` the named graph, which is an IRI, a BlankNode or NodeVariable.

### DataFactory

**Methods:**

- `.iri(String iri)` returns a new instance of IRI.
- `.blankNode()` returns a new instance of BlankNode.
- `.literal(String value, String language, String datatype)` returns a new instance of Literal.
- `.variable(String name)` returns a new instance of NodeVariable. This method is optional.
- `.triple([Object])` returns a new instance of Triple. 
- `.quad([Object])` returns a new instance of Quad.