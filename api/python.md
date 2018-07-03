# Python

This page contains a reference of all exported Kore functions to Python.

Kore must be compiled with **PYTHON=1** for this to work.

---

Before you write any Python code you must import the kore module:
```python
import kore
```

Note that the Python code is expected to run inside of the Kore worker
process, the kore module being imported is not available outside of
this worker process.

Like the C API Kore will pass an http\_request data structure to your
Python page handler. This data structure is of type **kore.http\_request**

## Getters

* host - The domain as a unicode string.
* agent - The user agent as a unicode string.
* path - The requested path as a unicode string.
* body - The entire incoming HTTP body as a PyBuffer.
* method - The requested method as an PyLong. (kore.HTTP\_METHOD\_GET, etc).
* body\_path - The path to the HTTP body on disk (if enabled).
* connection - The underlying client connection as a **kore.connection** object.

## Methods

---

# pgsql

### Synopsis

```python
result = await req.pgsql(db, query)
```

### Description

Performs an asynchronous PostgreSQL query.

| Parameter | Description |
| --- | --- |
| db | A previously configured database name. |
| query | The query to be performed. |

### Returns

The result of the query as a dictionary.

### Example

```python
async def myquery(req):
	result = await req.pgsql("db", "SELECT * FROM pg_delay(10)")
	req.response(200, json.dumps(result).encode("utf-8"))
```

---

# cookie

### Synopsis

```python
cookie = req.cookie(name)
```

### Description

Returns the cookie value for a given name.

| Parameter | Description |
| --- | --- |
| name | The name of the cookie to lookup. |

### Returns

Returns the value of the cookie as a unicode string or None if not found.

### Example

```python
def handler(req):
	cookie = req.cookie("my_session")
	if cookie != None:
		# use cookie value to do things.
```

---

# response

### Synopsis

```python
req.response(status, body)
```

### Description

Creates an HTTP response for the given HTTP request.

| Parameter | Description |
| --- | --- |
| status | The HTTP status code to for the response. |
| body | The HTTP body to be included in the response, this must be a binary buffer. |

### Returns

Nothing

### Example

```python
def handler(req):
	req.response(200, b'ok')
```

---

# argument

### Synopsis

```python
value = req.argument(name)
```

### Description

Looks up an HTTP parameter that was previously validated via a Kore
params block in the configuration.

| Parameter | Description |
| --- | --- |
| name | The name of the parameter to lookup. |

### Returns

The parameter as a unicode string or None if it was not found.

### Example

```python
def handler(req):
	id = req.argument("id")
	if id != None:
		# got an id from somewhere
```

---

# body\_read

### Synopsis

```python
length, chunk = req.body_read(length)
```

### Description

Reads up to *length* bytes from the HTTP body and returns the actual
bytes read and data in a tuple.

A returned length of 0 bytes indicates the end of the HTTP body.

| Parameter | Description |
| --- | --- |
| length | The number of bytes to read. |

### Returns

The length and data read.

### Example

```python
def handler(req):
	if req.method == kore.HTTP_METHOD_POST:
		try
			length, body = req.body_read(1024)
			# do stuff with the body.
		except:
			kore.log(kore.LOG_INFO, "some error occurred")
			req.response(500, b'')
```

---

# file\_lookup

### Synopsis

```python
file = req.file_lookup(name)
```

### Description

Lookup an uploaded file (via multipart/form-data).

| Parameter | Description |
| --- | --- |
| name | The name of the file that was uploaded. |

### Returns

A **kore.http\_file** object that can be used to read data from.

### Example

```python
def handler(req):
	if req.method == kore.HTTP_METHOD_POST:
		req.populate_multi()
		try
			file = req.file_lookup("myfile")
			length, data = file.read(1024)
			# do stuff with the data .
		except:
			kore.log(kore.LOG_INFO, "some error occurred")
			req.response(500, b'')
```

---

# populate\_get

### Synopsis

```python
req.populate_get()
```

### Description

Instructs Kore to go ahead and parse the incoming querystring and validate
parameters according to the configured params {} blocks in the configuration.

### Returns

Nothing

---

# populate\_post

### Synopsis

```python
req.populate_post()
```

### Description

Instructs Kore to go ahead and parse the incoming POST data which is of
content-type *application/x-www-form-urlencoded*  and validate
parameters according to the configured params {} blocks in the configuration.

### Returns

Nothing

---

# populate\_multipart

### Synopsis

```python
req.populate_multipart()
```

### Description

Instructs Kore to go ahead and parse the incoming body as content-type
multipart/form-data and validate parameters according to the configured
params {} blocks in the configuration.

### Returns

Nothing

---

# populate\_cookies

### Synopsis

```python
req.populate_cookies()
```

### Description

Instructs Kore to go ahead and parse the incoming cookie header (if any).

### Returns

Nothing

---

# request\_header

### Synopsis

```python
value = req.request_header(name)
```

### Description

Finds the incoming request header by name and returns it.

| Parameter | Description |
| --- | --- |
| name | The name of the header to lookup. |

### Returns

The value of the header as a unicode string or None if the header was not present.

### Example

```python
def myhandler(req):
	xrequest = req.request_header("x-request")
	if xrequest != None:
		req.response_header("x-response", xrequest)

	req.response(200, b'hello world)
```

---

# response\_header

### Synopsis

```python
req.response_header(name, value)
```

### Description

Adds the given header to the response that will be sent by req.response().

| Parameter | Description |
| --- | --- |
| name | The name of the header that will be added. |
| value | The value of the header that will be added. |

### Returns

Nothing

### Example

```python
def myhandler(req):
	xrequest = req.request_header("x-request")
	if xrequest != None:
		req.response_header("x-response", xrequest)

	req.response(200, b'hello world)
```

---

# websocket\_handshake

### Synopsis

```python
req.websocket_handshake(onconnect, onmsg, ondisconnect)
```

### Description

Adds the given header to the response that will be sent by req.response().

| Parameter | Description |
| --- | --- |
| onconnect | The name of the function to be called when a new websocket client is connected. |
| onmsg | The name of the function to be called when a websocket message arrives. |
| ondisconnect | The name of the function to be called when a new websocket client is removed. |

### Returns

Nothing

### Example

```python
def onconnect(c):
	kore.log(kore.LOG_INFO, "%s: connected" % c)

def onmessage(c, op, data):
	# data from c arrived
	# op is the websocket op, WEBSOCKET_OP_TEXT, WEBSOCKET_OP_BINARY
	# data is the data that arrived

def ondisconnect(c):
	kore.log(kore.LOG_INFO, "%s: disconnected" % c)

def ws_connect(req):
	req.websocket_handshake("onconnect", "onmsg", "ondisconnect")
```

---
