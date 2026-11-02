---
title: "filemetadata typed API"
summary: "Source-derived public signatures, properties, parameters, defaults, and return values."
category: "Reference"
order: 21
types: ["FileMetadataException", "FileMetadataSnapshot"]
dependencies: []
symbols: ["FileMetadataException", "FileMetadataSnapshot"]
related: ["api.md", "getting-started.md", "index.md"]
---

# filemetadata typed API

Generated from the package source declarations. Platform guards are shown where present; inherited members are documented on the base type. See the [API contracts](api.md) for behavior and limitations.

### FileMetadataSupported

`Public Function FileMetadataSupported() As Boolean`

Frozen: True only where the original macOS Finder/NSURL backend runs. Portable code gates on FileMetadataSnapshotSupported and FileTagsSupported. Compile-time only; read-only.

No parameters.

#### Returns

| Type |
| --- |
| `Boolean` |

### FileMetadataSnapshotSupported

`Public Function FileMetadataSnapshotSupported() As Boolean`

True when this compile target can construct FileMetadataSnapshot: macOS (NSURL resource values), Linux (path name plus user.xdg.tags) and Windows (FindFirstFileW). Compile-time only: it touches no file. Read-only. Check the snapshot's Has* properties before trusting a platform field.

No parameters.

#### Returns

| Type |
| --- |
| `Boolean` |

### FileTagsSupported

`Public Function FileTagsSupported() As Boolean`

True when this compile target implements file tags: macOS Finder tags and the Linux user.xdg.tags extended attribute. False on Windows, where SetFileTags throws UnsupportedOperationException. Compile-time only; a Linux filesystem without user extended attributes still fails per file. Read-only.

No parameters.

#### Returns

| Type |
| --- |
| `Boolean` |

### SetFileTags

`Public Sub SetFileTags(file As FileSystemItem, tags As Array Of String)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `file` | `FileSystemItem` | `ByVal` | Required |
| `tags` | `Array Of String` | `ByVal` | Required |

Returns: no value.

## FileMetadataException

`Public Class FileMetadataException Inherits Exception`

Linux and Windows native failure. Domain is "errno" (Linux), "Win32" (GetLastError) or "buoy-filemetadata" for conditions this package detects: 1 = tag attribute contains a NUL byte, 2 = tag attribute or resolved name is not valid UTF-8, 3 = native file name is not valid UTF-16. NativeCode equals ErrorCode. Messages never include paths or tag text. The macOS backend keeps throwing its original Exception messages.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `Operation` | `String` | Yes | No | — | — |
| `Domain` | `String` | Yes | No | — | — |
| `NativeCode` | `Integer` | Yes | No | — | — |

### Constructor

`Public Sub Constructor(message As String, nativeOperation As String, nativeDomain As String, code As Integer)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `message` | `String` | `ByVal` | Required |
| `nativeOperation` | `String` | `ByVal` | Required |
| `nativeDomain` | `String` | `ByVal` | Required |
| `code` | `Integer` | `ByVal` | Required |

Returns: no value.

## FileMetadataSnapshot

`Public Class FileMetadataSnapshot`

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `DisplayName` | `String` | Yes | No | — | — |
| `Tags` | `Array Of String` | Yes | No | — | — |
| `IsHidden` | `Boolean` | Yes | No | — | — |
| `IsPackage` | `Boolean` | Yes | No | — | — |
| `IsApplication` | `Boolean` | Yes | No | — | — |
| `ExcludedFromBackup` | `Boolean` | Yes | No | — | — |
| `HasTags` | `Boolean` | Yes | No | — | True when Tags was read from a native tag store: always on macOS; on Linux when the entry (after following links) is a regular file or directory on a filesystem with user extended attributes. Always False on Windows. When False, Tags is an empty sentinel, not an observed list. |
| `HasPackageStatus` | `Boolean` | Yes | No | — | True only on macOS. Elsewhere IsPackage is a False sentinel. |
| `HasApplicationStatus` | `Boolean` | Yes | No | — | True only on macOS. Elsewhere IsApplication is a False sentinel. |
| `HasBackupExclusion` | `Boolean` | Yes | No | — | True only on macOS. Elsewhere ExcludedFromBackup is a False sentinel. |

