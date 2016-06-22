# The TBin (Tagged Binary) serialization format

## Overview

TBin is a simple binary serialization schema that supports all the types in
[RDL](https://ardielle.github.io). All values get tagged
with a type. Struct field names get interned to prevent repetition of the strings.
Some tiny values are specially encoded to further pack the data.
If a schema is provided, or can be derived from an implementation langauges type reflection
facilities (i.e. as in Java and Go), it can provide additional data to pack homogenous arrays
and repetitive Structs.

Most implementations also provide encoding/decoding to more generic Map based structures, as
well as mapping to/from native structured types.

Regardless of encoding optimizations, the resulting string can be decoded without additional
information, i.e. the schema is not required.

## Primitive type encoding

### Basic Type Tags
16 basic tags are defined as _type_ tags, they can be used both as the tag for a specific data value, and
also can be used to specify a primitive type when expressing user-defined types. The first 16 tags, along with the ANY
tag, taking the values 0x00 - 0x10, are as follows:

| Tag | Value |
|-----|-------|
| NULL_TAG | 0x00 |
| BOOL_TAG | 0x01 |
| INT8_TAG | 0x02 |
| INT16_TAG | 0x03 |
| INT32_TAG | 0x04 |
| INT64_TAG | 0x05 |
| FLOAT32_TAG | 0x06 |
| FLOAT64_TAG | 0x07 |
| BYTES_TAG | 0x08 |
| STRING_TAG | 0x09  |
| TIMESTAMP_TAG | 0x0a |
| SYMBOL_TAG | 0x0b |
| UUID_TAG | 0x0c |
| ARRAY_TAG | 0x0d |
| MAP_TAG | 0x0e |
| STRUCT_TAG | 0x0f |


The null value is encoded as single NULL_TAG, i.e. a byte with a value of zero.

    NULL_TAG

Other values are encoded with a tag, followed by tag-specific data bytes.

### Variable Precision Integer (varint) Encoding 

Integer values are encoded, 7 bits at a time. The last byte of the sequence has a zero top bit, all other
bytes have the top bit set as a continuation marker. For example:

| value | byte sequence (hex) |
|--------|---------|
| 0 | 00 |
| 127 | 7f |
| 128 | 11 00 |
| 129 | 11 01 |
| 255 | 11 7f |
| 256 | 12 7f |
| 16383 | ff 7f |
| 16384 | 11 10 01 |

Signed integers are encoded with a zig-zag encoding format that maps signed integers to unsigned integers so that numbers
with a small absolute value (for instance, -1) also have a small varint encoded value:

    u = (n << 1) ^ (n >> 31)

For example:

| Signed Int | ZigZag value |
|--------|---------|
| 0 | 0 |
| -1 | 1 |
| 1 | 2 |
| -2 | 3 |
| 2 | 4 |

In general, all integer values use the zigzag signed encoding, whereas counts for complex types are
expressed as unsigned varints.

### Boolean Encoding

Boolean values are encoded as a BOOL_TAG followed by a numeric value, 0 meaning false:

    BOOL_TAG <n>

### Numeric Type Encoding

The 4 integer types are encoded with a tag for the type, followed by a signed zigzag varint for the value:

    INT8_TAG <n>
    INT16_TAG <n>
    INT32_TAG <n>
    INT64_TAG <n>

Floating point numbers are encoded as fixed size quantities, with the bits corresponding to the IEEE spec:

    FLOAT32_TAG <4 bytes>
    FLOAT64_TAG <8 bytes>


### Strings and Bytes Encoding

Bytes are encoded as an unsigned varint length, followed by that many bytes:

    BYTES_TAG <length> <bytes...>

Similarly, strings are encoded as an unsigned varint length, followed by that many UTF8 bytes.

    STRING_TAG <length> <utf8bytes...>
	"test" -> 09 04 74 65 73 74
	
If the string is small enough (< 32 bytes), then it is encoded with a special TINY_STR_TAG, which
has the length embedded in 5 bits of the tag:

    (TINY_STR_TAG | strlen(s)) <utf8bytes>
    "test" -> 24 74 65 73 74

### Array and Map Encoding

Arrays get encoded as a tag, followed by the length, and then every value is encoded.

    ARRAY_TAG <length> <value>*

Every value gets encoded with its own tag, allowing heterogeneous arrays. For packed arrays, 
they should be part of a user-defined type, described later.

Maps are encoded similarly, except the length is the number of key/value mappings, each of which
is expressed as a value:

    MAP_TAG <length> <key> <value>*

Both keys and values are encoded with its own tag, to get homogenous packed maps, user defined types,
as described later, must be used.

### Struct encoding
Structs get the number of fields encoded as an unsigned varint, followed by that many key/value pairs.

    STRUCT_TAG <count> (<field_id> [<namelen> <utf8bytes>] <value>)*

Structs are optimized such that the key names are interned as you go. So, repeated structs will tend to have a short
integer as the key, rather than the string. This is done by encoding a number as the key, and for the first occurrence
in the stream, following that by the string itself. Subsequent references to that key only include the number.

For example, the following polyline, consisting of an array of points:

	{
	   "points": [
	       {"x": 1, "y": 11},
	       {"x": 2, "y": 22},
	       {"x": 3, "y": 33}
	   ]
	}

gets encoded as:

    18                                ;; the tbin version tag
    0f  01                            ;; struct tag, count of 1
        00  06  70 6f  69  6e  74  73 ;;   key (id = 0), followed by the string "points"
        0d  03                        ;;   value, an array tag with a count of 3
            0e  02					  ;;    struct tag, count of 2
                01 1 78 02            ;;     key 'x' (first time), val = 1 (zigzag encoded)
                02 1 79 16            ;;     key 'y' (first time), val = 11
            0e  02					  ;;    struct tag, count of 2
                01 04                 ;;     key 'x' (second time), val = 2
                02 2c                 ;;     key 'y' (second time), val = 22
            0e  02					  ;;    struct tag, count of 2
                01 06                 ;;     key 'x' (second time), val = 3
                02 42                 ;;     key 'y' (second time), val = 33


### Other simple types

Timestamps are represented as a float64 number of seconds since the epoch, and thus
represented just like a float64:

    TIMESTAMP_TAG <8 byte IEEE double precision float>

A UUID is encoded as a fixed size 16 byte array:

    UUID_TAG <16 bytes>

A Symbol is an interned string, much like field names. The first occurrence of it should
define its name, subsequent references to it should only include its id:

    SYMBOL_TAG <symid> [<namelen> <namebytes>]


### Schema-informed encoding and User Defined Types

If you have a schema for the data, the encoding may be even more compressed.
The struct layout itself can generally be determined, eliminating field level tags. This is done by introducing
typedefs, which define new tags in this encoded stream.

| Tag | Value |
|-----|-------|
| ANY_TAG | 0x10 |
| DEF_ARRAY_TAG | 0x11 |
| DEF_MAP_TAG | 0x12 |
| DEF_STRUCT_TAG | 0x13 |
| DEF_UNION_TAG | 0x14 |
| DEF_ENUM_TAG | 0x15 |

The ANY_TAG is introduced for typedefs, it can be used where a type tag is required, but the decision of
type is to be included in the data. This is what happens with RDL `optional` fields in structs, or when
the key type in a map is specified, but the value is not.

Note that languages with type reflection abilities can usually generate such a schema on the fly from the
language types. This is a fairly common case.

Suppose you had the follow RDL schema for the above data:

	type Point Struct {
	    Int32 x;
	    Int32 y;
	}

	type Polyline Struct {
	    Array<Point> points;
	}

Then, the same data as above can be encoded like this:

	18
	//first the typedefs
	40 13 02		//defstruct with 2 fields
		01 78 04		//		“x” is type 0x04 (Int32)
		01 79 04		//		“y” is type 0x04 (Int32)
	41 11 40		//defarray of 0x40, Point
	42 13 01		//defstruct with 1 field
		06 70 6f 69 6e 74 73 41
		// “points” if type 0x41 (Array<Point>)
	//now the data follows
	42 //polyline
		03 		//3 items in the points array
		02 16	//1, 11 the data for x and y)
		04 2c	//2, 22
		06 42   //3, 33

As the repetitions of the data of a given type increases, so do the space savings.

Array typedefs take the item type as a parameter, then the length
is an argument to the actual occurrence of the new tag.

Struct typedefs define their fields an an ordered set of named fields, each with a type. If the type is an array or map,
then the appropriate item type/key type is included in the same definition. The ANY_TAG may be used if one or the other
of the key or value type is not to be constrained.

A type definition must be closed for this encoding to be used. Non-closed types do not get optimized.

### Version Tag

TBin is currently at version 1. TBin streams should always start with a version tag.

    VERSION_TAG | (version - 1)

	vN -> "0001 1NNN"   where N is (version - 1)
	v3 -> "0001 1000"
	v4 -> "0001 1001"

### Comparison of behavior with Protocol Buffers and Avro

Using the simple polyline type above, with the following data (shown in JSON, so you can read it) that includes some
multi-byte and negative numbers:

    {
        "points":[
            {"x":1,"y":11},
            {"x":2,"y":22},
            {"x":3,"y":33},
            {"x":10,"y":100},
            {"x":-23,"y":100},
            {"x":-23,"y":-33},
            {"x":10,"y":-33},
            {"x":103,"y":333},
            {"x":300,"y":1000},
            {"x":1234,"y":1234},
            {"x":12345678,"y":12321312},
            {"x":321321321,"y":33},
            {"x":1,"y":11}
        ]
    }


| encoding | size (in bytes) | notes |
|--------|---------|---------|
| JSON | 250 | no schema used or needed, but loses some type information. All whitespace removed. |
| TBin map | 160 | no schema used or needed, encoded from untyped data |
| TBin struct | 139 | no schema used or needed, encoded from untyped data |
| protobuf | 96 | schema required to both read and write (and it is 308 bytes) |
| TBin typedefs | 70 | schema used to encode, no schema needed to decode |
| Avro | 46 | schema required to both read and write (and it is 230 bytes) |


