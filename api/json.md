# JSON parser

Kore provides a built-in JSON parser for your applications.

## Index

* [kore\_json\_init](#init)
* [kore\_json\_cleanup](#cleanup)
* [kore\_json\_parse](#parse)
* [kore\_json\_find\_object](#findobject)
* [kore\_json\_find\_array](#findarray)
* [kore\_json\_find\_string](#findstring)
* [kore\_json\_find\_number](#findnumber)
* [kore\_json\_find\_literal](#findliteral)

* [kore\_json\_create\_object](#createobject)
* [kore\_json\_create\_array](#createarray)
* [kore\_json\_create\_string](#createstring)
* [kore\_json\_create\_number](#createnumber)
* [kore\_json\_create\_literal](#createliteral)

* [kore\_json\_strerror](#strerror)

## Examples

See the included [example](https://github.com/jorisvink/kore/tree/master/examples/json) in the Kore source tree for an example on how to use this API.

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

### Settings {#settings}

By default this function will set the following libcurl options:
(some of these are **required** by the Kore curl API to function, it is
advised you do not override these).

- CURLOPT\_NOSIGNAL is set to 1.
- CURLOPT\_URL is set to the URL passed.
- CURLOPT\_USERAGENT is set to "kore/version".
- CURLOPT\_PRIVATE is set to the kore\_client data structure.
- CURLOPT\_WRITEFUNCTION is set to the kore\_curl\_tobuf function.
- CURLOPT\_ERRORBUFFER is set to point to the internal error buffer.
- CURLOPT\_TIMEOUT is set to the configuration option value curl\_timeout.
- CURLOPT\_WRITEDATA is set to the response kore buffer of the data structure.

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
Check if a curl request was successful (checks if the curl handle is CURLE\_OK).

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
# kore\_curl\_strerror {#strerror}
### Synopsis
```
const char *kore_curl_strerror(struct kore_task *client)
```
### Description
Returns a pointer to a human readable error string set by libcurl.

| Parameter | Description |
| -- | -- |
| client | A kore\_curl data structure. |

### Returns
A pointer to a human readable error string.

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

# kore\_curl\_http\_setup {#httpsetup}
### Synopsis
```
void kore_curl_http_setup(struct kore_curl *client, int method, const void *data, size_t len);
```
### Description
Setup an HTTP client request. You must have called [kore\_curl\_init()](#init) first.

| Parameter | Description |
| -- | -- |
| client | A kore\_curl data structure. |
| method | The HTTP method to be used, see [below](#httpmethod) |
| data | Pointer to the HTTP body to be sent (may be NULL). |
| len | The length of the HTTP body data (may be 0). |

### HTTP methods {#httpmethod}

Supported HTTP methods are:

- HTTP\_METHOD\_GET
- HTTP\_METHOD\_PUT
- HTTP\_METHOD\_HEAD
- HTTP\_METHOD\_POST
- HTTP\_METHOD\_PATCH
- HTTP\_METHOD\_DELETE
- HTTP\_METHOD\_OPTIONS

### Returns
Nothing

---

# kore\_curl\_http\_set\_header {#setheader}
### Synopsis
```
void kore_curl_http_set_header(struct kore_curl *client, const char *header. const char *value);
```
### Description
Add an HTTP header to a configured HTTP client request.

If value is NULL or an empty string the header is removed.

| Parameter | Description |
| -- | -- |
| client | A kore\_curl data structure. |
| header | The HTTP header name. |
| value | The HTTP header value. |

### Returns
Nothing

---

# kore\_curl\_http\_get\_header {#getheader}
### Synopsis
```
int kore_curl_http_get_header(struct kore_curl *client, const char *header. const char **out);
```
### Description
Obtain an HTTP header from the response (if [kore\_curl\_success()](#success) was 1).

| Parameter | Description |
| -- | -- |
| client | A kore\_curl data structure. |
| header | The HTTP header name. |
| out | A pointer which will receive the pointer to the HTTP header value. |

### Returns

Returns KORE\_RESULT\_OK if the header was found, or KORE\_RESULT\_ERROR if not.

---
