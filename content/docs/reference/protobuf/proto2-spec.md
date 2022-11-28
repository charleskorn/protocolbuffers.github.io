---
title: "Protocol Buffers Version 2 Language Specification"
weight: 800
toc_hide: false
linkTitle: "Version 2 Language Specification"
no_list: "true"
type: docs
---


This is a language specification reference for version 2 of the Protocol Buffers
language (proto2). The syntax is specified using
[Extended Backus-Naur Form (EBNF)](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_Form):

```
|   alternation
()  grouping
[]  option (zero or one time)
{}  repetition (any number of times)
```

For more information about using proto2, see the
[language guide](/docs/programming-guides/proto).

## Lexical Elements {#lexical_elements}

### Letters and Digits {#letters_and_digits}

```
letter = "A" … "Z" | "a" … "z"
capitalLetter =  "A" … "Z"
decimalDigit = "0" … "9"
octalDigit   = "0" … "7"
hexDigit     = "0" … "9" | "A" … "F" | "a" … "f"
```

### Identifiers

```
ident = letter { letter | decimalDigit | "_" }
fullIdent = ident { "." ident }
messageName = ident
enumName = ident
fieldName = ident
oneofName = ident
mapName = ident
serviceName = ident
rpcName = ident
streamName = ident
messageType = [ "." ] { ident "." } messageName
enumType = [ "." ] { ident "." } enumName
groupName = capitalLetter { letter | decimalDigit | "_" }
```

### Integer Literals {#integer_literals}

```
intLit     = decimalLit | octalLit | hexLit
decimalLit = ( "1" … "9" ) { decimalDigit }
octalLit   = "0" { octalDigit }
hexLit     = "0" ( "x" | "X" ) hexDigit { hexDigit }
```

### Floating-point Literals

```
floatLit = ( decimals "." [ decimals ] [ exponent ] | decimals exponent | "."decimals [ exponent ] ) | "inf" | "nan"
decimals  = decimalDigit { decimalDigit }
exponent  = ( "e" | "E" ) [ "+" | "-" ] decimals
```

### Boolean

```
boolLit = "true" | "false"
```

### String Literals {#string_literals}

```
strLit = ( "'" { charValue } "'" ) | ( '"' { charValue } '"' )
charValue = hexEscape | octEscape | charEscape | /[^\0\n\\]/
hexEscape = '\' ( "x" | "X" ) hexDigit hexDigit
octEscape = '\' octalDigit octalDigit octalDigit
charEscape = '\' ( "a" | "b" | "f" | "n" | "r" | "t" | "v" | '\' | "'" | '"' )
```

### EmptyStatement

```
emptyStatement = ";"
```

### Constant

```
constant = fullIdent | ( [ "-" | "+" ] intLit ) | ( [ "-" | "+" ] floatLit ) |
                strLit | boolLit
```

## Syntax

The syntax statement is used to define the protobuf version.

```
syntax = "syntax" "=" ("'" "proto2" "'" | '"' "proto2" '"') ";"
```

## Import Statement {#import_statement}

The import statement is used to import another .proto's definitions.

```
import = "import" [ "weak" | "public" ] strLit ";"
```

Example:

```proto
import public "other.proto";
```

## Package

The package specifier can be used to prevent name clashes between protocol
message types.

```
package = "package" fullIdent ";"
```

Example:

```proto
package foo.bar;
```

## Option

