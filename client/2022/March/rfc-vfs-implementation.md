# The Virtual Filesystem RFC

## Table of Contents
1. [Introduction](#Introduction)
2. [Structure](#Structure)
3. [Limitations](#Limitations)

## Introduction

Vignette Extensions by default are untrusted, and only given limited access to the host system. As such, Vignette implements a virtual filesystem structure, utilizing Stride's VFS implementation, inspired of overlayfs from Docker.

## Structure

The VFS for the extensions are implemented as follows

```
 -------------------------
|  Writeable Layer (/data) |   < ---- Mapped from $GAME_DIR/data/$EXT/
 ------------------------- 
 ==========================
|      Readonly Layer       |
|    ------------------     |
|   |    VAST Bundle   |    | < ----- Mapped from $GAME_DIR/ext/$EXT.vast
|    ------------------     |
 ==========================
```

- The writeable layer is layered on top of the Readonly layer, appearing transparent to the developer. This means that every I/O operation that happens inside the `EXT_ROOT` is actually being redirected to `$GAME_DIR/data/$EXT/`.
   - This means you can read and write on `./`, but it does not mean you can modify your extension files you already bundled in the VAST bundle.

- The readonly layer is a virtual filesystem layer based on the central directory's contents contained in your VAST's headers. Files that were overlaid here cannot be written and modified, and can only be read.
  - While it may be possible to copy your extension's files to the writeable layer, the extension system is designed to prevent your context from being overriden. Hence, you cannot modify your extension's main objects by simply telling the your script context to use the writeable layer context.

## Limitations

Of course, such security safeguards also poses limitations, as Vignette's extension system is not an all-encompassing runtime like Node.js, particularly the following:

  - Extensions are isolated away from the host and each other, they can only read and write files within their own namespace.

  - File attributes are not available to the scripting side. Only basic operations such as listing, making, and removing files and directories are supported.

  - As the VAST file bundle is a ZIP-based system, you cannot modify your VAST bundle programmatically unless you recompile your VAST bundle.

  - Vignette operates on the concept of least-privilege: this means you still need to explicitly tell the core to grant you FS access otherwise it will return a `HostError`.