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

TODO: read/write or read-only?

**Methods:**

- `.equals(Node other)` returns true if and only if the argument is a) of the same type b) has the same contents (value and, if applicable, type or language)

TODO: to what extent should we use typed signatures (`.equals(Node other)`) versus actual JavaScript signatures (`.equals(other)`). The benefit of typed signatures is that you see the type inline; the drawback is that it is more specific than JavaScript itself. For ease of use, JavaScript might be preferred, specifying types in the explanation (or jsdoc-style).

### IRI extends Node

**Properties:**

- `String .value` the IRI as a string (example: `http://example.org/resource`)

### BlankNode extends Node

**Properties:**

- `String .value` blank node name as a string without leading `_:` (example: `blank3`)

### Literal extends Node

**Properties:**

- `String .value` the text value, unescaped, without language or type (example: `Brad Pitt`)
- `String .language` the language as lowercase [BCP47](http://tools.ietf.org/html/bcp47) string (examples: `en`, `en-gb`)
- `String .datatype` the datatype IRI as string

TODO: What if the literal has no language? Does it always have a datatype?

### NodeVariable extends Node

TODO: Why not just Variable?

**Properties:**

- `String .value` the name of the variable without leading `?` (example: `a`)

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

TODO: Do we need to define a different interface, or is a quad simply a triple with a graph different from undefined?

### DataFactory

**Methods:**

- `.iri(String iri)` returns a new instance of IRI.
- `.blankNode()` returns a new instance of BlankNode.
- `.literal(String value, String language, String datatype)` returns a new instance of Literal.
- `.variable(String name)` returns a new instance of NodeVariable. This method is optional.
- `.triple([Object])` returns a new instance of Triple. 
- `.quad([Object])` returns a new instance of Quad.

TODO: `.blankNode()` could/should have an optional suggested label.

TODO: `.literal()` could have a more intelligent constructor (valid language strings are not valid IRIs and vice-versa).

TODO: `.literal()` should support IRI datatype as well

TODO: Can/should `.literal()` accept built-in types like integers?

TODO: `.variable` is marked "optional", but what does this mean? Perhaps we need different stages of compatibility. Are quads optional, for instance?

TODO: Can `.triple` and `.quad` also support three/four-part constructors?

TODO: Can `.triple` and `.quad` also support simple strings, or should they be Nodes?

TODO: Is the argument of `.triple` and `.quad` optional (or why the brackets)?

TODO: Can `.triple` and `.quad` just be aliases of each other?
