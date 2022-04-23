# Extension System: Architecture RFC

*Author: [@sr229](https://git.io/sr229)*

## Table of Contents
1. [Introdcution](#introduction)
2. [Architecture](#architecture)
    - [Lifecycle of a Event](#lifecycle-of-a-event)
3. [Design Considerations](#design-considerations)

## Introduction

For any well-structured runtime to scale up to the user's needs without impacting performance, the runtime must be robust and scalable. JavaScript by default does not scale well as it is a single-threaded runtime, with every V8 backed by it's own thread. 

Obviously, this does not scale well, however, where JavaScript lags behind, is where we can compensate on the host side as most of the time an extension interfaces not in JavaScript-native interfaces, but on host-backed interface.

## Architecture

Inspired of [libuv](https://libuv.org), the library backing the well-known [Node.js](https://nodejs.org) runtime, Vignette will implement a fully-asynchronous, platform-agnostic, event-based architecture. 

![VSEL](https://yuri.might-be-super.fun/87j6kPv.png)

The Vignette Scripting Event Loop (VSEL), comprises of engine primitives and I/O operations, backed by the .NET Thread Pool Library (TPL) and a .NET-native Event Queue system. 


### Lifecycle of a Event

Similar in vein to libuv, everything starts in the I/O loop, or colloquially, the event loop. This establishes for every host operation from the scripting side to the host. The VSEL is tied to a single thread, similar in vein to V8 Isolates, and is possible to run multiple VSEL instances. However, unlike libuv, VSEL **MUST** be **thread-safe** unless stated otherwise due to engineering limitations.

The event loop follows libuv in a sense that despite being a single-threaded, any operation to the host (Network, I/O, Engine, UI, etc.) is fully asynchronous and non-blocking which are polled for completion. Every iteration of the loop will block waiting for the host operation status which had been added to the poller, and callbacks will be fired indicating the operation's conditions.

To better understand how VSEL operates, the following diagram illustrates every stage of the loop's iteration:

![VSEL Lifecycle](https://yuri.might-be-super.fun/67cGsqh.png)

- The loop concept of "now" is updated. The event loop caches the current time at the start of the VSEL loop tick to reduce the number of time-related system calls.

- If the loop is *alive*, an iteration is started. Otherwise, the loop will exit immediately. A loop's liveliness is determined if a VSEL loop has an active and referenced handles, active requests, or closing handles.

- Pending callbacks are then called. All and any host operation in this stage is called right after polling for host operation status, for the most part. There are cases where calling a callback might be deferred for the next loop iteration. If there are any host operations deferred previously, it will be run at this point.

- Idle handle callbacks are then called. These callbacks run on every iteration, if they are active.

- Poll timeout is then calcuated. Before blocking for any host operation, the loop calculates how long should it block. The rules are as follows:

   - If a specific method has a `[DoNotWait]` attribute, the timeout is 0ms.
   - If the loop is about to be stopped, the timeout is 0ms.
   - If there aren't any active handles or requests, the timeout is 0ms.
   - If there any idle handles active, the timeout is 0ms.
   - If there are any handles pending to be closed, the timeout is 0ms.
   - If none of the above cases matches, the timeout of the closest timer is taken, or if there are no active timers, the timeout is `Infinity`.

- The loop blocks waiting for the host operation. At this point the loop will block waiting for the host for the calculated duration from the previous step.

- Check handle callbacks are called. Check handles get their callbacks called right after the loop has blocked waitig for the host operation. Check handles are essentially the counterpart of prepare handles.

- Close callbacks are called. If a callback closes using a specified method to close the callback immediatedly, it will get the close callback called.

- In the special case a loop is only meant to run once with the on some special occassions, it implies forward progress. It's possible no host operation callbacks had been called after blocking for host, but some time has passed so there might be timers which are due, which will be called too.

- The loop's iteration ends. By default, our loop will continue from the start while it's alive, however, if it's a forward progress loop as stated previously, the iteration will end and will return.

## Design Considerations

While VSEL is modeled around the foundations of industry-tested libraries, there are some special design considerations which differs from these libraries:

- Under no circumstance the VSEL environment will rely on host-native functions. The internal implementation of VSEL should be pure .NET.

- Every operation must be thread-safe, including the VSEL instance itself, unless otherwise stated due to engineering reasons.

- The bindings for the scripting engine **MUST** be mapped in a non-blocking and asynchronous fashion - make use of the highly-parallel .NET `ThreadPool` as much as possible.


## References

- [Design overview - Libuv Documentation](http://docs.libuv.org/en/v1.x/design.html)

- [`epoll(7)` documentation - manpages](https://man7.org/linux/man-pages/man7/epoll.7.html)

- [.NET ThreadPool - Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/api/system.threading.threadpool?view=net-6.0)

- [Thread pool work scheduling - Libuv Documentation](http://docs.libuv.org/en/v1.x/threadpool.html#threadpool)