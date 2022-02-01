# Asset System RFC

## Table of Contents
1. [File Format](#File-Format)
2. [File System](#File-System)
    - [Scripts](#Scripts)
    - [Data](#Data)
    - [Locale](#Locale)
3. [Metadata](#Metadata)

## File Format
The Vignette Asset file format is basically an archive with `.vast` as its given extension. This archive contains metadata required by the client as well as files that the asset will use which are but not limited to scripts used to extend the client's capabilities, models, textures, shaders, and sounds.

## File System
Assets may require access to files. To provide a secure access, the client can provide a virtual file system for use. The root of the asset's archive is the root of the virtual file system. The contents are read-only and cannot be modified.

### Scripts
For scripts to be loaded and be visible to other scripts for import, they must be in the asset archive's `scripts` directory. Any scripts outside this directory will be ignored.

### Data
There may be cases where scripts may need to write to disk. A special directory named `.data` will be mounted. The physical location of this directory is located in `.data/<extension-id>/` which is found in the same location as the client executable.

### Locale
Scripts may show display text. To make text language-aware, localization files can be provided through the `lang` directory. This directory contains JSON files wherein the file names must correspond to the ISO-3166 Country Code and ISO-639 Language Code. For instance, US English is `en-US`.

The contents of the JSON file must contain an object with key-value pairs of the string ID and display text which can be easily provided by localization providing services like Crowdin.

## Metadata
The metadata file contains necessary information for the client to determine how this asset should be handled. It is a JSON file with the given schema.

```json
{
    "id": "my-extension",
    "name": "My Extension",
    "author": "The Vignette Authors",
    "version": {
        "asset": "1.0.0",
        "spec": "1.0.0"
    },
    "permissions": [
        "tracking",
        "rendering",
        "workbench"
    ],
    "dependencies": [
        {
            "id": "my-other-extension",
            "version": "1.0.0"
        }
    ]
}
```

|Key|Value|
|---|-----|
|`id`|The unique identifier for this asset.|
|`name`|The display name for this asset.|
|`author`|The display name for the author for this asset.|
|`version`|An object describing the asset's version and the required client version. <table><th>Key</th><th>Value</th><tr><td>`asset`</td><td>The asset's version in semantic versioning format.</td></tr><tr><td>`spec`</td><td>The asset specification version in semantic versioning format.</td></tr></table>|
|`permissions`| An array of strings determining what aspects of the client scripts will need access to.|
|`dependencies`| An array of objects determining the dependencies of this asset. The asset will not be loaded if a dependency is not found.<table><th>Key</th><th>Value</th><tr><td>`id`</td><td>The dependency asset's unique identifier.</td></tr><tr><td>`version`</td><td>The dependency asset's version in semantic versioning format.</td></tr></table>|
