---
title: "File metadata getting started"
summary: "Install and use filemetadata with an availability check and a complete example."
category: "Guides"
order: 10
keywords: ["filemetadata", "install", "example"]
related: ["index.md", "api.md", "typed-api.md"]
---

# Getting started

## Install and import

Copy the sealed `filemetadata-0.1.0.buipkg` archive into your application's `packages/` folder.

## First use

Construct `FileMetadataSnapshot` with an accessible local file or directory. The `Tags` getter returns a copy, so changing the returned array changes neither the snapshot nor the file. To change tags, call `SetFileTags` deliberately with the complete replacement list; an empty array clears them.

```basic
Import filemetadata

Sub Main()
    If Not FileMetadataSnapshotSupported() Then
        Print "File metadata is not available on this platform."
        Return
    End If
    Dim metadata As New FileMetadataSnapshot(New FileSystemItem("."))
    Print metadata.DisplayName + " hidden=" + Str(metadata.IsHidden)
    If metadata.HasTags Then
        For Each tag As String In metadata.Tags
            Print tag
        Next
    End If
    If metadata.HasPackageStatus Then Print "package=" + Str(metadata.IsPackage)
End Sub
```

The example checks the availability predicate and each `Has*` flag before using a platform-specific value. It does not catch input or access failures, so catch exceptions at your application boundary and show the user a useful message. Gate tag editing on `FileTagsSupported()`. On Linux, check `HasTags` for the file too: a filesystem without user extended attributes, or a special file, has no tag store.

## Platform and error handling

Snapshots work on macOS, Linux and Windows (`FileMetadataSnapshotSupported()`). Tags work on macOS (Finder tags) and Linux (the `user.xdg.tags` extended attribute); `FileTagsSupported()` is False on Windows, where `SetFileTags` throws `UnsupportedOperationException`. `FileMetadataSupported()` keeps its original meaning and is True only on macOS. Off macOS, `IsPackage`, `IsApplication` and `ExcludedFromBackup` are False sentinels, and `HasPackageStatus`, `HasApplicationStatus` and `HasBackupExclusion` report that. `HasTags` is False when `Tags` is a sentinel rather than an observed list.

Invalid input throws `InvalidArgumentException`: a Nil or missing file, a blank tag or one containing NUL, and on Linux a comma or a serialized value over 65536 bytes. Tag lists are validated before any native call, so a rejected list leaves the stored tags unchanged. `SetFileTags` validates tag text before checking the platform, so invalid tags throw `InvalidArgumentException` on Windows too. Linux and Windows native failures throw `FileMetadataException`, a subclass of `Exception`, with `Operation`, `Domain` (`errno`, `Win32` or `buoy-filemetadata`) and `NativeCode`. The macOS backend keeps its original exceptions. A malformed Linux tag attribute (a NUL byte or invalid UTF-8) throws rather than decoding lossily. The package neither grants filesystem permissions nor starts security-scoped access. Snapshot properties are copied values and need no close operation; they are read in separate native calls and are not atomic against concurrent changes.
## Continue reading

- [Overview](index.md)
- [Getting started](getting-started.md)
- [API reference](api.md)
