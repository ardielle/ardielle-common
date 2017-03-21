# The Rdl schema


This defines the schema for a schema, the output of the RDL parser. This can be
used to represent schemas in JSON, Protobuf, Avro, etc, from a single
definition.

This schema has the following attributes:

| Attribute | Value |
|-----------|-------|
| version   | 3     |


## Types

### Identifier

All names need to be of this restricted string type

`Identifier` is a `String` type with the following options:

| Option  | Value                       | Notes |
|---------|-----------------------------|-------|
| pattern | `"[a-zA-Z_]+[a-zA-Z_0-9]*`" |       |


### NamespacedIdentifier

A Namespace is a dotted compound name, using reverse domain name order (i.e.
"com.yahoo.auth")

`NamespacedIdentifier` is a `String` type with the following options:

| Option  | Value                                                    | Notes |
|---------|----------------------------------------------------------|-------|
| pattern | `"([a-zA-Z_]+[a-zA-Z_0-9]*)(\.[a-zA-Z_]+[a-zA-Z_0-9])*`" |       |


### TypeName

The identifier for an already-defined type

`TypeName` is an alias of type `Identifier`

### TypeRef

A type reference can be a simple name, or also a namespaced name.

`TypeRef` is an alias of type `NamespacedIdentifier`

### BaseType
`BaseType` is an `Enum` of the following values:

| Value     | Description |
|-----------|-------------|
| Bool      |             |
| Int8      |             |
| Int16     |             |
| Int32     |             |
| Int64     |             |
| Float32   |             |
| Float64   |             |
| Bytes     |             |
| String    |             |
| Timestamp |             |
| Symbol    |             |
| UUID      |             |
| Array     |             |
| Map       |             |
| Struct    |             |
| Enum      |             |
| Union     |             |
| Any       |             |


### ExtendedAnnotation

ExtendedAnnotation - parsed and preserved, but has no defined meaning in RDL.
Such annotations must begin with "x_", and may have an associated string
literal value (the value will be "" if the annotation is just a flag).

`ExtendedAnnotation` is a `String` type with the following options:

| Option  | Value               | Notes |
|---------|---------------------|-------|
| pattern | `"x_[a-zA-Z_0-9]*`" |       |


### TypeDef

TypeDef is the basic type definition.

`TypeDef` is a `Struct` type with the following fields:

