+++
title = "ProtoJSON Format"
weight = 62
description = "Covers how to use the Protobuf to JSON conversion utilities."
type = "docs"
+++

Protobuf supports a canonical encoding in JSON, making it easier to share data
with systems that do not support the standard protobuf binary wire format.

ProtoJSON Format is not as efficient as protobuf wire format. The converter uses
more CPU to encode and decode messages and (except in rare cases) encoded
messages consume more space. Furthermore, ProtoJSON format puts your field and
enum value names into encoded messages making it much harder to change those
names later. Removing fields is a breaking change that will trigger a parsing
error. In short, there are many good reasons why Google prefers to use the
standard wire format for virtually everything rather than ProtoJSON format.

The encoding is described on a type-by-type basis in the table later in this
topic.

When parsing JSON-encoded data into a protocol buffer, if a value is missing or
if its value is `null`, it will be interpreted as the corresponding
[default value](/programming-guides/editions#default). Multiple values for
singular fields (using duplicate or equivalent JSON keys) are accepted and the
last value is retained, as with binary format parsing. Note that not all
protobuf JSON parser implementations are conformant, and some nonconformant
implementations may reject duplicate keys instead.

When generating JSON-encoded output from a protocol buffer, if a protobuf field
has the default value and if the field doesn't support field presence, it will
be omitted from the output by default. An implementation may provide options to
include fields with default values in the output.

Fields that have a value set and that support field presence always include the
field value in the JSON-encoded output, even if it is the default value. For
example, a proto3 field that is defined with the `optional` keyword supports
field presence and if set, will always appear in the JSON output. A message type
field in any edition of protobuf supports field presence and if set will appear
in the output. Proto3 implicit-presence scalar fields will only appear in the
JSON output if they are not set to the default value for that type.

When representing numerical data in a JSON file, if the number that is is parsed
from the wire doesn't fit in the corresponding type, you will get the same
effect as if you had cast the number to that type in C++ (for example, if a
64-bit number is read as an int32, it will be truncated to 32 bits).

The following table shows how data is represented in JSON files.

<table>
  <tbody>
    <tr>
      <th>Protobuf</th>
      <th>JSON</th>
      <th>JSON example</th>
      <th>Notes</th>
    </tr>
    <tr>
      <td>message</td>
      <td>object</td>
      <td><code>{"fooBar": v, "g": null, ...}</code></td>
      <td>Generates JSON objects. Message field names are mapped to
        lowerCamelCase and become JSON object keys. If the
        <code>json_name</code> field option is specified, the specified value
        will be used as the key instead. Parsers accept both the lowerCamelCase
        name (or the one specified by the <code>json_name</code> option) and the
        original proto field name. <code>null</code> is an accepted value for
        all field types and treated as the default value of the corresponding
        field type. However, <code>null</code> cannot be used for the
        <code>json_name</code> value. For more on why, see
        <a href="/news/2023-04-28#json-name">Stricter validation for json_name</a>.
      </td>
    </tr>
    <tr>
      <td>enum</td>
      <td>string</td>
      <td><code>"FOO_BAR"</code></td>
      <td>The name of the enum value as specified in proto is used. Parsers
        accept both enum names and integer values.
      </td>
    </tr>
    <tr>
      <td>map&lt;K,V&gt;</td>
      <td>object</td>
      <td><code>{"k": v, ...}</code></td>
      <td>All keys are converted to strings.</td>
    </tr>
    <tr>
      <td>repeated V</td>
      <td>array</td>
      <td><code>[v, ...]</code></td>
      <td><code>null</code> is accepted as the empty list <code>[]</code>.</td>
    </tr>
    <tr>
      <td>bool</td>
      <td>true, false</td>
      <td><code>true, false</code></td>
      <td></td>
    </tr>
    <tr>
      <td>string</td>
      <td>string</td>
      <td><code>"Hello World!"</code></td>
      <td></td>
    </tr>
    <tr>
      <td>bytes</td>
      <td>base64 string</td>
      <td><code>"YWJjMTIzIT8kKiYoKSctPUB+"</code></td>
      <td>JSON value will be the data encoded as a string using standard base64
        encoding with paddings. Either standard or URL-safe base64 encoding
        with/without paddings are accepted.
      </td>
    </tr>
    <tr>
      <td>int32, fixed32, uint32</td>
      <td>number</td>
      <td><code>1, -10, 0</code></td>
      <td>JSON value will be a decimal number. Either numbers or strings are
        accepted. Empty strings are invalid. Exponent notation (such as `1e2`) is
        accepted in both quoted and unquoted forms.
      </td>
    </tr>
    <tr>
      <td>int64, fixed64, uint64</td>
      <td>string</td>
      <td><code>"1", "-10"</code></td>
      <td>JSON value will be a decimal string. Either numbers or strings are
        accepted. Empty strings are invalid. Exponent notation (such as `1e2`) is
        accepted in both quoted and unquoted forms.
      </td>
    </tr>
    <tr>
      <td>float, double</td>
      <td>number</td>
      <td><code>1.1, -10.0, 0, "NaN", "Infinity"</code></td>
      <td>JSON value will be a number or one of the special string values "NaN",
        "Infinity", and "-Infinity". Either numbers or strings are accepted.
        Empty strings are invalid. Exponent notation is also accepted.
      </td>
    </tr>
    <tr>
      <td>Any</td>
      <td><code>object</code></td>
      <td><code>{"@type": "url", "f": v, ... }</code></td>
      <td>If the <code>Any</code> contains a value that has a special JSON
        mapping, it will be converted as follows: <code>{"@type": xxx, "value":
        yyy}</code>. Otherwise, the value will be converted into a JSON object,
        and the <code>"@type"</code> field will be inserted to indicate the
        actual data type.
      </td>
    </tr>
    <tr>
        <td>Timestamp</td>
        <td>string</td>
        <td><code>"1972-01-01T10:00:20.021Z"</code></td>
        <td>Uses RFC 3339 (see <a href="#rfc3339">clarification</a>), where generated output will always be Z-normalized
          and uses 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are
          also accepted.
      </td>
    </tr>
    <tr>
      <td>Duration</td>
      <td>string</td>
      <td><code>"1.000340012s", "1s"</code></td>
      <td>Generated output always contains 0, 3, 6, or 9 fractional digits,
        depending on required precision, followed by the suffix "s". Accepted
        are any fractional digits (also none) as long as they fit into
        nano-seconds precision and the suffix "s" is required.
      </td>
    </tr>
    <tr>
      <td>Struct</td>
      <td><code>object</code></td>
      <td><code>{ ... }</code></td>
      <td>Any JSON object. See <code>struct.proto</code>.</td>
    </tr>
    <tr>
      <td>Wrapper types</td>
      <td>various types</td>
      <td><code>2, "2", "foo", true, "true", null, 0, ...</code></td>
      <td>Wrappers use the same representation in JSON as the wrapped primitive
        type, except that <code>null</code> is allowed and preserved during data
        conversion and transfer.
      </td>
    </tr>
    <tr>
      <td>FieldMask</td>
      <td>string</td>
      <td><code>"f.fooBar,h"</code></td>
      <td>See <code>field_mask.proto</code>.</td>
    </tr>
    <tr>
      <td>ListValue</td>
      <td>array</td>
      <td><code>[foo, bar, ...]</code></td>
      <td></td>
    </tr>
    <tr>
      <td>Value</td>
      <td>value</td>
      <td></td>
      <td>Any JSON value. Check
        <a href="/reference/protobuf/google.protobuf#value">google.protobuf.Value</a>
        for details.
      </td>
    </tr>
    <tr>
      <td>NullValue</td>
      <td>null</td>
      <td></td>
      <td>JSON null</td>
    </tr>
    <tr>
      <td>Empty</td>
      <td>object</td>
      <td><code>{}</code></td>
      <td>An empty JSON object</td>
    </tr>
  </tbody>
</table>

## ProtoJSON Wire Safety {#json-wire-safety}

When using ProtoJSON, only some schema changes are safe to make in a distributed
system. This contrasts with the same concepts applied to the
[the binary wire format](/programming-guides/editions#updating).

### JSON Wire-unsafe Changes {#wire-unsafe}

Wire-unsafe changes are schema changes that will break if you parse data that
was serialized using the old schema with a parser that is using the new schema
(or vice versa). You should almost never do this shape of schema change.

*   Changing a field to or from an extension of same number and type is not
    safe.
*   Changing a field between `string` and `bytes` is not safe.
*   Changing a field between a message type and `bytes` is not safe.
*   Changing any field from `optional` to `repeated` is not safe.
*   Changing a field between a `map<K, V>` and the corresponding `repeated`
    message field is not safe.
*   Moving fields into an existing `oneof` is not safe.

### JSON Wire-safe Changes {#wire-safe}

Wire-safe changes are ones where it is fully safe to evolve the schema in this
way without risk of data loss or new parse failures.

Note that nearly all wire-safe changes may be a breaking change to application
code. For example, adding a value to a preexisting enum would be a compilation
break for any code with an exhaustive switch on that enum. For that reason,
Google may avoid making some of these types of changes on public messages. The
AIPs contain guidance for which of these changes are safe to make there.

*   Changing a single `optional` field into a member of a **new** `oneof` is
    safe.
*   Changing a `oneof` which contains only one field to an `optional` field is
    safe.
*   Changing a field between any of `int32`, `sint32`, `sfixed32`, `fixed32` is
    safe.
*   Changing a field between any of `int64`, `sint64`, `sfixed64`, `fixed64` is
    safe.
*   Changing a field number is safe (as the field numbers are not used in the
    ProtoJSON format), but still strongly discouraged since it is very unsafe in
    the binary wire format.
*   Adding values to an enum is safe if the "Emit enum values as integers" is
    set on all relevant clients (see [options](#json-options))

### JSON Wire-compatible Changes (Conditionally safe) {#conditionally-safe}

Unlike wire-safe changes, wire-compatible means that the same data can be parsed
both before and after a given change. However, a client that reads it will get
lossy data under this shape of change. For example, changing an int32 to an
int64 is a compatible change, but if a value larger than INT32_MAX is written, a
client that reads it as an int32 will discard the high order bits.

You can make compatible changes to your schema only if you manage the roll out
to your system carefully. For example, you may change an int32 to an int64 but
ensure you continue to only write legal int32 values until the new schema is
deployed to all endpoints, and then start writing larger values after that.

#### Compatible But With Unknown Field Handling Problems {#compatible-ish}

Unlike the binary wire format, ProtoJSON implementations generally do not
propagate unknown fields. This means that adding to schemas is generally
compatible but will result in parse failures if a client using the old schema
observes the new content.

This means you can add to your schema, but you cannot safely start writing them
until you know the schema has been deployed to the relevant client or server (or
that the relevant clients set an Ignore Unknown Fields flag, discussed
[below](#json-options)).

*   Adding and removing fields is considered compatible with this caveat.
*   Removing enum values is considered compatible with this caveat.

#### Compatible But Potentially Lossy {#compatible-lossy}

*   Changing between any of the 32-bit integers (`int32`, `uint32`, `sint32`,
    `sfixed32`, `fixed32`) and any of the 64-bit integers ( `int64`, `uint64`,
    `sint64`, `sfixed32`) is a compatible change.
    *   If a number is parsed from the wire that doesn't fit in the
        corresponding type, you will get the same effect as if you had cast the
        number to that type in C++ (for example, if a 64-bit number is read as
        an int32, it will be truncated to 32 bits).
    *   Unlike binary wire format, `bool` is not compatible with integers.
    *   Note that the int64 types are quoted by default to avoid precision loss
        when handled as a double or JavaScript number, and the 32 bit types are
        unquoted by default. Conformant implementations will accept either case
        for all integer types, but nonconformant implementations may mishandle
        this case and not handle quoted int32s or unquoted int64s which may
        break under this change.
*   `enum` may be conditionally compatible with `string`
    *   If "enums-as-ints" flag is used by any client, then enums will instead
        be compatible with the integer types instead.

## RFC 3339 Clarification {#rfc3339}

[RFC 3339](https://www.rfc-editor.org/rfc/rfc3339) intends to declare a strict
subset of ISO-8601 format, and unfortunately some ambiguity was created since
RFC 3339 was published in 2002 and then ISO-8601 was subsequently revised
without any corresponding revisions of RFC 3339.

Most notably, ISO-8601-1988 contains this note:

> In date and time representations lower case characters may be used when upper
> case characters are not available.

It is ambiguous whether this note is suggesting that parsers should accept
lowercase letters in general, or if it is only suggesting that lowercase letters
may be used as a substitute in environments where uppercase cannot be
technically used. RFC 3339 contains a note that intends to clarify the
interpretation to be that lowercase letters should be accepted in general.

ISO-8601-2019 does not contain the corresponding note and is unambiguous that
lowercase letters are not allowed. This created some confusion for all libraries
that declare they support RFC 3339: today RFC 3339 declares it is a profile of
ISO-8601 but contains a note that is in reference to something that is no longer
in the latest ISO-8601 spec.

ProtoJSON spec takes the decision that the timestamp format is the stricter
definition of "RFC 3339 as a profile of ISO-8601-2019". Some Protobuf
implementations may be non-conformant by using a timestamp parsing
implementation that is implemented as "RFC 3339 as a profile of ISO-8601-1988,"
which will accept a few additional edge cases.

For consistent interoperability, parsers should only accept the stricter subset
format where possible. When using a non-conformant implementation that accepts
the laxer definition, strongly avoid relying on the additional edge cases being
accepted.

## JSON Options {#json-options}

A conformant protobuf JSON implementation may provide the following options:

*   **Always emit fields without presence**: Fields that don't support presence
    and that have their default value are omitted by default in JSON output (for
    example, an implicit presence integer with a 0 value, implicit presence
    string fields that are empty strings, and empty repeated and map fields). An
    implementation may provide an option to override this behavior and output
    fields with their default values.

    As of v25.x, the C++, Java, and Python implementations are nonconformant, as
    this flag affects proto2 `optional` fields but not proto3 `optional` fields.
    A fix is planned for a future release.

*   **Ignore unknown fields**: The protobuf JSON parser should reject unknown
    fields by default but may provide an option to ignore unknown fields in
    parsing.

*   **Use proto field name instead of lowerCamelCase name**: By default the
    protobuf JSON printer should convert the field name to lowerCamelCase and
    use that as the JSON name. An implementation may provide an option to use
    proto field name as the JSON name instead. Protobuf JSON parsers are
    required to accept both the converted lowerCamelCase name and the proto
    field name.

*   **Emit enum values as integers instead of strings**: The name of an enum
    value is used by default in JSON output. An option may be provided to use
    the numeric value of the enum value instead.
