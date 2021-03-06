# erebos

JavaScript client for [Swarm](https://github.com/ethersphere/go-ethereum) and
[PSS](https://github.com/ethersphere/go-ethereum/blob/pss/swarm/pss/README.md#postal-services-over-swarm).

## Disclaimer

This library is a work-in-progress client for Swarm and PSS, themselves still at
a proof-of-concept stage. It is intended for demonstration purposes only.\
APIs are likely to be changed and even removed between releases without prior notice.

At the moment this library may be only compatible with Swarm builds using
[the `pss-apimsg-hex` branch of `MainframeHQ/go-ethereum`](https://github.com/MainframeHQ/go-ethereum/tree/pss-apimsg-hex).

## Installation

```sh
yarn add erebos
```

## Example

Note: this is simplified code, it will not run as-is. See the
[example file](example.js) for a running example using a recent version of Node
(8+).

```js
import { createPSSWebSocket, decodeHex, encodeHex } from 'erebos'

// Open WebSocket connections to the PSS nodes
const [alice, bob] = await Promise.all([
  createPSSWebSocket('ws://localhost:8501'),
  createPSSWebSocket('ws://localhost:8502'),
])
// Retrieve Alice's public key and create the topic
const [key, topic] = await Promise.all([
  alice.getPublicKey(),
  alice.stringToTopic('PSS rocks'),
])
// Make Alice subscribe to the created topic and Bob add her public key
const [subscription] = await Promise.all([
  alice.subscribeTopic(topic),
  bob.setPeerPublicKey(key, topic),
])
// Actually subscribe to the messages stream
alice.createSubscription(subscription).subscribe(payload => {
  const msg = decodeHex(payload.Msg)
  console.log(`received message from ${payload.Key}: ${msg}`)
})
// Send message to Alice
bob.sendAsym(key, topic, encodeHex('hello world'))
```

## Types

* `hex = string`: hexadecimal-encoded string prefixed with `0x`

## API

### BZZ(url: string)

Creates a BZZ instance using the server provided as `url`.

#### BZZ.uploadRaw(data: string | Buffer, headers?: Object = {}): Promise

Uploads the provided `data` to the `bzzr:` endpoint and returns the created
hash.\
The `content-length` header will be added to the `headers` Object based on the `data`.

#### BZZ.downloadRaw(hash: string): Promise

Downloads the file matching the provided `hash` using the `bzzr:` endpoint.

#### BZZ.downloadRawBlob(hash: string): Promise

**Browser only - not available when using Node**\
Downloads the file matching the provided `hash` as a [`Blob`](https://developer.mozilla.org/en-US/docs/Web/API/Blob).

#### BZZ.downloadRawBuffer(hash: string): Promise

**Node only - not available in browser**\
Downloads the file matching the provided `hash` as a [`Buffer`](https://nodejs.org/dist/latest-v9.x/docs/api/buffer.html#buffer_buffer).

#### BZZ.downloadRawText(hash: string): Promise

Downloads the file matching the provided `hash` as a string.

### createPSSWebSocket(url: string): PSS

Shortcut for creating a PSS instance using a WebSocket transport connecting to
the provided URL.

### PSS(rpc: RPC)

Creates a PSS instance with the provided `RPC` instance.

#### PSS.getBaseAddr(): Promise

Calls
[`pss_baseAddr`](https://github.com/ethersphere/go-ethereum/blob/pss/swarm/pss/README.md#pss_baseaddr).

#### PSS.getPublicKey(): Promise

Calls
[`pss_getPublicKey`](https://github.com/ethersphere/go-ethereum/blob/pss/swarm/pss/README.md#pss_getpublickey).

#### PSS.sendAsym(key: hex, topic: hex, message: hex): Promise

Calls
[`pss_sendAsym`](https://github.com/ethersphere/go-ethereum/blob/pss/swarm/pss/README.md#pss_sendasym)
with the provided arguments:

* `key: hex`: public key of the peer
* `topic: hex`: destination topic
* `message: hex`

#### PSS.sendSym(keyID: string, topic: hex, message: hex): Promise

Calls
[`pss_sendSym`](https://github.com/ethersphere/go-ethereum/blob/pss/swarm/pss/README.md#pss_sendsym)
with the provided arguments:

* `keyID: hex`: symmetric key ID generated using `setSymmetricKey()`
* `topic: hex`: destination topic
* `message: hex`

#### PSS.setPeerPublicKey(key: hex, topic: hex, address?: hex = '0x'): Promise

Calls
[`pss_setPeerPublicKey`](https://github.com/ethersphere/go-ethereum/blob/pss/swarm/pss/README.md#pss_setpeerpublickey)
with the provided arguments:

* `key: hex`: public key of the peer
* `topic: hex`
* `address: hex`

#### PSS.setSymmetricKey(key: hex, topic: hex, address?: hex = '0x', useForDecryption: boolean = false): Promise

Calls
[`pss_setSymmetricKey`](https://github.com/ethersphere/go-ethereum/blob/pss/swarm/pss/README.md#pss_setsymmetrickey)
with the provided arguments:

* `key: hex`: public key of the peer
* `topic: hex`
* `address: hex`

Returns the generated symmetric key ID as a `string`.

#### PSS.stringToTopic(str: string): Promise

Calls
[`pss_stringToTopic`](https://github.com/ethersphere/go-ethereum/blob/pss/swarm/pss/README.md#pss_stringtotopic)
with the provided string and returns the generated topic.

#### PSS.subscribeTopic(topic: hex): Promise

Calls
[`pss_subscribe`](https://github.com/ethersphere/go-ethereum/blob/pss/swarm/pss/README.md#pss_subscribe)
with the provided topic and returns the subscription handle.

#### PSS.createSubscription(subscription: hex): Observable

Creates a
[RxJS Observable](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html)
that will emit events matching the provided subscription handle as created by
calling `subscribeTopic()` once subscribed to.

As documented in
[PSS API](https://github.com/ethersphere/go-ethereum/blob/pss/swarm/pss/README.md#pss_subscribe),
the received events will be objects with the following shape:

```js
{
  Msg: hex,
  Asymmetric: boolean,
  Key: string,
}
```

#### PSS.createTopicSubscription(topic: hex): Promise

Shortcut for calling `subscribeTopic()` followed by `createSubscription()`.

### RPC(transport: Subject)

Creates a RPC instance over the provided
[RxJS Subject](http://reactivex.io/rxjs/class/es6/Subject.js~Subject.html) and
starts subscribing to it.

#### RPC.connect()

Subscribes to the transport.

#### RPC.observe(method: string, params?: Array): Observable

Creates a
[RxJS Observable](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html)
that will call the `method` with the provided `params` once subscribed to.

#### RPC.promise(method: string, params?: Array): Promise

Calls the `method` with the provided `params` and provides the resulting
Promise.

#### RPC.subscribe(destinationOrNext: Observer | (any) => void, error?: (any) => void, complete?: () => void): () => void

Creates a
[RxJS Subscriber](http://reactivex.io/rxjs/class/es6/Subscriber.js~Subscriber.html)
for incoming events over the transport with the provided arguments and returns
the unsubscribe function.

### Utils

The following functions are exposed as part of the public API, they are useful
to make conversions between the values received and the parameters required by
the PSS API.

### encodeHex(input: string | byteArray, from: string = 'utf8'): hex

Creates a buffer calling
[`Buffer.from()`](https://nodejs.org/dist/latest-v9.x/docs/api/buffer.html#buffer_buffer_from_buffer_alloc_and_buffer_allocunsafe)
and converts it to a hexadecimal string prefixed by `0x`.

### decodeHex(input: hex, to: string = 'utf8'): string

Creates a buffer calling
[`Buffer.from()`](https://nodejs.org/dist/latest-v9.x/docs/api/buffer.html#buffer_buffer_from_buffer_alloc_and_buffer_allocunsafe)
after removing the `0x` prefix from the input and converts it to the destination
encoding.

## License

MIT.\
See [LICENSE](LICENSE) file.