| Name        | Type                                                        | Options  | Description                                                                    | Notes |
|-------------|-------------------------------------------------------------|----------|--------------------------------------------------------------------------------|-------|
| type        | [TypeRef](#typeref)                                         |          | The type this type is derived from. For base types, it is the same as the name |       |
| name        | [TypeName](#typename)                                       |          | The name of the type                                                           |       |
| comment     | String                                                      | optional | The comment for the type                                                       |       |
| annotations | Map&lt;[ExtendedAnnotation](#extendedannotation),String&gt; | optional | additional annotations starting with "x_"                                      |       |


### AliasTypeDef

AliasTypeDef is used for type definitions that add no additional attributes,
and thus just create an alias

`AliasTypeDef` is a `Struct` type with the following fields:

| Name        | Type                                                        | Options  | Description                                                                    | Notes                      |
|-------------|-------------------------------------------------------------|----------|--------------------------------------------------------------------------------|----------------------------|
| type        | [TypeRef](#typeref)                                         |          | The type this type is derived from. For base types, it is the same as the name | [from [TypeDef](#typedef)] |
| name        | [TypeName](#typename)                                       |          | The name of the type                                                           | [from [TypeDef](#typedef)] |
| comment     | String                                                      | optional | The comment for the type                                                       | [from [TypeDef](#typedef)] |
| annotations | Map&lt;[ExtendedAnnotation](#extendedannotation),String&gt; | optional | additional annotations starting with "x_"                                      | [from [TypeDef](#typedef)] |


### BytesTypeDef

Bytes allow the restriction by fixed size, or min/max size.

`BytesTypeDef` is a `Struct` type with the following fields:

| Name        | Type                                                        | Options  | Description                                                                    | Notes                      |
|-------------|-------------------------------------------------------------|----------|--------------------------------------------------------------------------------|----------------------------|
| type        | [TypeRef](#typeref)                                         |          | The type this type is derived from. For base types, it is the same as the name | [from [TypeDef](#typedef)] |
| name        | [TypeName](#typename)                                       |          | The name of the type                                                           | [from [TypeDef](#typedef)] |
| comment     | String                                                      | optional | The comment for the type                                                       | [from [TypeDef](#typedef)] |
| annotations | Map&lt;[ExtendedAnnotation](#extendedannotation),String&gt; | optional | additional annotations starting with "x_"                                      | [from [TypeDef](#typedef)] |
| size        | Int32                                                       | optional | Fixed size                                                                     |                            |
| minSize     | Int32                                                       | optional | Min size                                                                       |                            |
| maxSize     | Int32                                                       | optional | Max size                                                                       |                            |


### StringTypeDef

Strings allow the restriction by regular expression pattern or by an explicit
set of values. An optional maximum size may be asserted

`StringTypeDef` is a `Struct` type with the following fields:

| Name        | Type                                                        | Options  | Description                                                                    | Notes                      |
|-------------|-------------------------------------------------------------|----------|--------------------------------------------------------------------------------|----------------------------|
| type        | [TypeRef](#typeref)                                         |          | The type this type is derived from. For base types, it is the same as the name | [from [TypeDef](#typedef)] |
| name        | [TypeName](#typename)                                       |          | The name of the type                                                           | [from [TypeDef](#typedef)] |
| comment     | String                                                      | optional | The comment for the type                                                       | [from [TypeDef](#typedef)] |
| annotations | Map&lt;[ExtendedAnnotation](#extendedannotation),String&gt; | optional | additional annotations starting with "x_"                                      | [from [TypeDef](#typedef)] |
| pattern     | String                                                      | optional | A regular expression that must be matched. Mutually exclusive with values      |                            |
| values      | Array&lt;String&gt;                                         | optional | A set of allowable values                                                      |                            |
| minSize     | Int32                                                       | optional | Min size                                                                       |                            |
| maxSize     | Int32                                                       | optional | Max size                                                                       |                            |


### Number

A numeric is any of the primitive numeric types

`Number` is a `Union` of following types:

| Variant |
|---------|
| Int8    |
| Int16   |
| Int32   |
| Int64   |
| Float32 |
| Float64 |


### NumberTypeDef

A number type definition allows the restriction of numeric values.

`NumberTypeDef` is a `Struct` type with the following fields:

| Name        | Type                                                        | Options  | Description                                                                    | Notes                      |
|-------------|-------------------------------------------------------------|----------|--------------------------------------------------------------------------------|----------------------------|
| type        | [TypeRef](#typeref)                                         |          | The type this type is derived from. For base types, it is the same as the name | [from [TypeDef](#typedef)] |
| name        | [TypeName](#typename)                                       |          | The name of the type                                                           | [from [TypeDef](#typedef)] |
| comment     | String                                                      | optional | The comment for the type                                                       | [from [TypeDef](#typedef)] |
| annotations | Map&lt;[ExtendedAnnotation](#extendedannotation),String&gt; | optional | additional annotations starting with "x_"                                      | [from [TypeDef](#typedef)] |
| min         | [Number](#number)                                           | optional | Min value                                                                      |                            |
| max         | [Number](#number)                                           | optional | Max value                                                                      |                            |


### ArrayTypeDef

Array types can be restricted by item type and size

`ArrayTypeDef` is a `Struct` type with the following fields:

| Name        | Type                                                        | Options     | Description                                                                    | Notes                      |
|-------------|-------------------------------------------------------------|-------------|--------------------------------------------------------------------------------|----------------------------|
| type        | [TypeRef](#typeref)                                         |             | The type this type is derived from. For base types, it is the same as the name | [from [TypeDef](#typedef)] |
| name        | [TypeName](#typename)                                       |             | The name of the type                                                           | [from [TypeDef](#typedef)] |
| comment     | String                                                      | optional    | The comment for the type                                                       | [from [TypeDef](#typedef)] |
| annotations | Map&lt;[ExtendedAnnotation](#extendedannotation),String&gt; | optional    | additional annotations starting with "x_"                                      | [from [TypeDef](#typedef)] |
| items       | [TypeRef](#typeref)                                         | default=Any | The type of the items, default to any type                                     |                            |
| size        | Int32                                                       | optional    | If present, indicate the fixed size.                                           |                            |
| minSize     | Int32                                                       | optional    | If present, indicate the min size                                              |                            |
| maxSize     | Int32                                                       | optional    | If present, indicate the max size                                              |                            |


### MapTypeDef

Map types can be restricted by key type, item type and size

`MapTypeDef` is a `Struct` type with the following fields:

| Name        | Type                                                        | Options        | Description                                                                    | Notes                      |
|-------------|-------------------------------------------------------------|----------------|--------------------------------------------------------------------------------|----------------------------|
| type        | [TypeRef](#typeref)                                         |                | The type this type is derived from. For base types, it is the same as the name | [from [TypeDef](#typedef)] |
| name        | [TypeName](#typename)                                       |                | The name of the type                                                           | [from [TypeDef](#typedef)] |
| comment     | String                                                      | optional       | The comment for the type                                                       | [from [TypeDef](#typedef)] |
| annotations | Map&lt;[ExtendedAnnotation](#extendedannotation),String&gt; | optional       | additional annotations starting with "x_"                                      | [from [TypeDef](#typedef)] |
| keys        | [TypeRef](#typeref)                                         | default=String | The type of the keys, default to String.                                       |                            |
| items       | [TypeRef](#typeref)                                         | default=Any    | The type of the items, default to Any type                                     |                            |
| size        | Int32                                                       | optional       | If present, indicates the fixed size.                                          |                            |
| minSize     | Int32                                                       | optional       | If present, indicate the min size                                              |                            |
| maxSize     | Int32                                                       | optional       | If present, indicate the max size                                              |                            |


### StructFieldDef

Each field in a struct_field_spec is defined by this type

`StructFieldDef` is a `Struct` type with the following fields:

| Name        | Type                                                        | Options       | Description                                               | Notes |
|-------------|-------------------------------------------------------------|---------------|-----------------------------------------------------------|-------|
| name        | [Identifier](#identifier)                                   |               | The name of the field                                     |       |
| type        | [TypeRef](#typeref)                                         |               | The type of the field                                     |       |
| optional    | Bool                                                        | default=false | The field may be omitted even if specified                |       |
| default     | Any                                                         | optional      | If field is absent, what default value should be assumed. |       |
| comment     | String                                                      | optional      | The comment for the field                                 |       |
| items       | [TypeRef](#typeref)                                         | optional      | For map or array fields, the type of the items            |       |
| keys        | [TypeRef](#typeref)                                         | optional      | For map type fields, the type of the keys                 |       |
| annotations | Map&lt;[ExtendedAnnotation](#extendedannotation),String&gt; | optional      | additional annotations starting with "x_"                 |       |


### StructTypeDef

A struct can restrict specific named fields to specific types. By default, any
field not specified is allowed, and can be of any type. Specifying closed
means only those fields explicitly

`StructTypeDef` is a `Struct` type with the following fields:

| Name        | Type                                                        | Options       | Description                                                                                  | Notes                      |
|-------------|-------------------------------------------------------------|---------------|----------------------------------------------------------------------------------------------|----------------------------|
| type        | [TypeRef](#typeref)                                         |               | The type this type is derived from. For base types, it is the same as the name               | [from [TypeDef](#typedef)] |
| name        | [TypeName](#typename)                                       |               | The name of the type                                                                         | [from [TypeDef](#typedef)] |
| comment     | String                                                      | optional      | The comment for the type                                                                     | [from [TypeDef](#typedef)] |
| annotations | Map&lt;[ExtendedAnnotation](#extendedannotation),String&gt; | optional      | additional annotations starting with "x_"                                                    | [from [TypeDef](#typedef)] |
| fields      | Array&lt;[StructFieldDef](#structfielddef)&gt;              |               | The fields in this struct. By default, open Structs can have any fields in addition to these |                            |
| closed      | Bool                                                        | default=false | indicates that only the specified fields are acceptable. Default is open (any fields)        |                            |


### EnumElementDef

EnumElementDef defines one of the elements of an Enum

`EnumElementDef` is a `Struct` type with the following fields:

| Name    | Type                      | Options  | Description                           | Notes |
|---------|---------------------------|----------|---------------------------------------|-------|
| symbol  | [Identifier](#identifier) |          | The identifier representing the value |       |
| comment | String                    | optional | the comment for the element           |       |


### EnumTypeDef

Define an enumerated type. Each value of the type is represented by a symbolic
identifier.

`EnumTypeDef` is a `Struct` type with the following fields:

| Name        | Type                                                        | Options  | Description                                                                    | Notes                      |
|-------------|-------------------------------------------------------------|----------|--------------------------------------------------------------------------------|----------------------------|
| type        | [TypeRef](#typeref)                                         |          | The type this type is derived from. For base types, it is the same as the name | [from [TypeDef](#typedef)] |
| name        | [TypeName](#typename)                                       |          | The name of the type                                                           | [from [TypeDef](#typedef)] |
| comment     | String                                                      | optional | The comment for the type                                                       | [from [TypeDef](#typedef)] |
| annotations | Map&lt;[ExtendedAnnotation](#extendedannotation),String&gt; | optional | additional annotations starting with "x_"                                      | [from [TypeDef](#typedef)] |
| elements    | Array&lt;[EnumElementDef](#enumelementdef)&gt;              |          | The enumeration of the possible elements                                       |                            |


### UnionTypeDef

Define a type as one of any other specified type.

`UnionTypeDef` is a `Struct` type with the following fields:

| Name        | Type                                                        | Options  | Description                                                                        | Notes                      |
|-------------|-------------------------------------------------------------|----------|------------------------------------------------------------------------------------|----------------------------|
| type        | [TypeRef](#typeref)                                         |          | The type this type is derived from. For base types, it is the same as the name     | [from [TypeDef](#typedef)] |
| name        | [TypeName](#typename)                                       |          | The name of the type                                                               | [from [TypeDef](#typedef)] |
| comment     | String                                                      | optional | The comment for the type                                                           | [from [TypeDef](#typedef)] |
| annotations | Map&lt;[ExtendedAnnotation](#extendedannotation),String&gt; | optional | additional annotations starting with "x_"                                          | [from [TypeDef](#typedef)] |
| variants    | Array&lt;[TypeRef](#typeref)&gt;                            |          | The type names of constituent types. Union types get expanded, this is a flat list |                            |


### Type

A Type can be specified by any of the above specialized Types, determined by
the value of the the 'type' field

`Type` is a `Union` of following types:

| Variant                         |
|---------------------------------|
| [BaseType](#basetype)           |
| [StructTypeDef](#structtypedef) |
| [MapTypeDef](#maptypedef)       |
| [ArrayTypeDef](#arraytypedef)   |
| [EnumTypeDef](#enumtypedef)     |
| [UnionTypeDef](#uniontypedef)   |
| [StringTypeDef](#stringtypedef) |
| [BytesTypeDef](#bytestypedef)   |
| [NumberTypeDef](#numbertypedef) |
| [AliasTypeDef](#aliastypedef)   |


### ResourceInput

ResourceOutput defines input characteristics of a Resource

`ResourceInput` is a `Struct` type with the following fields:

| Name        | Type                                                        | Options       | Description                                                                        | Notes |
|-------------|-------------------------------------------------------------|---------------|------------------------------------------------------------------------------------|-------|
| name        | [Identifier](#identifier)                                   |               | the formal name of the input                                                       |       |
| type        | [TypeRef](#typeref)                                         |               | The type of the input                                                              |       |
| comment     | String                                                      | optional      | The optional comment                                                               |       |
| pathParam   | Bool                                                        | default=false | true of this input is a path parameter                                             |       |
| queryParam  | String                                                      | optional      | if present, the name of the query param name                                       |       |
| header      | String                                                      | optional      | If present, the name of the header the input is associated with                    |       |
| pattern     | String                                                      | optional      | If present, the pattern associated with the pathParam (i.e. wildcard path matches) |       |
| default     | Any                                                         | optional      | If present, the default value for optional params                                  |       |
| optional    | Bool                                                        | default=false | If present, indicates that the input is optional                                   |       |
| flag        | Bool                                                        | default=false | If present, indicates the queryparam is of flag style (no value)                   |       |
| context     | String                                                      | optional      | If present, indicates the parameter comes form the implementation context          |       |
| annotations | Map&lt;[ExtendedAnnotation](#extendedannotation),String&gt; | optional      | additional annotations starting with "x_"                                          |       |


### ResourceOutput

ResourceOutput defines output characteristics of a Resource

`ResourceOutput` is a `Struct` type with the following fields:

| Name        | Type                                                        | Options       | Description                                                            | Notes |
|-------------|-------------------------------------------------------------|---------------|------------------------------------------------------------------------|-------|
| name        | [Identifier](#identifier)                                   |               | the formal name of the output                                          |       |
| type        | [TypeRef](#typeref)                                         |               | The type of the output                                                 |       |
| header      | String                                                      |               | the name of the header associated with this output                     |       |
| comment     | String                                                      | optional      | The optional comment for the output                                    |       |
| optional    | Bool                                                        | default=false | If present, indicates that the output is optional (the server decides) |       |
| annotations | Map&lt;[ExtendedAnnotation](#extendedannotation),String&gt; | optional      | additional annotations starting with "x_"                              |       |


### ResourceAuth

ResourceAuth defines authentication and authorization attributes of a resource.
Presence of action, resource, or domain implies authentication; the
authentication flag alone is required only when no authorization is done.

`ResourceAuth` is a `Struct` type with the following fields:

| Name         | Type   | Options       | Description                                                        | Notes |
|--------------|--------|---------------|--------------------------------------------------------------------|-------|
| authenticate | Bool   | default=false | if present and true, then the requester must be authenticated      |       |
| action       | String | optional      | the action to authorize access to. This forces authentication      |       |
| resource     | String | optional      | the resource identity to authorize access to                       |       |
| domain       | String | optional      | if present, the alternate domain to check access to. This is rare. |       |


### ExceptionDef

ExceptionDef describes the exception a symbolic response code maps to.

`ExceptionDef` is a `Struct` type with the following fields:

| Name    | Type   | Options  | Description                            | Notes |
|---------|--------|----------|----------------------------------------|-------|
| type    | String |          | The type of the exception              |       |
| comment | String | optional | the optional comment for the exception |       |


### Resource

A Resource of a REST service

`Resource` is a `Struct` type with the following fields:

| Name         | Type                                                        | Options    | Description                                                                                    | Notes |
|--------------|-------------------------------------------------------------|------------|------------------------------------------------------------------------------------------------|-------|
| type         | [TypeRef](#typeref)                                         |            | The type of the resource                                                                       |       |
| method       | String                                                      |            | The method for the action (typically GET, POST, etc for HTTP access)                           |       |
| path         | String                                                      |            | The resource path template                                                                     |       |
| comment      | String                                                      | optional   | The optional comment                                                                           |       |
| inputs       | Array&lt;[ResourceInput](#resourceinput)&gt;                | optional   | An Array named inputs                                                                          |       |
| outputs      | Array&lt;[ResourceOutput](#resourceoutput)&gt;              | optional   | An Array of named outputs                                                                      |       |
| auth         | [ResourceAuth](#resourceauth)                               | optional   | The optional authentication or authorization directive                                         |       |
| expected     | String                                                      | default=OK | The expected symbolic response code                                                            |       |
| alternatives | Array&lt;String&gt;                                         | optional   | The set of alternative but non-error response codes                                            |       |
| exceptions   | Map&lt;String,[ExceptionDef](#exceptiondef)&gt;             | optional   | A map of symbolic response code to Exception definitions                                       |       |
| async        | Bool                                                        | optional   | A hint to server implementations that this resource would be better implemented with async I/O |       |
| annotations  | Map&lt;[ExtendedAnnotation](#extendedannotation),String&gt; | optional   | additional annotations starting with "x_"                                                      |       |
| consumes     | Array&lt;String&gt;                                         | optional   | Optional hint for resource acceptable input types                                              |       |
| produces     | Array&lt;String&gt;                                         | optional   | Optional hint for resource output content types                                                |       |
| name         | [Identifier](#identifier)                                   | optional   | The optional name of the resource                                                              |       |


### Schema

A Schema is a container for types and resources. It is self-contained (no
external references). and is the output of the RDL parser.

`Schema` is a `Struct` type with the following fields:

| Name      | Type                                          | Options  | Description                                     | Notes |
|-----------|-----------------------------------------------|----------|-------------------------------------------------|-------|
| namespace | [NamespacedIdentifier](#namespacedidentifier) | optional | The namespace for the schema                    |       |
| name      | [Identifier](#identifier)                     | optional | The name of the schema                          |       |
| version   | Int32                                         | optional | The version of the schema                       |       |
| comment   | String                                        | optional | The comment for the entire schema               |       |
| types     | Array&lt;[Type](#type)&gt;                    | optional | The types this schema defines.                  |       |
| resources | Array&lt;[Resource](#resource)&gt;            | optional | The resources for a service this schema defines |       |
| base      | String                                        | optional | the base path for resources in the schema.      |       |

