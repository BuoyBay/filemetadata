---
title: "File metadata API"
summary: "Public filemetadata types and operations with behavior and lifetime notes."
category: "Reference"
order: 20
keywords: ["filemetadata", "API", "reference"]
symbols: ["FileMetadataSupported", "FileMetadataSnapshotSupported", "FileTagsSupported", "FileMetadataException", "FileMetadataSnapshot", "FileMetadataSnapshot.HasTags", "FileMetadataSnapshot.HasPackageStatus", "FileMetadataSnapshot.HasApplicationStatus", "FileMetadataSnapshot.HasBackupExclusion", "FileMetadataSnapshot.Tags", "FileMetadataSnapshot.DisplayName", "FileMetadataSnapshot.IsHidden", "FileMetadataSnapshot.IsPackage", "FileMetadataSnapshot.IsApplication", "FileMetadataSnapshot.ExcludedFromBackup", "SetFileTags"]
related: ["index.md", "getting-started.md", "typed-api.md"]
---

# API reference

`Import filemetadata` exposes the following public API. Properties described as read-only have no public setter.

| Symbol | Behavior |
| --- | --- |
| `FileMetadataSupported() As Boolean` | Frozen: True only where the original macOS backend runs. Compile-time; read-only. |
| `FileMetadataSnapshotSupported() As Boolean` | True on macOS, Linux and Windows. Compile-time; read-only. |
| `FileTagsSupported() As Boolean` | True on macOS and Linux; False on Windows. Compile-time; read-only. |
| `FileMetadataSnapshot(file As FileSystemItem)` | Reads metadata without changing the file or reading its contents. |
| `FileMetadataSnapshot.DisplayName As String` | Read-only. macOS: localized name. Linux: final path component (`.`/`..` resolved). Windows: on-disk entry name; roots report the full path. |
| `FileMetadataSnapshot.Tags As Array Of String` | Read-only copy of the captured tags. Windows: always empty. |
| `FileMetadataSnapshot.HasTags As Boolean` | True when `Tags` was read from a native tag store. |
| `FileMetadataSnapshot.IsHidden As Boolean` | Read-only. macOS: Finder hidden. Linux: leading dot. Windows: `FILE_ATTRIBUTE_HIDDEN`. |
| `FileMetadataSnapshot.IsPackage As Boolean` | Read-only package-directory flag; False sentinel off macOS. |
| `FileMetadataSnapshot.HasPackageStatus As Boolean` | True only on macOS. |
| `FileMetadataSnapshot.IsApplication As Boolean` | Read-only application flag; False sentinel off macOS. |
| `FileMetadataSnapshot.HasApplicationStatus As Boolean` | True only on macOS. |
| `FileMetadataSnapshot.ExcludedFromBackup As Boolean` | Read-only backup-exclusion flag; False sentinel off macOS. |
| `FileMetadataSnapshot.HasBackupExclusion As Boolean` | True only on macOS. |
| `SetFileTags(file As FileSystemItem, tags As Array Of String)` | Replace the entire tag list; an empty array clears it. Writes user data. macOS: Finder tags. Linux: `user.xdg.tags`, comma-joined UTF-8, following symbolic links. Windows: always `UnsupportedOperationException`. |
| `FileMetadataException` | Linux/Windows native failure with read-only `Operation`, `Domain` and `NativeCode` (equal to `ErrorCode`). |

## Linux tag format

The format matches KDE KFileMetaData: UTF-8 tags joined by `,` with no escaping. Tags containing a comma are rejected because the format cannot represent them. Reading skips empty segments and keeps every other segment verbatim. An absent attribute is an observed empty list (`HasTags = True`). `ENOTSUP`, or an entry that is not a regular file or directory, gives `HasTags = False`. `buoy-filemetadata` code 1 means the attribute contains NUL, code 2 means it is not valid UTF-8, and code 3 means a Windows name is not valid UTF-16. Only `user.xdg.tags` is read or written; other attributes are untouched.

## Errors and availability

Snapshots work on macOS, Linux and Windows (`FileMetadataSnapshotSupported()`). Tags work on macOS (Finder tags) and Linux (the `user.xdg.tags` extended attribute); `FileTagsSupported()` is False on Windows, where `SetFileTags` throws `UnsupportedOperationException`. `FileMetadataSupported()` keeps its original meaning and is True only on macOS. Off macOS, `IsPackage`, `IsApplication` and `ExcludedFromBackup` are False sentinels, and `HasPackageStatus`, `HasApplicationStatus` and `HasBackupExclusion` report that. `HasTags` is False when `Tags` is a sentinel rather than an observed list.

Invalid input throws `InvalidArgumentException`: a Nil or missing file, a blank tag or one containing NUL, and on Linux a comma or a serialized value over 65536 bytes. Tag lists are validated before any native call, so a rejected list leaves the stored tags unchanged. `SetFileTags` validates tag text before checking the platform, so invalid tags throw `InvalidArgumentException` on Windows too. Linux and Windows native failures throw `FileMetadataException`, a subclass of `Exception`, with `Operation`, `Domain` (`errno`, `Win32` or `buoy-filemetadata`) and `NativeCode`. The macOS backend keeps its original exceptions. A malformed Linux tag attribute (a NUL byte or invalid UTF-8) throws rather than decoding lossily. The package neither grants filesystem permissions nor starts security-scoped access. Snapshot properties are copied values and need no close operation; they are read in separate native calls and are not atomic against concurrent changes.
## Continue reading

- [Overview](index.md)
- [Getting started](getting-started.md)
- [API reference](api.md)


[Full typed API: properties, parameters, defaults, and returns](typed-api.md)
