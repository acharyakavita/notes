# Node

## Core Modules

`http` - Classes, methods and events to create Node.js http server

`url` - Methods for URL resolution and parsing

`querystring` - Methods to deal with query string

`path` - Methods to deal with file paths

`fs` - Classes, methods, and events to work with file I/O

`util` - Utility functions useful for programmers

## Event Loop

The event loop is what allows Node.js to perform non-blocking I/O operations — despite the fact that JavaScript is single-threaded — by offloading operations to the system kernel whenever possible.

Since most modern kernels are multi-threaded, they can handle multiple operations executing in the background. When one of these operations completes, the kernel tells Node.js so that the appropriate callback may be added to the `poll` queue to eventually be executed.

```
   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘
```

Each phase has a FIFO queue of callbacks to execute

### timers

This phase executes callbacks scheduled by `setTimeout()` and `setInterval()`.

### pending callbacks

Executes I/O callbacks deferred to the next loop iteration.

### idle, prepare

Only used internally.

### poll

Retrieve new I/O events; execute I/O related callbacks (almost all with the exception of close callbacks, the ones scheduled by timers, and `setImmediate()`); node will block here when appropriate.

### check

`setImmediate()` callbacks are invoked here.

### close callbacks

Some close callbacks, e.g. socket.on('close', ...).

Between each run of the event loop, Node.js checks if it is waiting for any asynchronous I/O or timers and shuts down cleanly if there are not any.

## process.nextTick() and setImmediate()

`nextTick()` postpones the execution of action until the next pass around the event loop, or it simply calls the callback function once the event loop's current execution is complete, whereas `setImmediate()` executes a callback on the next cycle of the event loop and returns control to the event loop for any I/O operations.

## EventEmitter

A class that holds all the objects that can emit events. Whenever an object from the EventEmitter class throws an event all attached functions are called upon synchronously.

```javascript
const EventEmitter = require('events');
const eventEmitter = new EventEmitter();
..
eventEmitter.on('start', () => {
  console.log('started');
});
..
eventEmitter.emit('start');
```

Has other events such as:

`once()`: add a one-time listener

`removeListener()` / `off()`: remove an event listener from an event

`removeAllListeners()`: remove all listeners for an event

## Streams

A stream is an abstract interface for working with streaming data in Node.js. The `node:stream` module provides an API for implementing the stream interface. Streams can be readable, writable, or both and all streams are instances of `EventEmitter`.

### Types

`Writable`: streams to which data can be written (for example, fs.createWriteStream()).

`Readable`: streams from which data can be read (for example, fs.createReadStream()).

`Duplex`: streams that are both Readable and Writable (for example, net.Socket).

`Transform`: Duplex streams that can modify or transform the data as it is written and read (for example, zlib.createDeflate()).

## Buffer

Buffer objects are used to represent a fixed-length sequence of bytes and is a subclass of JavaScript's `Uint8Array` class and extends it with methods that cover additional use cases.

## Piping

Piping in NodeJS is useful when we need to write some data coming from a source stream to another stream. We basically pipe the read stream and write stream to one another.

## Blocking I/O

Node.js uses an event-driven, non-blocking I/O model that allows it to handle I/O operations more efficiently. By using callbacks, Node.js can continue processing other tasks while waiting for I/O operations to complete. This means that Node.js can handle multiple requests simultaneously without causing any delays. Additionally, Node.js uses a single-threaded event loop architecture, which allows it to handle a high volume of requests without any issues.

## Child process

The `child_process.spawn()` method spawns the child process asynchronously, without blocking the Node.js event loop. The `child_process.spawnSync()` function provides equivalent functionality in a synchronous manner that blocks the event loop until the spawned process either exits or is terminated.

`child_process.exec()`: spawns a shell and runs a command within that shell, passing the `stdout` and `stderr` to a callback function when complete.

`child_process.execFile()`: similar to `child_process.exec()` except that it spawns the command directly without first spawning a shell by default.

`child_process.fork()`: spawns a new Node.js process and invokes a specified module with an IPC communication channel established that allows sending messages between parent and child.

`child_process.execSync()`: a synchronous version of `child_process.exec()` that will block the Node.js event loop.

`child_process.execFileSync()`: a synchronous version of `child_process.execFile()` that will block the Node.js event loop.
