# YAZip Motivation

__WORK IN PROGRESS - Pull requests are welcome!__

Here we explain what is YAZip and why it is an important compression mechanism for structured data.

## What?

__YAZip__ (YAML As Zip, or Yet Another Zip format) is a serialized data minifier. It can turn any JSON or YAML document into much smaller text. And the best part: it is also valid YAML!

## How?

YAML 1.2 has some features that can (but probably shouldn't...) be used for compressing serialized data:

- Strings do not require quotes, except for empty strings, or strings containing escaped sequences or special characters.
- We can save a few bytes with empty values instead of null.
- Booleans can be shortened (`true` -> `yes`, `false` -> `no`).
- We can use curly braces and square brackets to declare YAML flow mappings and flow sequences, respectively, much like you would for JSON objects and lists.
- Top-level objects do not require braces; by using new-lines instead of commas we can save exactly 2 bytes. Example:

```yaml
# This:
{url: github.com,name: GitHub,mascot: Octocat}
---
# ...is equivalent to this:
url: github.com
name: GitHub
mascot: Octocat
```
- And most importantly, we can use an unhealthy amount of [anchors and aliases](https://yaml.org/spec/1.2/spec.html#id2760395) to duplicate strings and similar objects!

## Why?

There are actually a handful of reasons:

- A YAML library conformant with spec 1.2 can read any valid JSON, YAML or YAZip alike.
- YAZip automatically eliminates redundancy from both keys and values, which is a common occurrence in a single API call that returns multiple objects in a list.
- Like YAML, YAZip lets you include several documents in a single file. Unfortunately, anchors are not shared between documents, which means no interdocument compression (without some other protocol for using a single document instead).
- Protobuf uses a message descriptor, which is shared between the server and the client, to properly encode data; while YAZip is schemaless.
- MessagePack and Protobuf require a specific library and code to decode, while YAZip can be read by any YAML 1.2 library, which is already implemented as a library in basically all popular programming languages.
- Unlike MessagePack and Protobuf, YAZip can be read directly by a human person (albeit with some difficulty).

These features make YAZip an interesting choice for:

- APIs that feed a lot of structured data to clients, are already using JSON/YAML, and/or plan to migrate from one of these formats to YAZip and back, since YAML 1.2 can read all three formats alike.
- Compressing multiple files with structured data in a filesystem, when using file-specific mechanisms. _// TODO: Start discussing strategies on this_
- An alternative to JWT (or other JSON-centric projects) that wants to use YAZip instead of JSON to decrease document sizes.

## Examples

Disclaimer: For handling JSON, I've used [JSONCompare](https://jsoncompare.com/). For handling Protobuf, I've used [JSON to Protobuf Creator](https://www.site24x7.com/tools/json-to-protobuf.html) and [depene/js-protobuf-encode-decode](https://github.com/depene/js-protobuf-encode-decode). For YAML validation, I've used [JSON to YAML](https://www.json2yaml.com/).

### Minimal structured data

- Prettified JSON:

```json
{
	"foo": "bar"
}
```

- Minified JSON (13 bytes):

```json
{"foo":"bar"}
```

- MessagePack (9 bytes):

```txt
<81 A3> foo <A3> bar
```

- Protobuf (5 bytes):

```txt
message Example {
  required string foo = 1;
}

<0A 03 62 61 72>
```

- YAZip / Standard YAML (8 bytes):

```yaml
foo: bar
```

__Result__: YAZip's final size is 61.5% of the minified JSON's, 88.9% of MessagePack's, and 133.3% of Protobuf's.

### List of similar objects

- Prettified JSON:

```json
[{
	"origin": "github",
	"user": "epiceric",
	"repo": "basicc"
}, {
	"origin": "github",
	"user": "berna-l",
	"repo": "sts-docs"
}, {
	"origin": "github",
	"user": "epiceric",
	"repo": "mtmg.com.br"
}]
```

- Minified JSON (169 bytes):

```json
[{"origin":"github","user":"epiceric","repo":"basicc"},{"origin":"github","user":"berna-l","repo":"sts-docs"},{"origin":"github","user":"epiceric","repo":"mtmg.com.br"}]
```

- MessagePack (130 bytes):

```txt
<90 83 A6> origin <A6> github <A4> user <A8> epiceric <A4> repo <A6> basicc <83 A6> origin <A6> github <A4> user <A7> berna-l <A4> repo <A8> sts-docs <83 A6> origin <A6> github <A4> user <A8> epiceric <A4> repo <AB> mtmg.com.br
```

- Protobuf (90 bytes) -- note that we _must_ enclose our array with an object:

```txt
message Example {
  message DATA {
    required string origin = 0;
    required string user = 1;
    required string repo = 2;
  }
  repeated DATA data = 0;
}

<02 1A 02 06 67 69 74 68 75 62 0A 08 65 70 69 63 65 72 69 63 12 06 62 61 73 69 63 63 02 1B 02 06 67 69 74 68 75 62 0A 07 62 65 72 6E 61 2D 6C 12 08 73 74 73 2D 64 6F 63 73 02 1F 02 06 67 69 74 68 75 62 0A 08 65 70 69 63 65 72 69 63 12 0B 6D 74 6D 67 2E 63 6F 6D 2E 62 72>
```

- Standard YAML (152 bytes):

```yaml
- origin: github
  user: epiceric
  repo: basicc
- origin: github
  user: berna-l
  repo: sts-docs
- origin: github
  user: epiceric
  repo: mtmg.com.br
```

- YAZip (113 bytes):

```yaml
[&D {origin: github,user: epiceric,&E repo: basicc},{<<: *D,user: berna-l,*E: sts-docs},{<<: *D,*E: mtmg.com.br}]
```

Anchors within anchors let us replicate the same object or strings over and over when only a few fields actually change. The better the common denominator (i.e. the object with the closest fields to all other objects), the better the compression.

__Result__: YAZip's final size is 66.9% of the minified JSON's, 86.9% of MessagePack's, and 125.6% of Protobuf's.

## Implementation

_//TODO: Create an official implementation_

## How can I help?

- By reviewing the YAZip specification. _//TODO: Create a YAZip specification_
- By improving the official implementations of the YAZip encoder. _//TODO: Create an official implementation_
- By creating new implementations of the YAZip encoder in other languages.
- By updating open-source YAML parser libraries that do not conform to YAML 1.2 yet.
- By implementing the YAZip encoder on your favorite open-source projects.
