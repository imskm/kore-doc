# Asynchronous libcurl support

Kore allows you to schedule CURL easy handles onto its internal event loop
allowing you to make asynchronous requests for anything libcurl supports.

On top of that this API provides higher level interface for making HTTP
client requests a bit easier.

Kore must be built with CURL=1 in order to use this API.

## Index

* [kore\_curl\_init](#init)
* [kore\_curl\_clean](#cleanup)
* [kore\_curl\_bind\_request](#bindrequest)
* [kore\_curl\_bind\_callback](#bindcallback)
* [kore\_curl\_run](#run)
* [kore\_curl\_success](#success)
* [kore\_curl\_logerror](#logerror)
* [kore\_curl\_response\_as\_bytes](#responsebytes)
* [kore\_curl\_response\_as\_string](#responsestring)

## Examples

See the included [example](https://github.com/jorisvink/kore/tree/master/examples/async-curl) in the Kore source tree for an implementation of this API.

---

# kore\_curl\_init {#init}
### Synopsis
```
void kore_curl_init(struct kore_curl *client, const char *url);
```
### Description
Initializes a **kore\_curl** context. This must be called before any other
function can be safely used.

| Parameter | Description |
| -- | -- |
| client | A kore\_curl data structure. |
| url | The URL to be associated with this context. |

### Returns
Nothing

---
# kore\_curl\_cleanup {#cleanup}
### Synopsis
```
void kore_curl_cleanup(struct kore_curl *client);
```
### Description
Cleanup the **kore\_curl** context and release all resources. This must be
called when you no longer need the context.

| Parameter | Description |
| -- | -- |
| client | A kore\_curl data structure. |

### Returns
Nothing

---
# kore\_curl\_bind\_request {#bindrequest}
### Synopsis
```
void kore_curl_bind_request(struct kore_curl *client, struct http_request *req);
```
### Description
Bind an HTTP request to a kore\_curl request. Binding means that the HTTP
request will be woken up by Kore once the curl request has a result.

Using this in combination with the HTTP state machine allows you to build
request handlers in a completely asynchronous fashion.

| Parameter | Description |
| -- | -- |
| client | A kore\_curl data structure. |
| req | The HTTP request to be bound to the task. |

### Returns
Nothing

---
# kore\_curl\_bind\_callback {#bindcallback}
### Synopsis
```
void kore_curl_bind_callback(struct kore_curl *client,
    void (*cb)(struct kore_curl *, void *), void *arg)
```
### Description
Bind a callback to a curl request. Much like the HTTP binding this will cause
Kore to call this callback once a curl request has a result.

You may **not** bind both a callback and an HTTP request.

| Parameter | Description |
| -- | -- |
| client | A kore\_curl data structure. |
| cb | The callback to call. |
| arg | User-supplied pointer that is passed to the callback. |

### Returns
Nothing

---
# kore\_curl\_run {#run}
### Synopsis
```
void kore_curl_run(struct kore_curl *client)
```
### Description
Schedules the kore\_curl context onto the event loop. After calling this
Kore will wake-up the bound HTTP request or call the bound callback once
there is a result.

| Parameter | Description |
| -- | -- |
| client | A kore\_curl data structure. |

### Returns
Nothing

---
# kore\_curl\_success {#success}
### Synopsis
```
int kore_curl_success(struct kore_curl *client)
```
### Description
Check if a curl request was successfull (checks if the curl handle is CURLE\_OK).

| Parameter | Description |
| -- | -- |
| client | A kore\_curl data structure. |

### Returns
Returns 1 if the curl handle its result value was CURLE\_OK or 0 otherwise.

---
# kore\_curl\_logerror {#logerror}
### Synopsis
```
void kore_curl_logerror(struct kore_task *client)
```
### Description
Log a notice with the error that the curl handle has returned.

| Parameter | Description |
| -- | -- |
| client | A kore\_curl data structure. |

### Returns
Nothing

---

# kore\_curl\_response\_as\_bytes {#responsebytes}
### Synopsis
```
void kore_curl_response_as_bytes(struct kore_curl *client, const u_int8_t **body, size_t *len)
```
### Description
Obtain the response from a curl request as plain bytes. The **bytes** and **len** pointers are populated to point to the response that was obtained.

| Parameter | Description |
| -- | -- |
| client | A kore\_curl data structure. |
| body | Pointer which will be pointed to the response bytes. |
| len | Pointer to a size\_t that will contain the length of the response. |

### Returns
Nothing

---

# kore\_curl\_response\_as\_string {#responsestring}
### Synopsis
```
char *kore_curl_response_as_string(struct kore_curl *client);
```
### Description
Obtain the response from a curl request as a pointer to a C string.

You must not free the returned pointer.

| Parameter | Description |
| -- | -- |
| client | A kore\_curl data structure. |

### Returns
A pointer to the response as a C string.

---