Options can be used in proto files, messages, enums and services. An option can
be a protobuf defined option or a custom option. For more information, see
[Options](/docs/programming-guides/proto#options) in the language guide.

```
option = "option" optionName  "=" constant ";"
optionName = ( ident | "(" fullIdent ")" ) { "." ident }
```

For examples:

```proto
option java_package = "com.example.foo";
```

## Fields

Fields are the basic elements of a protocol buffer message. Fields can be normal
fields, group fields, oneof fields, or map fields. A field has a label, type and
field number.

```
label = "required" | "optional" | "repeated"
type = "double" | "float" | "int32" | "int64" | "uint32" | "uint64"
      | "sint32" | "sint64" | "fixed32" | "fixed64" | "sfixed32" | "sfixed64"
      | "bool" | "string" | "bytes" | messageType | enumType
fieldNumber = intLit;
```

### Normal field {#normal_field}

Each field has label, type, name and field number. It may have field options.

```
field = label type fieldName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
fieldOptions = fieldOption { ","  fieldOption }
fieldOption = optionName "=" constant
```

Examples:

```proto
optional foo.bar nested_message = 2;
repeated int32 samples = 4 [packed=true];
```

### Group field {#group_field}

**Note that this feature is deprecated and should not be used when creating new
message types -- use nested message types instead.**

Groups are one way to nest information in message definitions. The group name
must begin with capital letter.

```
group = label "group" groupName "=" fieldNumber messageBody
```

Example:

```proto
repeated group Result = 1 {
    required string url = 2;
    optional string title = 3;
    repeated string snippets = 4;
}
```

### Oneof and oneof field {#oneof_and_oneof_field}

A oneof consists of oneof fields and a oneof name. Oneof fields do not have
labels.

```
oneof = "oneof" oneofName "{" { option | oneofField | emptyStatement } "}"
oneofField = type fieldName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
```

Example:

```proto
oneof foo {
    string name = 4;
    SubMessage sub_message = 9;
}
```

### Map field {#map_field}

A map field has a key type, value type, name, and field number. The key type can
be any integral or string type. Note, the key type may not be an enum.

```
mapField = "map" "<" keyType "," type ">" mapName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
keyType = "int32" | "int64" | "uint32" | "uint64" | "sint32" | "sint64" |
          "fixed32" | "fixed64" | "sfixed32" | "sfixed64" | "bool" | "string"
```

Example:

```proto
map<string, Project> projects = 3;
```

## Extensions and Reserved {#extensions_and_reserved}

Extensions and reserved are message elements that declare a range of field
numbers or field names.

### Extensions

Extensions declare that a range of field numbers in a message are available for
third-party extensions. Other people can declare new fields for your message
type with those numeric tags in their own .proto files without having to edit
the original file.

```
extensions = "extensions" ranges ";"
ranges = range { "," range }
range =  intLit [ "to" ( intLit | "max" ) ]
```

Examples:

```proto
extensions 100 to 199;
extensions 4, 20 to max;
```

### Reserved

Reserved declares a range of field numbers or field names in a message that can
not be used.

```
reserved = "reserved" ( ranges | strFieldNames ) ";"
strFieldNames = strFieldName { "," strFieldName }
strFieldName = "'" fieldName "'" | '"' fieldName '"'
```

Examples:

```proto
reserved 2, 15, 9 to 11;
reserved "foo", "bar";
```

## Top Level definitions {#top_level_definitions}

### Enum definition {#enum_definition}

The enum definition consists of a name and an enum body. The enum body can have
options and enum fields.

```
enum = "enum" enumName enumBody
enumBody = "{" { option | enumField | emptyStatement } "}"
enumField = ident "=" [ "-" ] intLit [ "[" enumValueOption { ","  enumValueOption } "]" ]";"
enumValueOption = optionName "=" constant
```

Example:

```proto
enum EnumAllowingAlias {
  option allow_alias = true;
  EAA_UNSPECIFIED = 0;
  EAA_STARTED = 1;
  EAA_RUNNING = 2 [(custom_option) = "hello world"];
}
```

### Message definition {#message_definition}

A message consists of a message name and a message body. The message body can
have fields, nested enum definitions, nested message definitions, extend
statements, extensions, groups, options, oneofs, map fields, and reserved
statements.

```
message = "message" messageName messageBody
messageBody = "{" { field | enum | message | extend | extensions | group |
option | oneof | mapField | reserved | emptyStatement } "}"
```

Example:

```proto
message Outer {
  option (my_option).a = true;
  message Inner {   // Level 2
    required int64 ival = 1;
  }
  map<int32, string> my_map = 2;
  extensions 20 to 30;
}
```

### Extend

If a message in the same or imported .proto file has reserved a range for
extensions, the message can be extended.

```
extend = "extend" messageType "{" {field | group | emptyStatement} "}"
```

Example:

```proto
extend Foo {
  optional int32 bar = 126;
}
```

### Service definition {#service_definition}

```
service = "service" serviceName "{" { option | rpc | stream | emptyStatement } "}"
rpc = "rpc" rpcName "(" [ "stream" ] messageType ")" "returns" "(" [ "stream" ]
messageType ")" (( "{" { option | emptyStatement } "}" ) | ";" )
stream = "stream" streamName "(" messageType "," messageType ")" (( "{"
{ option | emptyStatement } "}") | ";" )
```

Example:

```proto
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

## Proto file {#proto_file}

```
proto = syntax { import | package | option | topLevelDef | emptyStatement }
topLevelDef = message | enum | extend | service
```

An example .proto file:

```proto
syntax = "proto2";
import public "other.proto";
option java_package = "com.example.foo";
enum EnumAllowingAlias {
  option allow_alias = true;
  EAA_UNSPECIFIED = 0;
  EAA_STARTED = 1;
  EAA_RUNNING = 1;
  EAA_FINISHED = 2 [(custom_option) = "hello world"];
}
message Outer {
  option (my_option).a = true;
  message Inner {   // Level 2
    required int64 ival = 1;
  }
  repeated Inner inner_message = 2;
  optional EnumAllowingAlias enum_field = 3;
  map<int32, string> my_map = 4;
  extensions 20 to 30;
}
message Foo {
  optional group GroupMessage = 1 {
    optional bool a = 1;
  }
}
```