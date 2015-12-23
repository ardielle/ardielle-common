# RDL - Resource Description Language

RDL is a machine-readable description of a schema that describes data types,
and resources and data tables using those types. Such a schema can be used to describe
HTTP web services and storage tables, as well as serve as the source of truth for data encoding
mechanisms like Protocol Buffers and Avro, as well as augment JSON and other encoding schemes by
providing data validation.

RDL is particularly useful in describing REST-based APIs, providing a first class artifact that
describes the API in an implementation-independent manner.

A summary of RDL types and syntax is described [here](https://ardielle.github.io/), and the
definition of the resulting schema, itself expressed in RDL,
is [here](https://github.com/ardielle/ardielle-common/blob/master/rdl.rdl).

Note that the [JSON representation](https://github.com/ardielle/ardielle-common/blob/master/rdl.json)
shows the schema data object that the <code>rdl.rdl</code> file compiles to, for reference. Tools can start
from either representation, they are equivalent.

The generated markdown docs for <code>rdl.rdl</code> are
[here](https://github.com/ardielle/ardielle-common/blob/master/rdl.md).

Additional documentation on the TBin encoding format is [here](https://github.com/ardielle/ardielle-common/blob/master/tbin.md).



