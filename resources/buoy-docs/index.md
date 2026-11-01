---
title: "File metadata"
summary: "Read file display names, visibility and tags, and explicitly replace tags."
category: "Overview"
order: 0
keywords: ["filemetadata", "macOS", "Linux", "Windows", "tags", "xattr", "native"]
related: ["getting-started.md", "api.md", "typed-api.md"]
---

# File metadata

Use `filemetadata` to display a file's name, visibility and tags, plus macOS package, application and backup-exclusion flags. Taking a snapshot only reads metadata and never reads file contents. Replacing tags is a separate, explicit operation.

## Platform availability

Snapshots work on macOS, Linux and Windows (`FileMetadataSnapshotSupported()`). Tags work on macOS (Finder tags) and Linux (the `user.xdg.tags` extended attribute); `FileTagsSupported()` is False on Windows, where `SetFileTags` throws `UnsupportedOperationException`. `FileMetadataSupported()` keeps its original meaning and is True only on macOS. Off macOS, `IsPackage`, `IsApplication` and `ExcludedFromBackup` are False sentinels, and `HasPackageStatus`, `HasApplicationStatus` and `HasBackupExclusion` report that. `HasTags` is False when `Tags` is a sentinel rather than an observed list.

## Package use

Install `filemetadata-0.1.0.buipkg` in the consumer project's `packages/` directory, then use `Import filemetadata`. This package is independently sealed; it does not require importing the other desktop integration packages.

## Behavior and lifetime

Invalid input throws `InvalidArgumentException`: a Nil or missing file, a blank tag or one containing NUL, and on Linux a comma or a serialized value over 65536 bytes. Tag lists are validated before any native call, so a rejected list leaves the stored tags unchanged. `SetFileTags` validates tag text before checking the platform, so invalid tags throw `InvalidArgumentException` on Windows too. Linux and Windows native failures throw `FileMetadataException`, a subclass of `Exception`, with `Operation`, `Domain` (`errno`, `Win32` or `buoy-filemetadata`) and `NativeCode`. The macOS backend keeps its original exceptions. A malformed Linux tag attribute (a NUL byte or invalid UTF-8) throws rather than decoding lossily. The package neither grants filesystem permissions nor starts security-scoped access. Snapshot properties are copied values and need no close operation; they are read in separate native calls and are not atomic against concurrent changes.
## Continue reading

- [Overview](index.md)
- [Getting started](getting-started.md)
- [API reference](api.md)
