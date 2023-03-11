## properties of the system

- each file is identified by its hash (multihash).
    - the hash can calculated solely from what is directly in a file.
    - there are multiple ways to hash a file. Please publish your file under the same hash as referenced by other files so it's reachable.
    - the writer/uploader decides which hash function to use (in multihash)
    - the reader/downloader does not need to worry about what hash type it is. It should be an opaque bytestring.
- each file can have multiple referecnes
- each file can have multiple chunk
- each chunk can have multiple data segments, each either owned (in the file) or referenced (in another file)
- each chunk can have multiple control segments (metadata like MIME), all owned

### key-value storage

files are stored in a key-value storage by their hash. The storage layer is not to understand the semantic of the hash.

The storage layer is expected to understand references of a file, and keep all reachable files alive. It is for the operator to decide the exact retention policy.

### binary encoding

The exact binary encoding of each file is not restricted, as long as they all convey the same exact semantic, nothing added or taken away.

A reference binary encoding and key-value storage is included is this repository.

## trivia

- each chunk can be flattened to a sequence of bytes. However, that is not the only way to understand a file. Directory listings, for example, use referenced segment to represent files in a directory.
- filename is not included in the file, but in a directory file
- the directory file can be treated as a normal file (diffed).
- the storage backend is tasked to keep all the reachable files alive.
- no compaction is provided, for simplicity of the implementation. You can compress the files together during transfer or storage though.

## Example

Example bag of files. `aaaaaa, bbbbbb, cccccc` are hashes (omitted).

`aaaaaa` is version 0 of `hello.txt`
`bbbbbb` is version 1 of `hello.txt`
`cccccc` is `readme.md`
`dddddd` is the project index (like a git commit snapshot)

```
file aaaaaa
chunk 0 "mime=text/plain"
    "some content here"

file bbbbbb
references
    0) aaaaaa
chunk 0 "mime=text/plain"
    [@0.0>0-10] (byte range from ref 0 chunk 0)
    "there"

file cccccc
chunk 0 "mime=text/x-commonmark"
    "pretend this is project description"
chunk 1 "mime=x-some-wiki/metadata"
    "some meta data here"

file dddddd
references
    0) bbbbbb
    1) cccccc
chunk 0 "mime=x-unnamed-vcs/dir-listing"
    "hello.txt"
    [@0>0-0]
    "readme.md"
    [@1>0-0]
chunk 0 "mime=x-unnamed-vcs/commit-message"
    "first commit"
```
