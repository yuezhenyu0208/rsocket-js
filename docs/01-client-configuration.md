# Client Configuration

Creating a client requires that you specify rsocket settings as well as a transport
client.

## Browser Example

For browser usage, a client will typically be configured to use WebSocket as the
transport layer and to send/receive JSON objects:

```javascript
import {
  RSocketClient, 
  JsonSerializers,
} from 'rsocket-core';
import RSocketWebSocketClient from 'rsocket-websocket-client';

// Create an instance of a client
const client = new RSocketClient({
  // send/receive objects instead of strings/buffers
  serializers: JsonSerializers,
  setup: {
    // ms btw sending keepalive to server
    keepAlive: 60000, 
    // ms timeout if no keepalive response
    lifetime: 180000, 
    // format of `data`
    dataMimeType: 'application/json', 
    // format of `metadata`
    metadataMimeType: 'application/json', 
  },
  transport: new RSocketWebSocketClient({url: 'wss://...'}),
});

// Open the connection
client.connect().subscribe({
  onComplete: socket => {
    // socket provides the rsocket interactions fire/forget, request/response,
    // request/stream, etc as well as methods to close the socket.

    socket.fireAndForget({
      data: {some: {json: {value: 1}}},
      metadata: {another: {json: {value: true}}},
    });
  },
  onError: error => console.error(error),
  onSubscribe: cancel => {/* call cancel() to abort */}
});
```

## Node.js Example

For Node.js, usage will typically involve the TCP client. The following example
demonstrates the TCP client usage and sending/receiving Buffer objects (see the
above example for how to send/receive JSON objects instead):

```javascript
import {
  RSocketClient, 
  BufferEncoders,
} from 'rsocket-core';
import RSocketTcpClient from 'rsocket-tcp-client';

// Create an instance of a client
const client = new RSocketClient({
  // note: default `serializers` is pass-through
  setup: {
    // ms btw sending keepalive to server
    keepAlive: 60000, 
    // ms timeout if no keepalive response
    lifetime: 180000, 
    // format of `data`
    dataMimeType: 'application/octet-stream', 
    // format of `metadata`
    metadataMimeType: 'application/octet-stream', 
  },
  // Transports default to sending/receiving strings:
  // Use BufferEncoders to enable binary
  transport: new RSocketTcpClient(
    {host: '127.0.0.1', port: 8080}, // options to Node.js net.connect()
    BufferEncoders,
  ),
});

// Open the connection
client.connect().subscribe({
  onComplete: socket => {
    // socket provides the rsocket interactions fire/forget, request/response,
    // request/stream, etc as well as methods to close the socket.

    socket.fireAndForget({
      data: new Buffer(...),
      metadata: null, // or new Buffer(...)
    });
  },
  onError: error => console.error(error),
  onSubscribe: cancel => {/* call cancel() to abort */}
});
```

## Next

See the [client API documentation](./02-client-api.md).
