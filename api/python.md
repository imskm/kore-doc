# Python

This page contains a reference of all exported Kore functions to Python.

Kore must be compiled with **PYTHON=1** for this to work.
Python 3.6 or higher should be used.

---

# Pulling in python files
You can import python files into your Kore application using the
**python_import** configuration option:

```
python_import ./src/hello.py
```

# Intro
Before you write any Python code you must import the kore module:
```python
import kore
```

Note that the Python code is expected to run inside of the Kore worker
process, the kore module being imported is not available outside of
this worker process.

## Shortcuts

* [Kore module](#koremodule)
  * [constants](#koremoduleconstants)
  * [functions](#koremodulefunctions)
    * [log](#log)
    * [fatal](#fatal)
    * [register\_database](#registerdatabase)
    * [websocket\_broadcast](#websocketbroadcast)


* [Http module](#httpmodule)
  * [constants](#httpmoduleconstants)
  * [functions](#httpmodulefunctions)
    * [pgsql](#pgsql)
    * [cookie](#cookie)
    * [response](#response)
    * [argument](#argument)
    * [body\_read](#bodyread)
    * [file\_lookup](#filelookup)
    * [populate\_get](#populateget)
    * [populate\_post](#populatepost)
    * [populate\_cookies](#populatecookies)
    * [populate\_multipart](#populatemultipart)
    * [request\_header](#requestheader)
    * [response\_header](#responseheader)
    * [websocket\_handshake](#websockethandshake)

# <a name="koremodule"></a>Kore

## <a name="koremoduleconstants"></a>Constants

The kore module exports some constant that can be used by the Python code.

* LOG\_INFO
* LOG\_NOTICE
* RESULT\_OK
* RESULT\_RETRY
* RESULT\_ERROR
* MODULE\_LOAD
* MODULE\_UNLOAD
* CONN\_PROTO\_HTTP
* CONN\_PROTO\_UNKNOWN
* CONN\_PROTO\_WEBSOCKET
* CONN\_STATE\_ESTABLISHED

## <a name="koremodulefunctions"></a>Functions

---

# <a name="log"></a>log

### Synopsis

```python
kore.log(priority, string)
```

### Description

Logs the given string to syslog.

| Parameter | Description |
| --- | --- |
| priority | Log level priority (kore.LOG\_ constants). |
| string | The string to be sent to syslog. |

### Returns

Nothing

### Example

```python
kore.log(kore.LOG_INFO, 'this is a log string from a worker process')
```

---

# <a name="fatal"></a>fatal

### Synopsis

```python
kore.fatal(reason)
```

### Description

Terminates the worker process with the given reason as the error message.

| Parameter | Description |
| --- | --- |
| reason | String containing the reason why the worker is terminating. |

### Returns

Nothing

### Example

```python
kore.fatal('worker going dead')
```

---

# <a name="websocketbroadcast"></a>websocket\_broadcast

### Synopsis

```python
kore.websocket_broadcast(src, op, data, scope)
```

### Description

Broadcast a websocket message to all other connected websocket clients.

| Parameter | Description |
| --- | --- |
| src | The source **kore.connection** object. |
| op | The websocket op type. |
| data | The data to be broadcasted. |
| scope | Wether or not this is broadcasted to all workers or just this one. |

### Returns

Nothing

### Example

```python
def onmessage(c, op, data):
	kore.websocket_broadcast(c, op, data, kore.WEBSOCKET_BROADCAST_GLOBAL)
```

---

# <a name="registerdatabase"></a>register\_database

### Synopsis

```python
kore.register_database(name, connstring)
```

### Description

Associates a pgsql connection string with a shortname for a database that
can be used later in <a href="#pgsql">pgsql</a>.

| Parameter | Description |
| --- | --- |
| name | The friendly name for the database. |
| connstring | The pgsql connection string. |

### Returns

Nothing

### Example

```python
kore.register_database("db", "host=/tmp dbname=hello")
```

---


# <a name="httpmodule"></a>HTTP

Like the C API Kore will pass an http\_request data structure to your
Python page handler. This data structure is of type **kore.http\_request**

## <a name="httpmoduleconstants"></a>Constants

* HTTP\_METHOD\_GET
* HTTP\_METHOD\_PUT
* HTTP\_METHOD\_HEAD
* HTTP\_METHOD\_POST
* HTTP\_METHOD\_DELETE
* HTTP\_METHOD\_OPTIONS
* HTTP\_METHOD\_PATCH

## <a name="httpmodulegetters"></a>Getters

* host - The domain as a unicode string.
* agent - The user agent as a unicode string.
* path - The requested path as a unicode string.
* body - The entire incoming HTTP body as a PyBuffer.
* method - The requested method as an PyLong. (kore.HTTP\_METHOD\_GET, etc).
* body\_path - The path to the HTTP body on disk (if enabled).
* connection - The underlying client connection as a **kore.connection** object.

## <a name="httpmodulefunctions"></a>Functions

---

# <a name="pgsql"></a>pgsql

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

# <a name="cookie"></a>cookie

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

# <a name="response"></a>response

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

# <a name="argument"></a>argument

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

# <a name="bodyread"></a>body\_read

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

# <a name="filelookup"></a>file\_lookup

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

# <a name="populateget"></a>populate\_get

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

# <a name="populatepost"></a>populate\_post

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

# <a name="populatemultipart"></a>populate\_multipart

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

# <a name="populatecookies"></a>populate\_cookies

### Synopsis

```python
req.populate_cookies()
```

### Description

Instructs Kore to go ahead and parse the incoming cookie header (if any).

### Returns

Nothing

---

# <a name="requestheader"></a>request\_header

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

# <a name="responseheader"></a>response\_header

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

# <a name="websockethandshake"></a>websocket\_handshake

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
