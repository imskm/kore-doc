# Websockets

Kore provides an easy to use interface for websockets. Once a connection has been upgraded to a websocket it will use user given callbacks for incoming websocket messages.

The prototypes for the callbacks are as follows:

```
void onconnect(struct connection *c);
void onmessage(struct connection *c, u_int8_t op, const void *data, size_t length);
void ondisconnect(struct connection *);
```

## Index

* [kore\_websocket\_handshake](#handshake)
* [kore\_websocket\_send](#send)
* [kore\_websocket\_broadcast](#broadcast)

---

# kore\_websocket\_handshake {#handshake}

### Synopsis

```
void kore_websocket_handshake(struct http_request *req, const char *onconnect,
    const char *onmessage, const char *ondisconnect)
```

### Description

Performs an upgrade of an HTTP connection to a websocket connection.

| Parameter | Description |
| --- | --- |
| req | The HTTP request. |
| onconnect | Name of the function that will be called when a new websocket client connects. |
| onmessage | Name of the function that will be called when a new websocket message arrives. |
| ondisconnect | Name of the function that will be called when a websocket client disconnects. |

### Returns

Nothing

---

# kore\_websocket\_send {#send}

### Synopsis

```
void kore_websocket_send(struct connection *c, u_int8_t op, const void *data, size_t length)
```

### Description

Sends a websocket message to a client.

| Parameter | Description |
| --- | --- |
| c | A client connection that has upgraded to websockets. |
| op | The websocket op type. |
| data | The data to be sent. |
| length | The length of the data to be sent. |

| Websocket ops |
| --- |
| WEBSOCKET\_OP\_TEXT |
| WEBSOCKET\_OP\_BINARY |

### Returns

Nothing

---

# kore\_websocket\_broadcast {#broadcast}

### Synopsis

```
void kore_websocket_broadcast(struct connection *src, u_int8_t op, const void *data, size_t length, int scope)
```

### Description

Broadcasts data to all connected websocket clients.

| Parameter | Description |
| --- | --- |
| src | Source client connection. |
| op | The websocket op type. |
| data | The data to be sent. |
| length | The length of the data to be sent. |
| scope | The scope of the broadcast \(worker only or all workers\). |

| scope |
| --- |
| WEBSOCKET\_BROADCAST\_LOCAL |
| WEBSOCKET\_BROADCAST\_GLOBAL |

### Returns

Nothing

---



