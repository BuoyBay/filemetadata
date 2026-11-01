# File metadata and tags · 0.1.0

`Import filemetadata`. `New FileMetadataSnapshot(file)` reads `DisplayName`, `Tags`, `IsHidden`, `IsPackage`, `IsApplication`, and `ExcludedFromBackup`. `SetFileTags(file, tags)` explicitly replaces the file's tags; an empty array clears them. Reading a snapshot never changes metadata and never reads file contents. Access errors throw.

## Platforms

| | macOS | Linux | Windows |
| --- | --- | --- | --- |
| Backend | NSURL resource values (unchanged) | path name + `user.xdg.tags` extended attribute | `FindFirstFileW` / `GetFileAttributesW` |
| `DisplayName` | localized name | final path component; `.`/`..` resolved with `realpath` | on-disk entry name (exact casing); a drive or share root reports the full path |
| `IsHidden` | Finder hidden state | name starts with `.` (`.hidden` lists are not read) | `FILE_ATTRIBUTE_HIDDEN` |
| `Tags` / `HasTags` | Finder tags / always True | `user.xdg.tags` / see below | empty / always False |
| `IsPackage`, `IsApplication`, `ExcludedFromBackup` | real values | False sentinels | False sentinels |
| `SetFileTags` | Finder tags | `user.xdg.tags` | `UnsupportedOperationException` |

Predicates are compile-time only and touch no file:

- `FileMetadataSupported()` is frozen to its original meaning: True only on macOS.
- `FileMetadataSnapshotSupported()` is True on macOS, Linux and Windows.
- `FileTagsSupported()` is True on macOS and Linux, False on Windows.

Off macOS, `HasPackageStatus`, `HasApplicationStatus` and `HasBackupExclusion` are False and the matching fields are False sentinels: Linux and Windows have no Finder package, application or backup-exclusion metadata, and the package does not invent any. `HasTags` is False when `Tags` is a sentinel rather than an observed list.

## Linux tags

The format follows KDE KFileMetaData, the main `user.xdg.tags` implementation: UTF-8 tags joined by `,` with no escaping. Reading skips empty segments and keeps every other segment verbatim, including spaces; writing preserves order and duplicates. An empty list removes the attribute, and clearing an attribute that is already absent succeeds. Other extended attributes are never read or changed.

- **Rejected before any native call** (`InvalidArgumentException`; the stored value is unchanged): a blank tag, a tag containing NUL or a comma (commas cannot be represented), or a serialized value over 65536 bytes (the kernel `XATTR_SIZE_MAX`). A filesystem limit below that is reported natively. For example, btrfs refuses 64 KiB with `ENOSPC` and keeps the old value.
- **Absent attribute**: `HasTags = True`, `Tags` empty.
- **No tag store**: `HasTags = False` when the filesystem returns `ENOTSUP` or the entry is not a regular file or directory (the only kinds that may carry `user.*` attributes). Writing to such an entry throws the native error, for example `EPERM` for a FIFO.
- **Symbolic links**: the name and hidden flag describe the link entry. Tags are read from and written to the link target, as KDE does. A dangling link throws `InvalidArgumentException`.
- **Malformed foreign value**: a NUL byte or invalid UTF-8 throws `FileMetadataException` (`Domain = "buoy-filemetadata"`, code 1 or 2) instead of returning a lossy decode.
- **Native failures** (for example `EACCES` without read or write permission) throw `FileMetadataException` with `Domain = "errno"` and the raw errno in `NativeCode`/`ErrorCode`. Messages never include paths or tag text.
- **Bounds and races**: one `getxattr` call into a 64 KiB buffer, so there is no size-then-read race. The type check, name and tags are separate calls, so a snapshot is not atomic against a concurrent rename or attribute write.

## Windows

Windows has no tag store in this package. `SetFileTags` validates tag text first (existing order) and then always throws `UnsupportedOperationException`, including for an empty list, so tags are never silently dropped. The snapshot never opens the file: a junction or symbolic link reports its own entry (even when its target is missing), and paths are made absolute lexically (`GetFullPathNameW`). Wildcards (`*`, `?`) throw `InvalidArgumentException`. A missing file (`ERROR_FILE_NOT_FOUND`/`ERROR_PATH_NOT_FOUND`) throws `InvalidArgumentException`. Other failures throw `FileMetadataException` with `Domain = "Win32"`, or `buoy-filemetadata` code 3 for a name that is not valid UTF-16. Existence comes from the native lookup, not `FileSystemItem.Exists`, because in Buoy 0.72 `Exists` misses Unicode names on Windows.

## Validation


Maintainers run the private validation suite before release. See the repository README for public build instructions.

