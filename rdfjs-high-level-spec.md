# RDF/JS High Level API

## Mission Statement

**Status:** This document is a _draft_.<br>

## Data interfaces

### Dataset

**Important:** Unlike Arrays, RDF Datasets have the following properties:

* they are un-ordered (and quads within them cannot be accessed using `[]` notation)
* like Sets, they are unique (duplicate identical quads are not allowed)

**Properties:**

- `number .length` Number of quads in the dataset. (Similar to `Array.length`) 

**Static/Factory methods:**

- `static Promise<Dataset> import (Function factory, Stream stream)`
  creates a new Dataset and returns the result of calling `dataset.import(stream)` (see below)

**Array/Iterable compatibility API methods:**

These methods have the same semantics as corresponding methods on an `Array` instance. 

- `Boolean every (Function callback)`
  tests whether all quads in the dataset pass the test implemented by the provided function

- `Dataset filter (Function callback)`
  returns a new Dataset with all quads that pass the test implemented by the provided function

- `undefined forEach (Function callback)`
  executes a provided function once per each quad in the dataset

- `Boolean includes (Quad quad)`
  determines whether a dataset includes a certain quad, returning `true` or `false` as appropriate

- `Dataset map (Function callback)`
  returns a new dataset with the results of calling a provided function on every quad in this dataset

- `Boolean some (Function callback)`
  tests whether some quad in the dataset passes the test implemented by the provided function

**Set-like operations API methods:**

These methods deal with (un-ordered) set operations such as set union, intersection, etc.

- `Dataset add (Quad quad)`
  adds a given quad to the dataset, returns itself (this dataset)

- `Dataset addAll (Array<Quad>|Dataset quads)`
  adds all given quads to the dataset, returns itself (this dataset)

- `Dataset difference (Dataset other)`
  returns a set difference of this dataset and the given one

- `Dataset intersection (Dataset other)`
  returns a set intersection of this dataset and the given one

- `Dataset merge (Array<Quad>|Dataset other)`
  returns a set union operation, a new dataset comprised of this dataset merged with all of the quads in the given dataset. (Similar semantics to `Array.concat()`, except that Dataset is of course un-ordered.)

**Misc RDF Dataset API methods:**

- `Dataset clone ()`
  returns a copy of this dataset (and all its quads)

- `Boolean equals (Dataset other)`
  returns `true` if this dataset contains the same quads as the given one

- `Promise<Dataset> import (Stream stream)`

- `Dataset match (Term subject, Term predicate, Term object, Term graph)`
  synchronous method, returns a Dataset containing all the quads that match the given pattern. See also the RDFJS spec's `Stream.match()` method.

- `Dataset remove(Quad quad)`
  removes the given quad from this dataset, returns itself (this dataset)

- `Dataset removeMatches (Term subject, Term predicate, Term object, Term graph)`
  removes all quads that match the given pattern from this dataset, returns itself (this dataset)

- `Array toArray ()`
  returns an array containing all the quads in this dataset

- `String toCanonical ()`
  returns the results of a `normalize()` operation on the quads of this dataset as canonical [N-Triples](https://www.w3.org/TR/n-triples/#canonical-ntriples) string

- `Stream toStream ()`
  returns the stream containing all of the quads in this dataset

- `string toString ()`
