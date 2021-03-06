hypercore-xsalsa20-onwrite-hook
===============================

> A write hook to decrypt data using a xsalsa20 cipher into a Hypercore
> storage when replicating from peers.

## Installation

```sh
$ npm install hypercore-xsalsa20-onwrite-hook
```

## Usage

```js
const onwrite = hook(sharedSecret)
```

## Example

```js
const replicate = require('hypercore-replicate')
const hypercore = require('hypercore')
const crypto = require('hypercore-crypto')
const pump = require('pump')
const hook = require('hypercore-xsalsa20-onwrite-hook')
const ram = require('random-access-memory')

const key = crypto.randomBytes(32)

const { publicKey, secretKey } = crypto.keyPair()

const source = hypercore(ram, publicKey, {
  secretKey,
})

const cipher = hypercore(ram, publicKey, {
  secretKey,
  onwrite: hook(key)
})

const edge = hypercore(ram, publicKey)

const reader = hypercore(ram, publicKey, {
  onwrite: hook(key)
})

source.append(Buffer.from('hello'), (err) => {
  source.head(console.log) // plaintext

  // load cipher hypercore
  pump(source.createReadStream(), cipher.createWriteStream(), (err) => {
    if (err) throw err
    cipher.head(console.log) // ciphertext

    replicate(cipher, edge, (err) => {
      if (err) throw err
      edge.head(console.log) // ciphertext

      replicate(edge, reader, (err) => {
        if (err) throw err
        reader.head(console.log) // plaintext
      })
    })
  })
})
```

## API

### `const onwrite = hook(nonceStorage, sharedKey)`

Creates a `onwrite()` hook for a Hypercore feed that uses the
xsalsa20 cipher to encipher or decipher blocks in a Hypercore feed.
Nonces are computed from the Hypercore feed's public key and block
index.

## License

MIT
