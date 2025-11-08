---
layout: page
title: GitLite
description: A C++ clone of Git built from scratch to demystify its core content-addressed object model and data structures.
# img: assets/img/12.jpg
importance: 6
category: personal
related_publications: false
---

## Overview

I built GitLite to deepen my knowledge of Git's architecture. Git is everywhere in software development.

<div class="row justify-content-sm-center">
    <div class="col-sm-5 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/everywhere meme.jpg" title="everywhere meme" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">I know you can hear this image.</div>

But its internals—how it stores files, tracks changes, and manages history—often remain a black box *(yeah, again a black box analogy ;)*. By reimplementing a subset of its functionality, I aimed to gain hands-on experience with concepts like content-addressable storage, hashing, and tree-based data structures.

GitLite is a command-line tool that mimics Git's essential operations without the full complexity of the original. It handles object storage (blobs for files, trees for directories, commits for snapshots), hashing with SHA-1, compression via zlib, and basic repository management. Its purpose is primarily educational: it demonstrates how version control can be built from the ground up, making it a great teaching aid or starting point for custom VCS experiments. For developers, it's a reminder that powerful tools like Git are built on simple, elegant primitives.

## Research

Before writing a single line of code, I dove into Git's documentation and internals. I started with the official Git book and tutorials on "Git from the Bottom Up," which explained key concepts like objects, refs, and the object database. I learned that Git stores everything as objects identified by SHA-1 hashes: blobs for file contents, trees for directory structures (as lists of mode/path/hash entries), and commits as key-value metadata with pointers to trees and parents.

I researched SHA-1 hashing using OpenSSL for secure, efficient computation and zlib for compression/decompression of objects. File system interactions were crucial, so I studied C++'s `<filesystem>` library for path handling and recursion. I also examined how Git ignores files (via `.gitignore`) but opted for a simplified hardcoded approach initially.

To validate ideas, I experimented with real Git repositories: using commands like `git cat-file` and `git ls-tree` to inspect objects, and manually crafting small repos to understand serialization formats (e.g., tree objects as binary data with null terminators). This hands-on dissection helped me map Git's binary formats to C++ structures.

## Implementation

GitLite's core revolves around a `GitRepository` class that manages the `.git` directory, including paths for objects, refs, and HEAD. Objects are abstracted via a `GitObject` base class, with derived classes for `GitBlob` (file content), `GitTree` (directory listings as vectors of `GitTreeLeaf` structs with mode, path, and SHA), and `GitCommit` (key-value lists for metadata like tree, parent, author, and message).

Key functionalities include:

- **Object Serialization/Deserialization**: Blobs are raw data; trees serialize to "mode path\0binary_sha" sequences (with octal modes and hex-to-binary SHA conversion); commits use newline-separated key-values followed by the message. Parsing reverses this, handling positions for spaces, nulls, and binary data.

- **Hashing and Storage**: The `object_write` function serializes an object, prepends a header (e.g., "blob size\0"), computes SHA-1, compresses with zlib, and stores in `.git/objects/<prefix>/<rest>`. `object_read` decompresses and extracts data.

- **Commands**:
  - `init`: Creates `.git` subdirs, default config, and HEAD ref.
  - `hash-object`: Reads a file, creates a blob, and writes/returns its SHA.
  - `cat-file`: Finds an object by name (resolving refs/HEAD), verifies type, and outputs data.
  - `write-tree`: Recursively scans the worktree (skipping ignores), hashes files as blobs, subdirs as trees, sorts entries, and writes the root tree.
  - `commit-tree`: Builds a commit object with tree/parent/author/timestamp/message, then hashes it.
  - `ls-tree`: Parses a tree and lists entries in "mode path\tsha" format.
  - `log`: Traverses commit parents from HEAD or a given SHA, printing SHA, author, and message.
  - `checkout`: Parses a commit's tree, recursively restores files/dirs to the worktree, and updates HEAD (detached).

Error handling covers malformed data, type mismatches, and file ops. The main entry point parses commands and dispatches to these functions.

## Results

GitLite successfully implements a functional subset of Git, allowing users to initialize repos, hash files, snapshot directories, create commits, view history, and switch states. The `test.sh` script automates verification: it creates a temp repo, adds files, runs commands, and checks outputs (e.g., ls-tree matches expected file listings, log shows commit chains, checkout restores files). All tests pass, confirming reliability for basic workflows.

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/gitlite-test.png" title="gitlite-test" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">Compiling and testing GitLite with a script.</div>

In practice, it handles small projects well—e.g., committing changes to text files and reverting via checkout. Performance is efficient for its scale, thanks to zlib and filesystem recursion. While not production-ready (no remotes, branches, or merging), it accurately replicates Git's object model, proving the project's educational value.

## Reflection

GitLite uses C++17 for modern features like `<filesystem>`, CMake for builds, OpenSSL for SHA-1, and zlib for compression. This stack was ideal for low-level control and portability.

If I had more time, I'd add an index (staging area) for partial commits, branch support (via refs), merge functionality (three-way merges), and proper `.gitignore` parsing instead of hardcoded ignores. I'd also implement short SHA resolution, packfiles for efficiency, and more robust error messages. Overall, this project solidified my grasp of systems design—next, I might extend it to a distributed VCS or explore Rust for safer memory handling.

> View the GitLite source code and documentation on [GitHub](https://github.com/carb0ned0/GitLite).
