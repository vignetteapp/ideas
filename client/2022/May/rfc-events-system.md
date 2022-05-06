# Events and Timers RFC 
*Author: [@sr229](https://github.com/sr229)*

## Table of Contents

- [Introduction](#introduction)
- [Events API](#events-api)
   - [Supported Host Events](#supported-host-events)
- [Timers API](#timers-api)

## Introduction

To track state changes from the engine to the script, Vignette introduces a version of the events and timers system, inspired of Garry's Mod and Node.js. This complements the Event Loop system for Vignette and ensures an accessible and easy way to track or handle state changes between realms.


## Events API

Consider the following code as an example:

```ts
import { vignette, Event } from 'vignette';

const emitter = new Event();

emitter.on("onTick", function() => {
    vignette.console.log("Tick!");
});
```

Inspired of Node.js `EventEmitter` and Garry's Mod hooks, Vignette events hooks on any host behavior, may it be engine, the render or even the event loop's tick. The API would be identical to `EventEmitter`, with our own additions whenever we see fit.

## Supported Host Events

By default Vignette would provide these builtin events.

- `load`: runs whenever the extension is loaded.
- `unload`: runs whenever the extension is about to be unloaded.
- `onTick`: runs at the end of the game loop's lifecycle.
- `onKeyPress`: runs on keypresses. This is useful for handling macros or your own shortcuts.

This list is non-exhaustive, and would differ on implementation. All these events must be a [`EventTarget`](https://nodejs.org/api/events.html#eventtarget-and-event-api).

## Timers API

V8 provides `setTimeout()` and `setInterval()` APIs, therefore, this is also supported in Vignette and is supported by the VSEL loop.

```ts
setTimeout(() => vignette.console.log("This fires after 5000ms"), 5000);

setInterval(() => vignette.console.log("this fires every 500ms"), 500);
```

## References

- [`setInterval()` - MDN](https://developer.mozilla.org/en-US/docs/Web/API/setInterval)

- [`setTimeout()` - MDN](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout)

- [`EventEmitter` reference - Node.js](https://nodejs.org/api/events.html#class-eventemitter)

- [`Event` and `EventTarget` - Node.js](https://nodejs.org/api/events.html#eventtarget-and-event-api).