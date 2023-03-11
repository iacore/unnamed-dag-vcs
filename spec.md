## properties of the system

- each file is identified by its hash (multihash).
    - the hash can calculated solely from what is directly in a file.
    - there are multiple ways to hash a file. Please publish your file under the same hash as referenced by other files so it's reachable.
    - the writer/uploader decides which hash function to use (in multihash)
    - the reader/downloader does not need to worry about what hash type it is. It should be an opaque bytestring.
- each file can have multiple referecnes
- each file can have multiple chunk
- each chunk can have multiple attributes (metadata)
- each chunk consists of multiple segments, either owning or referenced

## trivia

- each chunk **can** be flattened to a sequence of bytes. That loses some infomation that is important in, e.g. directory listing.
- filename is not included in the file, but in a directory file
- the directory file can be treated as a normal file (diffed).
- the storage backend is tasked to keep all the reachable files alive.
- no compaction is provided, for simplicity of the implementation. You can compress the files together during transfer or storage though.

## Example 0

Example bag of files. `aaaaaa, bbbbbb, cccccc` are hashes (omitted).

```
file aaaaaa ( should be hash here )
chunk 0 mime=text/plain
    "some content here"

file bbbbbb
references
    0) aaaaaa
chunk 0 mime=text/plain
    [@0.0>0-10] (byte range from ref 0 chunk 0)
    "there"

file cccccc
chunk 0 mime=text/x-commonmark
    "this is project description"
chunk 1 mime=x-some-wiki/metadata
    "some meta data here"

file index
references
    0) bbbbbb
    1) cccccc
chunk 0 mime=x-unnamed-vcs/dir-listing
    "hello.txt"
    [@0>0-0]
    "readme.md"
    [@1>0-0]
```
