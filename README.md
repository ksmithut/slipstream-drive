# slipstream-drive

An RPC framework that leverages protocol buffers to make server to server
communication fast.

# Installation

```
npm install --save slipstream-drive
```

# Usage

server usage

```proto
// greeter.proto
message Request {
  string name = 1;
}

message Response {
  string message = 1;
}

service Greeter {
  rpc SayHello (Request) returns (Response);
}
```

```js
'use strict'

const slipstream = require('slipstream-drive')

const proto = slipstream.loadFile('greeter.proto')
const server = slipstream.server()

const greeter = slipstream.service(proto.Greeter, {
  sayHello(request) {
    return { message: `Hello ${request.name}` }
  }
})

server.register(greeter)

server.start('8000')
```

client usage

```js
const slipstream = require('slipstream-drive')

const proto = slipstream.loadFile('greeter.proto')
const greeter = slipstream.client('localhost:8000', proto.Greeter)

greeter.sayHello({ name: 'Jack Bliss' })
  .then((response) => {
    console.log('Greeting:', response.message)
  })
```

# API

`slipstream` is heretofore a shortcut for `require('slipstream-drive') as far as
these docs are concerned.

## `slipstream.loadFile(path)`

Loads a `.proto` file. Services will be on the returned object with the service
names being the properties.

- `path` The path to the proto file to load. This path must be relative to cwd
  or an absolute path. Protocol buffer imports are not resolved at this time.

## `slipstream.load(protobuf)`

Loads the protocol buffer schema from a string/buffer. Services will be on the
returned object with the service names being the properties.

- `protobuf` A string/buffer of the protocol buffer definition.

## `slipstream.service(protobufService[, handlers])`

Returns a slipstream service object

- `protobufService` The protocol buffer service as returned from `.load` or
  `.loadFile`. If it's not passed, it's assumed that this will be a json
  service, rather than a protocol buffer service.
- `handlers` An object who's keys correspond with the method names of the
  service. The keys must be the lowerCamelCase version of their schema
  counterparts (if a protobuf service is passed). The values must be functions.
  Those functions can return promises so long as the promises resolve to an
  object the match the definition. Handlers can be added after this initial
  declaration.

### `service.use(methodName, handler)`

Adds a handler to a given method name.

- `methodName` The name of the method to bind to.
- `handler` The function that should be called when the method is called.

## `slipstream.server()`

Returns a new slipstream server instance

### `server.register(service)`

Registers a service with the server

- `service` The slipstream service instance to bind to that namespace.

### `server.start(port)`

Starts the server. Returns a promise that resolves when the server has started.

### `server.stop([force])`

Stops the server. Returns a promise that resolves when all remaining connections
have finished, unless force is `true`

- `force` Whether or not to force stop the server. Defaults to false
