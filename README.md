# monte

[![MIT License](https://img.shields.io/apm/l/atomic-design-ui.svg?)](LICENSE)
[![go.dev reference](https://img.shields.io/badge/go.dev-reference-007d9c?logo=go&logoColor=white&style=flat-square)](https://pkg.go.dev/github.com/lithdew/monte)
[![Discord Chat](https://img.shields.io/discord/697002823123992617)](https://discord.gg/HZEbkeQ)

The bare minimum for high performance, fully-encrypted RPC over TCP in Go.

## Features

1. Send requests, receive responses, or send messages with no expectations for responses.
2. Send from 50MiB/s to 1500MiB/s, with zero to two allocations max per sent message or RPC call.
3. Gracefully establish multiple client connections to a single endpoint up to a configurable limit.
4. Set the total number of connections that may concurrently be accepted and handled by a single endpoint.
5. Configure read/write timeouts, dial timeouts, handshake timeouts, or customize the handshaking protocol.
6. All messages, once the handshake protocol is complete, are encrypted and non-distinguishable from each other.
7. Supports graceful shutdowns for both client and server, with extensive tests for highly-concurrent scenarios.

## Protocol

### Handshake

1. Send X25519 curve point (32 bytes) to peer.
2. Receive X25519 curve point (32 bytes) from our peer.
3. Multiply X25519 curve scalar with X25519 curve point received from our peer.
4. Derive a shared key by using BLAKE-2b as a key derivation function over our scalar point multiplication result.
5. Encrypt further communication with AES 256-bit GCM using our shared key, with a nonce counter increasing for every
incoming/outgoing message.

### Message Format

1. Encrypted messages are prefixed with an unsigned 32-bit integer denoting the message's length.
2. The decoded message content is prefixed with an unsigned 32-bit integer designating a sequence number.
3. The sequence number is used as an identifier to identify requests/responses from one another.
4. The sequence number 0 is reserved for requests that do not expect a response.

## Benchmarks

```
$ cat /proc/cpuinfo | grep 'model name' | uniq
model name : Intel(R) Core(TM) i7-7700HQ CPU @ 2.80GHz

$ go test -bench=. -benchtime=10s
goos: linux
goarch: amd64
pkg: github.com/lithdew/monte
BenchmarkSend-8                          1643733              7082 ns/op         197.70 MB/s         123 B/op          1 allocs/op
BenchmarkSendNoWait-8                   14516896               913 ns/op        1533.80 MB/s         152 B/op          0 allocs/op
BenchmarkRequest-8                        424249             28276 ns/op          49.51 MB/s         156 B/op          2 allocs/op
BenchmarkParallelSend-8                  5316900              2450 ns/op         571.40 MB/s         124 B/op          1 allocs/op
BenchmarkParallelSendNoWait-8           11475540              1072 ns/op        1305.66 MB/s         154 B/op          0 allocs/op
BenchmarkParallelRequest-8               1384652              7824 ns/op         178.93 MB/s         156 B/op          2 allocs/op
```