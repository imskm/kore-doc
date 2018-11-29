# Python

This page contains a reference of all exported Kore functions to Python.

Kore must be compiled with **PYTHON=1** for this to work.
Python 3.6 or higher should be used.

---

# Pulling in Python files
You can import Python files into your Kore application using the
**python_import** configuration option:

```
python_import ./src/hello.py
```

If you would like to add a search path for modules (PYTHONPATH) you can
specify this using the **python\_path** configuration option:

```
python_path /var/site-packages/mymodules/
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
    * [timer](#timer)
    * [proc](#proc)
    * [fatal](#fatal)
    * [tracer](#tracer)
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


* [Asynchronous functions](#async)


# <a name="koremodule"></a>Kore

## <a name="koremoduleconstants"></a>Constants

The kore module exports some constants that can be used by the Python code.

* LOG\_INFO
* LOG\_NOTICE
* RESULT\_OK
* RESULT\_RETRY
* RESULT\_ERROR
* MODULE\_LOAD
* MODULE\_UNLOAD
* TIMER\_ONESHOT
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

# <a name="timer"></a>timer

### Synopsis

```python
kore.timer(method, delay, flags)
```

### Description

Creates a timer that will run after **delay** milliseconds and call the
**method** specified when expires.

A timer can be primed to run at intervals or as a one-shot occasion.

For a intervals, **flags** is 0 while for one-shot **flags** should
be **kore.TIMER\_ONESHOT**

A kore.timer object may be stopped by calling the **close** method on it.

| Parameter | Description |
| --- | --- |
| method | The method to call when the timer fires. |
| delay | The delay in milliseconds before the timer fires. |
| flags | Either 0 for an interval timer or kore.TIMER_ONESHOT for a one-shot timer. |

### Returns

A kore.timer object.

### Example

```python
def callback():
	print("called")

mytimer = kore.timer(callback, 100, kore.TIMER\_ONESHOT)

# Stop the timer.
mytimer.close()
```

---

# <a name="proc"></a>proc

### Synopsis

```python
kore.proc(command, optional_timeout)
```

### Description

Spawns a new process with the given **command** and optional **timeout** in
milliseconds.

If timeout is non-zero Kore will raise TimeoutError if the process has not
exited before the timeout expires.

If a TimeoutError exception occurs the process is still active and **must**
be killed by calling the **kill** method on the process object.

See the [asynchronous process](#asyncproc) section for more.

| Parameter | Description |
| --- | --- |
| command | The command to be executed. |
| timeout | Optional timeout in milliseconds. |

### Returns

A kore.proc object.

### Example

```python
async def request(req):
	proc = kore.proc("ls -l /tmp")

	try:
		stdout = proc.recv(8192)
		retcode = proc.reap()
		req.response(200, stdout)
	except TimeoutError:
		proc.kill()
		print("process timed out")
		req.response(500, b'')
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

# <a name="tracer"></a>tracer

### Synopsis

```python
kore.tracer(method)
```

### Description

Sets the callback Kore will call for any uncaught exceptions.

The callback will get 3 parameters:
* The exception type
* The exception value
* The traceback

| Parameter | Description |
| --- | --- |
| method | The method to call for uncaught exceptions. |

### Returns

Nothing

### Example

```python
import traceback

def tracer(etype, value, tb):
	traceback.print_exception(etype, value, tb)

def kore_parent_configure(args):
	kore.tracer(tracer)
```

---

# <a name="websocketbroadcast"></a>websocket\_broadcast

### Synopsis

```python
kore.websocket_broadcast(src, op, data, scope)
```

### Description

Broadcasts a websocket message to all other connected websocket clients.

| Parameter | Description |
| --- | --- |
| src | The source **kore.connection** object. |
| op | The websocket op type. |
| data | The data to be broadcasted. |
| scope | Whether or not this is broadcasted to all workers or just this one. |

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
Python page handler. This data structure is of type **kore.http\_request**.

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
* method - The requested method as a PyLong. (kore.HTTP\_METHOD\_GET, etc).
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
| status | The HTTP status code to include in the response. |
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


# <a name="async"></a>Async/await

Kore exposes several functions that can be awaited upon allowing you to
write concurrent page handlers without any callbacks.

