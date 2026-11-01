# Read and update Finder tags and native file metadata

`Import filemetadata`. `New FileMetadataSnapshot(file)` reads `DisplayName`, `Tags`, `IsHidden`, `IsPackage`, `IsApplication`, and `ExcludedFromBackup`. `SetFileTags(file, tags)` explicitly replaces the file's tags; an empty array clears them. Reading a snapshot never changes metadata and never reads file contents. Access errors throw.

## Install

From a Buoy consumer project with BPM installed:

```sh
bpm add filemetadata --git https://github.com/BuoyBay/filemetadata.git --rev refs/heads/main
```

BPM records the resolved commit in its lockfile. Keep that lockfile with the
consumer project for reproducible installs.

## Example

```buoy
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

## Build from source

With the native `build-packages` driver from [devtools](https://github.com/BuoyBay/devtools)
on your `PATH`, run from this checkout:

```sh
build-packages --root "$PWD" --compile-consumer filemetadata
```

The driver checks source targets and compiles an import consumer. See the
[API guide](docs/API.md) for behavior, limits and any host prerequisites.

## Local Buoy setup

On your own machine, install Buoy and run:

```sh
buoy ai init --both
```

The generated agent files and editor settings stay local and are ignored by Git.

## Contributing

[Contributing](CONTRIBUTING.md) · [License](LICENSE)
