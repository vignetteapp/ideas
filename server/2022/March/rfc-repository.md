# Extension System: Remote Repository RFC

*Author: @JcdeA*

## Table of Contents
1. [Introduction](#Introduction)
2. [Implementation](#Implementation)

## Introduction
Vignette will operate an extension repostitory, inspired by the Debian's package repostory. The main goals of this RFC is to ensure an open future for all users, and to create an easily replicable structure, so third-party implementations can be created.

## Implementation
The repository will be a plain HTTP(S) file server. This has many benefits, such as the ease of utilizing a CDN.

There are certain restrictions to the structure of the filesystem, and the filenames.

### Filesystem Structure
Each VAST Extension should be stored in the following format:
```
publisher/
    extname/
        version/
            publisher.extname-version.vast
```
The publisher name, version number, extension name should match the following regular expression: `^[a-zA-Z0-9-_.]+$`.



