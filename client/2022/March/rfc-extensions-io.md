# Extension System : IO RFC

## Table of Contents
1. [Introduction](#Introduction)
2. [Filesystem Access](#Filesystem-Access)
    - [API](#API)
    - [Data Representation](#Data-Representation)
3. [Networking](#Networking) 


## Introduction

Vignette maintains a comprehensive and robust API, inspired of VSCode. However, there are certain considerations to take into account. To design a successful extension system, we must take account of both security of the users and the flexibility we provide to the developers.

## Filesystem Access

Access to the file system is considered a dangerous permission to grant due to the dangers it present. As such the objective of the extension system is only to provide the following:

* Do NOT grant host level filesystem access to the extension.
* Present a limited API that covers common use-cases.
* Do NOT allow the extension to manage OS-specific file-related attributes.

### API

Vignette's FS system will be a reduced version of the FS API of Node.js. A basic file operation may look something like this.

```typescript
import { vignette } from 'vignette';
import { fs } from 'fs';

export function activate() {
    const fileBuf = fs.readFile('./data.txt');
    if (typeof fileBuf === Blob)
      vignette.notification.send('info', 'Successfully read data.txt');
    else 
      vignette.notification.send('error', `Expected Blob, got ${typeof fileBuf} instead.`)
}
```

The filesystem API only allows basic file operations, `readFile`, `unlink`, and `writeFile` and only takes a `Buffer` or a `Blob` as a input.

### Data Representation

Data that is read through the filesystem API must be either `Buffer` or `Blob`, and vice versa must recieve a `Blob` or a `Buffer` too. This simplifies the need to create our own types.

## Networking

Vignette implements the asynchronous `fetch()` API as a module. `XMLHttpResponse()` is unsupported. 

```typescript
import { fetch, Request, Response } from 'fetch';

export async function activate() {
   const reqObj = new Request('https://henke.moe');
   const res = await fetch(reqObj);

   if (typeof res === Response)
     vignette.notification.send('info', 'Successfully queried to https://henke.moe');
   else
     vignette.notification.send('error', 'Error querying to remote.');
}

```

However, `fetch()` will require the `net` intent otherwise Vignette will throw a `HostError`.

```
HostError: Extension tried to access fetch() without the 'net' intent. Amend your extension's manifest to grant the intent.
```
