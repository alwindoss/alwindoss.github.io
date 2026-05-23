+++
date = '2026-05-18T23:15:32+05:30'
draft = false
title = 'How to Use Elliptic Curve Diffie Hellman Library in Go'
categories = ["technology"] # <--- Add this
type = "technology"
+++
Go introduced a new library called [Elliptic Curve Diffie Hellman](https://en.wikipedia.org/wiki/Elliptic-curve_Diffie%E2%80%93Hellman)([crypto/ecdh](https://pkg.go.dev/crypto/ecdh)) in v1.20.

Let's see how to use this library to exchange encrypted data between two entities without sharing the secret that was used to encrypt the data.

### Generate public key on the client

```go
clientCurve := ecdh.P256()
clientPrivKey, err := clientCurve.GenerateKey(rand.Reader)
if err != nil {
	t.Fatalf("Error: %v", err)
}
clientPubKey := clientPrivKey.PublicKey()
```

### Generate public key on the server

```go
serverCurve := ecdh.P256()
serverPrivKey, err := serverCurve.GenerateKey(rand.Reader)
if err != nil {
	t.Fatalf("Error: %v", err)
}
serverPubKey := serverPrivKey.PublicKey()
```

The `clientPubkey` and `serverPubKey` can be shared over the network as plain text.

### Generate Secret using the public key

Client generates the `clientSecret` using the server's public key.

```go
clientSecret, err := clientPrivKey.ECDH(serverPubKey)
if err != nil {
	t.Fatalf("Error: %v", err)
}
```

Server generates the `serverSecret` using the client's public key

```go
serverSecret, err := serverPrivKey.ECDH(clientPubKey)
if err != nil {
	t.Fatalf("Error: %v", err)
}
```

### Sharing Encrypted Data

Using the `clientSecret` the client can encrypt data and share over the network while the server will be able to decrypt the data using the `serverSecret`.

What was wonderful about this is the fact that the secret in itself was not shared over the network.

### Conclusion

Diffie-Hellman key exchange algorithm is used more often than we realize. It is a very nice addition to the Go standard library.

### Full Code

```go
package main

import (
	"bytes"
	"crypto/ecdh"
	"crypto/rand"
	"log"
)

func main() {

	clientCurve := ecdh.P256()
	clientPrivKey, err := clientCurve.GenerateKey(rand.Reader)
	if err != nil {
		log.Fatalf("Error: %v", err)
	}
	clientPubKey := clientPrivKey.PublicKey()

	serverCurve := ecdh.P256()
	serverPrivKey, err := serverCurve.GenerateKey(rand.Reader)
	if err != nil {
		log.Fatalf("Error: %v", err)
	}
	serverPubKey := serverPrivKey.PublicKey()

	clientSecret, err := clientPrivKey.ECDH(serverPubKey)
	if err != nil {
		log.Fatalf("Error: %v", err)
	}
	serverSecret, err := serverPrivKey.ECDH(clientPubKey)
	if err != nil {
		log.Fatalf("Error: %v", err)
	}
	if !bytes.Equal(clientSecret, serverSecret) {
		log.Fatalf("The secrets do not match")
	}
	log.Printf("The secrets match")
}
```