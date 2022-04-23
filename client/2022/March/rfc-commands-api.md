# Commands API RFC
*Author: [@sr229](https://git.io/sr229) and [@LeNitrous](https://github.com/LeNitrous)*

## Table of Contents
1. [Introduction](#Introduction)
2. [Format](#Format)
3. [Data Interchange](#Data-Interchange)
    - [Allowed types in interactive mode](#Allowed-types-in-interactive-mode)
    - [Allowed types in extension API](#Allowed-types-in-extension-API)


## Introduction

As extensions do not have a view of the outside world, there is only limited ways they can communicate to the user and other extensions. This comes in a form of Commands. Commands are small executable code that can be invoked interactively or via the extension API.

## Format

A command may have the following format to invoke it's method:

```
<vendor>.<extension>:<method> <args[]>
```

Which can translate to the following:

```typescript

async function sayHello(name: string) {
    return `Hello ${name}`;
}

async function activate(context: vignette.ExtensionContext) {
    vignette.commands.register("vendor.example:sayHello", sayHello(context.args));
}

```

This allows simple data interchange and invocation between commands, which allows a simple but straightforward inter-process communication between extensions.


## Data Interchange

Commands are given their own specific recieving channel unique to each extension, and cannot be listened by another extension. There are security reasons for such:

- This prevents any kind of Man-in-the-middle attacks by any malicious extension if such extensions are transfering sensitive data (Camera data, tracking data, etc.)

- A unique token aside from the vendor provided extension ID also ensures that only the intended recipient gets the message.

- Commands are one way channels, and channels are disposed of immediately on restarts or when the target extension restarts and/or is replaced by a newer version.

### Allowed types in interactive mode

Vignette may come with a interactive mode shell to test how these commands work in a user-interactable environment. However, they're only allowed to transfer the following.

- `Number`
- `String`
- `boolean`


### Allowed types in extension API

Vignette does not impose restrictions on which data to transfer between extensions, and as such, any JavaScript data is allowed.