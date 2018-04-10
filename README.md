# golang socket.io [![GoDoc](https://godoc.org/github.com/henvic/socketio?status.svg)](https://godoc.org/github.com/henvic/socketio)

golang socket.io is an implementation for the [socket.io](https://socket.io) protocol in Go. There is a lack of specification for the socket.io protocol, so reverse engineering is the easiest way to find out how it works.

**This is a work in progress. Many features, such as middleware and binary support, are missing.**

**golang socket.io is an adapted work from [github.com/graarh/golang-socketio](https://github.com/graarh/golang-socketio).**


## on "connection", "error", and "disconnection"
**Wait for the socket.io connection event before emitting messages or you risk losing them** due in an unpredictable fashion (due to concurrency: connection latency, server load, etc.).

To overcome this you should always use socketio.Connect instead of socketio.DialOnly (only exposed for debugging).

And before emitting a message on a namespace, you want to wait for the ready signal, like so:

```go
ctx, cancel := context.WithTimeout(context.Background(), 10 * time.Second)
defer cancel()

select {
	case <-ctx.Done():
		return ctx.Err()
	case <-exampleNamespace.Ready():
		// don't need to do anything
}

if err := exampleNamespace.Emit("fleet", 100); err != nil {
	return err
}
```

You probably want to use a `select` receiving a second channel, such as context.Done() to avoid program loop, leak memory, or both in case of failure on all non-trivial programs.

This is not necessarily on the default namespace, which is automatically ready.

## Connecting to a socket.io server with a custom namespace
You can connect to a namespace and start emitting messages to it with:

```go
c, err := socketio.Connect(u, websocket.NewTransport())

if err != nil {
	return err
}

// ...

exampleNamespace, err := c.Of("example")	

If err != nil {
	return err
}

<-exampleNamespace.Ready() // don't do this, use a select like shown above instead!

If err := exampleNamespace.Emit("list", "friends"); err != nil {
	return err
}
```

## Running the example

1. `npm install` to install the dependencies for the example server
2. `node server.js`
2. `go run example.go`

If you need to improve this library you should consider using these tools:

* [Charles Proxy](https://www.charlesproxy.com)
* [Wireshark WebSocket wiki page](https://wiki.wireshark.org/WebSocket)

This library is used by the [WeDeploy](https://wedeploy.com) Command-Line Interface tool to open a shell and execute commands on Docker container services.