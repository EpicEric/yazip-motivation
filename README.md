# YAZip Motivation

__WORK IN PROGRESS - Pull requests are welcome!__

Here we explain what is YAZip and why it is an important compression mechanism for structured data.

## What?

__YAZip__ (Yet Another Zip) is a structured data minifier. It can turn any JSON or YAML document into much smaller text. And the best part: it is also valid YAML!

## How?

YAML 1.2 has some features that can (but probably shouldn't...) be used for compressing structured data:

- Strings do not require quotes, except for empty strings.
  - But we can use empty strings instead of null!
- Booleans can be shortened (`true` -> `yes`, `false` -> `no`).
- Top-level objects do not require braces; by using new-lines instead of commas we can save exactly 2 bytes. Example:
```yaml
# This:
{url: github.com,name: GitHub,mascot: Octocat}
---
# ...is equivalent to this:
url: github.com
name: GitHub
mascot: Octocat
# Wow!
```
- And most importantly, we can use an unhealthy amount of [anchors and aliases](https://yaml.org/spec/1.2/spec.html#id2760395) to duplicate strings and similar objects!

## Why?

There are actually a handful of reasons:

- A YAML library conformant with spec 1.2 can read any valid JSON, YAML or YAZip, which means clients parsing API data as YAML do not need to be updated if the server decides to migrate from one format to another.
- YAZip automatically eliminates redundancy from both keys and values, which is a common occurrence in a single API call that returns multiple objects.
- Like YAML, YAZip lets you include several documents in a single file, which means we can even have interdocument compression. Some file-specific mechanisms can be used if working on a filesystem.
- Unlike Protobuf, YAZip can be read directly by a human person (albeit with some difficulty).
- Protobuf uses a message descriptor, which is shared between the server and the client, to properly encode data; YAZip doesn't.
- Protobuf requires a specific library and code to encode/decode, while YAZip can be read by any YAML 1.2 library, which is implemented in most of the popular programming languages' standard library.

## Examples

Disclaimer: For handling JSON, I've used [JSONCompare](https://jsoncompare.com/). For handling Protobuf, I've used [JSON to Protobuf Creator](https://www.site24x7.com/tools/json-to-protobuf.html) and [depene/js-protobuf-encode-decode](https://github.com/depene/js-protobuf-encode-decode).

### Minimal structured data

- JSON (prettified, 17 bytes):

```json
{
	"foo": "bar"
}
```

- JSON (minified, 13 bytes):

```json
{"foo":"bar"}
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

__Result__: YAZip final size is 61.5% of minified JSON, and 133.3% of Protobuf.

### List of similar objects

- Prettified JSON (201 bytes):

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

Anchors within anchors let us replicate the same object or strings over and over when only a few fields actually change.

__Result__: YAZip final size is 66.9% of minified JSON, and 125.6% of Protobuf.

## Implementation

_//TODO: Create an implementation in Python or Javascript._

## How can I help?

- By reviewing the YAZip specification. _//TODO: Create a YAZip specification_
- By improving the official implementations of the YAZip encoder. _//TODO: Create a YAZip implementation_
- By creating new implementations of the YAZip encoder in other languages.
- By updating open-source YAML parsers that do not conform to YAML 1.2 yet.
- By implementing the YAZip encoder on your favorite open-source projects.