### Constructor

`Public Sub Constructor(file As FileSystemItem)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `file` | `FileSystemItem` | `ByVal` | Required |

Returns: no value.

### FileMetadataLinuxEntryKind

`Public Function FileMetadataLinuxEntryKind(entryPath As String, ByRef taggable As Boolean) As Integer`

Classify the entry after following links with statx(STATX_TYPE). taggable = True only for regular files and directories, the only kinds that may carry user.* attributes (xattr(7)). Read-only.

Availability: `#If TargetLinux Then`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `entryPath` | `String` | `ByVal` | Required |
| `taggable` | `Boolean` | `ByRef` | Required |

#### Returns

| Type |
| --- |
| `Integer` |

### FileMetadataLinuxReadTags

`Public Function FileMetadataLinuxReadTags(entryPath As String, value As Bytes) As Integer`

Copy the raw user.xdg.tags bytes into value. A single getxattr call into a VFS-maximum buffer avoids the size-then-read race. Read-only.

Availability: `#If TargetLinux Then`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `entryPath` | `String` | `ByVal` | Required |
| `value` | `Bytes` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Integer` |

### FileMetadataLinuxWriteTags

`Public Function FileMetadataLinuxWriteTags(entryPath As String, value As Bytes) As Integer`

Create or atomically replace user.xdg.tags with a non-empty value. Writes user data.

Availability: `#If TargetLinux Then`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `entryPath` | `String` | `ByVal` | Required |
| `value` | `Bytes` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Integer` |

### FileMetadataLinuxRemoveTags

`Public Function FileMetadataLinuxRemoveTags(entryPath As String) As Integer`

Remove user.xdg.tags. Returns the raw errno, including ENODATA when the attribute was already absent. Writes user data.

Availability: `#If TargetLinux Then`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `entryPath` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Integer` |

### FileMetadataLinuxRealPath

`Public Function FileMetadataLinuxRealPath(entryPath As String, resolved As Bytes) As Integer`

realpath(3) bytes for an existing path; used only to name "." and "..". Read-only.

Availability: `#If TargetLinux Then`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `entryPath` | `String` | `ByVal` | Required |
| `resolved` | `Bytes` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Integer` |

### FileMetadataWindowsFullPath

`Public Function FileMetadataWindowsFullPath(entryPath As String, ByRef fullPath As String) As Integer`

Lexical GetFullPathNameW: resolves "." and ".." without touching the disk. Read-only.

Availability: `#If TargetWindows Then`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `entryPath` | `String` | `ByVal` | Required |
| `fullPath` | `String` | `ByRef` | Required |

#### Returns

| Type |
| --- |
| `Integer` |

### FileMetadataWindowsEntry

`Public Function FileMetadataWindowsEntry(fullPath As String, ByRef entryName As String, ByRef attributes As Integer) As Integer`

FindFirstFileW on a non-root path: the on-disk entry name and attributes in one directory read. A reparse point (symbolic link or junction) reports its own entry, never its target. Read-only.

Availability: `#If TargetWindows Then`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `fullPath` | `String` | `ByVal` | Required |
| `entryName` | `String` | `ByRef` | Required |
| `attributes` | `Integer` | `ByRef` | Required |

#### Returns

| Type |
| --- |
| `Integer` |

### FileMetadataWindowsAttributes

`Public Function FileMetadataWindowsAttributes(fullPath As String, ByRef attributes As Integer) As Integer`

GetFileAttributesW, used for volume and share roots, which have no directory entry for FindFirstFileExW. Read-only.

Availability: `#If TargetWindows Then`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `fullPath` | `String` | `ByVal` | Required |
| `attributes` | `Integer` | `ByRef` | Required |

#### Returns

| Type |
| --- |
| `Integer` |
