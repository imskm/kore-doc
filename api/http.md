# HTTP

This page contains all available public functions related to understanding and responding to HTTP requests.

## Index

* [http\_response](#http_response)
* [http\_response\_header](#http_response_header)
* [http\_response\_stream](#http_response_stream)
* [http\_response\_fileref](#http_response_fileref)
* [http\_request\_header](#http_request_header)
* [http\_file\_lookup](#http_file_lookup)
* [http\_file\_read](#http_file_read)
* [http\_file\_rewind](#http_file_rewind)
* [http\_populate\_post](#http_populate_post)
* [http\_populate\_qs](#http_populate_qs)
* [http\_populate\_multipart\_form](#http_populate_multipart_form)
* [http\_body\_read](#http_body_read)
* [http\_state\_run](#http_state_run)
* [http\_state\_create](#http_state_create)
* [http\_state\_get](#http_state_get)
* [http\_state\_cleanup](#http_state_cleanup)
* [http\_status\_text](#http_status_text)
* [http\_method\_text](#http_method_text)
* [http\_argument\_get\_string](#http_argument_get_string)
* [http\_argument\_get\_byte](#http_argument_get_byte)
* [http\_argument\_get\_int16](#http_argument_get_int16)
* [http\_argument\_get\_uint16](#http_argument_get_uint16)
* [http\_argument\_get\_int32](#http_argument_get_int32)
* [http\_argument\_get\_uint32](#http_argument_get_uint32)
* [http\_argument\_get\_int64](#http_argument_get_int64)
* [http\_argument\_get\_uint64](#http_argument_get_uint64)
* [http\_argument\_get\_float](#http_argument_get_float)
* [http\_argument\_get\_double](#http_argument_get_double)

---

# http\_response {#http_response}

### Synopsis

```
void http_response(struct http_request *req, int status, const void *data, size_t length)
```

### Description

Creates an HTTP response for an HTTP request.

| Parameter | Description |
| --- | --- |
| req | The HTTP request to respond to. |
| status | The HTTP status code to include in the response. |
| data | The data to be sent in the response body. |
| length | The length of the data to be sent. |

### Returns

Nothing

---

# http\_response\_header {#http_response_header}

### Synopsis

```
void http_response_header(struct http_request *req, const char *header, const char *value)
```

### Description

Includes HTTP header into response.

| Parameter | Description |
| --- | --- |
| req | The HTTP request to respond to. |
| header | The HTTP header to include in the response. |
| value | The HTTP header value to include in the response. |

### Returns

Nothing

---

# http\_response\_stream {#http_response_stream}

### Synopsis

```
void http_response_stream(struct http_request *req, int status, void *base, size_t length, int (*cb)(struct netbuf *), void *arg)
```

### Description

Creates an HTTP response for an HTTP request much like _http\_response\(\)_.

However unlike that function this one does not copy the response body data but rather streams from it. It will call the given callback _cb_ when all data has been sent.

| Parameter | Description |
| --- | --- |
| req | The HTTP request to respond to. |
| status | The HTTP status code to include in the response. |
| base | The base pointer of the data to be sent in the response body. |
| length | The length of the data to be sent. |
| cb | A callback that is called when all data has been sent. |
| arg | A user supplied argument that is passed in the callback. |

### Returns

Nothing

---

# http\_response\_fileref {#http_response_fileref}

### Synopsis

```
void http_response_fileref(struct http_request *req, struct kore_fileref *ref)
```

### Description

Creates an HTTP response for an HTTP request much like _http\_response\(\)_.

This function however takes a kore filereference data structure as its argument
and will send the ondisk file it represents to the client.

| Parameter | Description |
| --- | --- |
| req | The HTTP request to respond to. |
| ref | The kore file reference. |

### Returns

Nothing

---

# http\_request\_header {#http_request_header}

### Synopsis

```
int http_request_header(struct http_request *req, const char *header, const char **out)
```

### Description

Attempts to find the given _header_ in an HTTP request and returns the value of the header in the _out_ parameter. The returned pointer should not be freed.

| Parameter | Description |
| --- | --- |
| req | The HTTP request. |
| header | The name of the header to find. |
| out | Pointer to where the pointer to the result is stored. |

### Returns

KORE\_RESULT\_OK if a result was set in _out_.
KORE\_RESULT\_ERROR if the header was not present in the request.

---

# http\_file\_lookup {#http_file_lookup}

### Synopsis

```
struct http_file *http_file_lookup(struct http_request *req, const char *name)
```

### Description

Lookup a file that was uploaded as part of a multipart form submission.

| Parameter | Description |
| --- | --- |
| req | The HTTP request. |
| name | The name of the form field the file was sent as. |

### Returns

A pointer to an _http\_file_ data structure containing information about the uploaded file.

Will return NULL if the file could not be found.

### Note

You must call _http\_populate\_multipart\_form\(\)_ before this function will return any result at all.

---

# http\_file\_read {#http_file_read}

### Synopsis

```
ssize_t http_file_read(struct http_file *file, void *buf, size_t length)
```

### Description

Read file data from the given file up to _length_ size.

| Parameter | Description |
| --- | --- |
| file | The file from which to read. |
| buf | Where the file data is copied into. |
| length | The maximum length that can be copied into _buf_. |

### Returns

Returns the number of bytes successfully read from the file, or 0 on end of file, or -1 on error.

---

# http\_file\_rewind {#http_file_rewind}

### Synopsis

```
void http_file_rewind(struct http_file *file)
```

### Description

Sets the offset member of the given _file_ data structure back to 0 for sequential reads.

| Parameter | Description |
| --- | --- |
| file | The file to rewind. |

### Returns

Nothing

---

# http\_populate\_post {#http_populate_post}

### Synopsis

```
void http_populate_post(struct http_request *req)
```

### Description

Processes an HTTP POST by taking the HTTP body and parsing it according to _application/x-www-form-urlencoded_.

This function will automatically match any fields found against configured validators to check if they contained sensible data. If the validators fail the field is automatically removed.

| Parameter | Description |
| --- | --- |
| req | The HTTP request to parse. |

### Returns

Nothing

---

# http\_populate\_qs {#http_populate_qs}

### Synopsis

```
void http_populate_qs(struct http_request *req)
```

### Description

The same as _http\_populate\_post\(\)_ but for query string parameters instead.

| Parameter | Description |
| --- | --- |
| req | The HTTP request to parse. |

### Returns

Nothing

---

# http\_populate\_multipart\_form {#http_populate_multipart_form}

### Synopsis

```
void http_populate_multipart_form(struct http_request *req)
```

### Description

Parses a multipart form that was received via a POST request.

| Parameter | Description |
| --- | --- |
| req | The HTTP request to parse. |

### Returns

Nothing

---

# http\_body\_read {#http_body_read}

### Synopsis

```
ssize_t http_body_read(struct http_request *req, void *out, size_t length)
```

### Description

Attempts to read data from the HTTP body received in a request.

| Parameter | Description |
| --- | --- |
| req | The HTTP request. |
| out | Where the data is copied into. |
| length | The maximum number of bytes that will fit in _out_. |

### Returns

Returns the number of bytes copied or 0 on end of body or -1 on error.

---

# http\_state\_run {#http_state_run}

### Synopsis

```
int http_state_run(struct http_state *states, u_int8_t elm, struct http_request *req)
```

### Description

Runs an HTTP state machine.

| Parameter | Description |
| --- | --- |
| states | The HTTP state machine to be run. |
| elm | The number of elements in this state machine. |
| req | The HTTP request to be passed to the state machine functions. |

### Returns

* KORE\_RESULT\_OK
* KORE\_RESULT\_ERROR
* KORE\_RESULT\_RETRY

### Note

This should be called from a page handler.

---

# http\_state\_create {#http_state_create}

### Synopsis

```
void *http_state_create(struct http_request *req, size_t len, void (*onfree)(struct http_request *))
```

### Description

Allocates a state attached to the HTTP request that the user can use to
store temporary data for the request.

| Parameter | Description |
| --- | --- |
| req | The HTTP request. |
| len | Size of the state context. |
| onfree | Callback function called when an HTTP request is being free'd. |

### Returns

The state context.

---

# http\_state\_get {#http_state_get}

### Synopsis

```
void *http_state_get(struct http_request *req)
```

### Description

Get the user-defined state attached to the HTTP request.

| Parameter | Description |
| --- | --- |
| req | The HTTP request. |

### Returns

The state context or NULL if none was set.

---

# http\_state\_cleanup {#http_state_cleanup}

### Synopsis

```
void http_state_cleanup(struct http_request *req)
```

### Description

Free the previously allocated user-defined state attached to the HTTP request.

| Parameter | Description |
| --- | --- |
| req | The HTTP request. |

### Returns

Nothing

---

# http\_status\_text {#http_status_text}

### Synopsis

```
const char *http_status_text(int status)
```

### Description

Returns a pointer to a human readable string for the given HTTP status code.

| Parameter | Description |
| --- | --- |
| status | The HTTP status code for which to find the string. |

### Returns

A pointer to a human readable string for the HTTP status code.

---

# http\_method\_text {#http_method_text}

### Synopsis

```
const char *http_method_text(int method)
```

### Description

Returns a pointer to a human readable string for the given HTTP method.

| Parameter | Description |
| --- | --- |
| method | The HTTP method for which to find the matching string. |

### Returns

A pointer to a human readable string for the HTTP method.

---

# http\_argument\_get\_string {#http_argument_get_string}

### Synopsis

```
int http_argument_get_string(struct http_request *req, const char *name, char **out)
```

### Description

Lookup an argument as a string. The caller should not free the result.

| Parameter | Description |
| --- | --- |
| req | The HTTP request. |
| name | The name of the argument to find. |
| out | Where the pointer to the result is stored. |

### Returns

KORE\_RESULT\_OK if the argument was found or KORE\_RESULT\_ERROR if it was not found.

---

# http\_argument\_get\_byte {#http_argument_get_byte}

### Synopsis

```
int http_argument_get_byte(struct http_request *req, const char *name, u_int8_t *out)
```

### Description

Lookup an argument as a byte.

| Parameter | Description |
| --- | --- |
| req | The HTTP request. |
| name | The name of the argument to find. |
| out | Where the result is stored. |

### Returns

KORE\_RESULT\_OK if the argument was found or KORE\_RESULT\_ERROR if it was not found.

---

# http\_argument\_get\_int16 {#http_argument_get_int16}

### Synopsis

```
int http_argument_get_int16(struct http_request *req, const char *name, int16_t *out)
```

### Description

Lookup an argument as a 16-bit signed integer. This function will check that the result fits in a 16-bit signed integer before returning it.

| Parameter | Description |
| --- | --- |
| req | The HTTP request. |
| name | The name of the argument to find. |
| out | Where the result is stored. |

### Returns

KORE\_RESULT\_OK if the argument was found or KORE\_RESULT\_ERROR if it was not found.

---

# http\_argument\_get\_uint16 {#http_argument_get_uint16}

### Synopsis

```
int http_argument_get_uint16(struct http_request *req, const char *name, uint16_t *out)
```

### Description

Lookup an argument as a 16-bit unsigned integer. This function will check that the result fits in a 16-bit unsigned integer before returning it.

| Parameter | Description |
| --- | --- |
| req | The HTTP request. |
| name | The name of the argument to find. |
| out | Where the result is stored. |

### Returns

KORE\_RESULT\_OK if the argument was found or KORE\_RESULT\_ERROR if it was not found.

---

# http\_argument\_get\_int32 {#http_argument_get_int32}

### Synopsis

```
int http_argument_get_int32(struct http_request *req, const char *name, int32_t *out)
```

### Description

Lookup an argument as a 32-bit signed integer. This function will check that the result fits in a 32-bit signed integer before returning it.

| Parameter | Description |
| --- | --- |
| req | The HTTP request. |
| name | The name of the argument to find. |
| out | Where the result is stored. |

### Returns

KORE\_RESULT\_OK if the argument was found or KORE\_RESULT\_ERROR if it was not found.

---

# http\_argument\_get\_uint32 {#http_argument_get_uint32}

### Synopsis

```
int http_argument_get_uint32(struct http_request *req, const char *name, uint32_t *out)
```

### Description

Lookup an argument as a 32-bit unsigned integer. This function will check that the result fits in a 32-bit unsigned integer before returning it.

| Parameter | Description |
| --- | --- |
| req | The HTTP request. |
| name | The name of the argument to find. |
| out | Where the result is stored. |

### Returns

KORE\_RESULT\_OK if the argument was found or KORE\_RESULT\_ERROR if it was not found.

---

# http\_argument\_get\_int64 {#http_argument_get_int64}

### Synopsis

```
int http_argument_get_int64(struct http_request *req, const char *name, int64_t *out)
```

### Description

Lookup an argument as a 64-bit signed integer. This function will check that the result fits in a 64-bit signed integer before returning it.

| Parameter | Description |
| --- | --- |
| req | The HTTP request. |
| name | The name of the argument to find. |
| out | Where the result is stored. |

### Returns

KORE\_RESULT\_OK if the argument was found or KORE\_RESULT\_ERROR if it was not found.

---

# http\_argument\_get\_uint64 {#http_argument_get_uint64}

### Synopsis

```
int http_argument_get_uint64(struct http_request *req, const char *name, uint64_t *out)
```

### Description

Lookup an argument as a 64-bit unsigned integer. This function will check that the result fits in a 64-bit unsigned integer before returning it.

| Parameter | Description |
| --- | --- |
| req | The HTTP request. |
| name | The name of the argument to find. |
| out | Where the result is stored. |

### Returns

KORE\_RESULT\_OK if the argument was found or KORE\_RESULT\_ERROR if it was not found.

---

# http\_argument\_get\_float {#http_argument_get_float}

### Synopsis

```
int http_argument_get_float(struct http_request *req, const char *name, float *out)
```

### Description

Lookup an argument as a float. This function will check that the result fits in a float before returning it.

| Parameter | Description |
| --- | --- |
| req | The HTTP request. |
| name | The name of the argument to find. |
| out | Where the result is stored. |

### Returns

KORE\_RESULT\_OK if the argument was found or KORE\_RESULT\_ERROR if it was not found.

---

# http\_argument\_get\_double {#http_argument_get_double}

### Synopsis

```
int http_argument_get_double(struct http_request *req, const char *name, double *out)
```

### Description

Lookup an argument as a double. This function will check that the result fits in a double before returning it.

| Parameter | Description |
| --- | --- |
| req | The HTTP request. |
| name | The name of the argument to find. |
| out | Where the result is stored. |

### Returns

KORE\_RESULT\_OK if the argument was found or KORE\_RESULT\_ERROR if it was not found.

---

# Data structures

### http\_request

### http\_file

### http\_header

### http\_arg

### http\_state

---