Kore currently supports:

  * [Asynchronous socket i/o](#asyncsocket)
  * [Locks](#asynclock)
  * [Queues](#asyncqueue)
  * [Processes](#asyncproc)
  * [Gathering](#asyncgather)

For example code covering all asynchronous features, please see the
[python-async](https://github.com/jorisvink/kore/tree/master/examples/python-async) example.

## <a name="asyncsocket"></a>Asynchronous sockets

You can use an existing Python socket object and wrap it inside of a Kore
socket. When you have a wrapped socket you can connect/accept, send or recv
asynchronously.

* [recv](#socketrecv)
* [send](#socketsend)
* [accept](#socketaccept)
* [connect](#socketconnect)
* [close](#socketclose)

Example:

```python
import kore
import socket

async def page(req):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    # Wrap the socket in a Kore object.
    ksock = kore.socket_wrap(sock)

    # Connect and send some data.
    await ksock.connect("127.0.0.1", 3332)
    await ksock.send("hello world")

    # Read a response.
    response = await ksock.recv(1024)

    # Closes the underlying socket automatically.
    ksock.close()
```

---

### <a name="socketrecv"></a>sock.recv

### Synopsis

```python
data = await sock.recv(1024)
```

### Description

Reads up to **maxlen** bytes from the socket.

| Parameter | Description |
| --- | --- |
| maxlen | Maximum number of bytes to be read. |

### Returns

The bytes read as a byte object or None on EOF.

### Example

```python
bytes = await sock.recv(1024)
if bytes is None:
    printf("eof!")
else:
    print("received %s" % bytes)
```

---

### <a name="socketsend"></a>sock.send

### Synopsis

```python
await sock.send(data)
```

### Description

Sends the given data over the socket.

| Parameter | Description |
| --- | --- |
| data | The data to be sent (as bytes). |

### Returns

Nothing. In case the peer closes the socket before all data is written
a RuntimeException is thrown.

### Example

```python
# Will return when all bytes are sent.
await sock.send("Hello world")
```

---

### <a name="socketaccept"></a>sock.accept

### Synopsis

```python
await sock.accept()
```

### Description

Accepts a new connection on the socket.

### Returns

A kore.socket data object that can be used to send/recv data on.

### Example

```python
conn = await sock.accept()
await conn.send("welcome")
conn.close()
```

---

### <a name="#socketconnect"></a>sock.connect

### Synopsis

```python
await sock.connect()
```

### Description

Connects to a peer.

### Returns

Nothing.

### Example

```python
# AF_INET socket.
await sock.connect("127.0.0.1", 8888)
data = await sock.recv(1024)
sock.close()

# AF_UNIX socket.
await sock.connect("/var/run/socket.sock")
data = await sock.recv(1024)
sock.close()

```

---

## <a name="asynclock"></a>Locks

Kore provides a locking mechanism where multiple coroutines can wait
upon a lock in a non-blocking way. This allows synchronization between
coroutines in a straight forward way.

A lock is created by calling **kore.lock()**.

An coroutine can use the lock with the Python "async with" syntax.

Example:

```python
# Create the lock
lock = kore.lock()

async def request(req):
	# Only allow one request at a time to this handler.
	async with lock:
		# do things
		req.response(200, b'ok')
```

---

## <a name="asyncqueue"></a>Queues

Kore provides a queue system that allows objects to be sent back and forth
between coroutines in a non-blocking fashion.

The queue is a FIFO queue.

* [push](#queuepush)
* [pop](#queuepop)

---

### <a name="queuepush"></a>queue.push

### Synopsis

```python
queue.push(object)
```

### Description

Pushes the given **object** onto the queue. The coroutine that waited first
on the queue using **pop** will be woken up.

| Parameter | Description |
| --- | --- |
| object | The object to be pushed onto the queue. |

### Returns

Nothing.

### Example

```python
queue = kore.queue()

d = {
    "foo": "bar"
}

queue.push(d)
```

---

### <a name="queuepop"></a>queue.pop

### Synopsis

```python
object = await queue.pop()
```

### Description

Waits for an object to be pushed onto the queue and will return that object.

### Returns

The object that was received.

### Example

```python
d = await queue.pop()
```

---

## <a name="asyncproc"></a>Asynchronous processes

For a more detailed example, see [this](https://git.kore.io/kore/file/examples/python-async/src/async_process.py) source file.

Or see the [kore.proc](https://docs.kore.io/3.2.0/api/python.html#proc) description.

* [kill](#prockill)
* [reap](#procreap)
* [recv](#procrecv)
* [send](#procsend)
* [close\_stdin](#procclosestdin)

---

### <a name="prockill"></a>proc.kill

### Synopsis

```python
proc.kill()
```

### Description

Sends an immediate SIGKILL to the process.

### Returns

Nothing

---

### <a name="procreap"></a>proc.reap

### Synopsis

```python
retcode = proc.reap()
```

### Description

Reaps the process and captures its exit code as the return value.

### Returns

The process exit code.

---

### <a name="procrecv"></a>proc.recv

### Synopsis

```python
data = await proc.recv(1024)
```

### Description

Reads up to **maxlen** bytes from the stdout pipe of the process.

| Parameter | Description |
| --- | --- |
| maxlen | Maximum number of bytes to be read. |

### Returns

The bytes read as a byte object or None on EOF.

---

### <a name="procsend"></a>proc.send

### Synopsis

```python
await proc.send(data)
```

### Description

Sends the given data to the process its stdin.

| Parameter | Description |
| --- | --- |
| data | The data to be sent (as bytes). |

### Returns

Nothing. In case the process closes its stdin before all data is written
a RuntimeException is thrown.

---

### <a name="procclosestdin"></a>proc.close\_stdin

### Synopsis

```python
proc.close_stdin()
```

### Description

Closes the write pipe to the process, making the process see EOF on stdin.

### Returns

Nothing.

---
