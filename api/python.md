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
    * [task\_create](#taskcreate)
    * [gather](#gather)
    * [suspend](#suspend)
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
  * [Sockets](#asyncsocket)
  * [Locks](#asynclock)
  * [Queues](#asyncqueue)
  * [Processes](#asyncproc)


# Kore {#koremodule}

## Constants {#koremoduleconstants}

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

## Functions {#koremodulefunctions}

---

# log {#log}

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

# timer {#timer}

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

# proc {#proc}

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

# fatal {#fatal}

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

# tracer {#tracer}

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

# task\_create {#taskcreate}

### Synopsis

```python
kore.task_create(method)
```

### Description

Creates a new task that will run the given method.

There is no way to obtain the result from a task, it will run until
it returns from the parent function.

| Parameter | Description |
| --- | --- |
| method | The method to call from the new task. |

### Returns

Nothing

### Example

```python
import kore

def coro(id):
	print("i am coro %d" % id)

def kore_worker_configure():
	kore.task_create(coro(1))
	kore.task_create(coro(2))
```

---

# gather {#gather}

### Synopsis

```python
kore.gather(coroutines)
```

### Description

Awaits all given coroutines and returns their result in a list.

If a coroutine throws an exception that exception is returned as the
result for that coroutine.

| Parameter | Description |
| --- | --- |
| coroutines | The coroutines to wait for. |

### Returns

Nothing

### Example

```python
import kore

def coro(id):
	print("i am coro %d" % id)

async def request(req):
	results = kore.gather(coro(1), coro(2))

	for result in results:
		if isinstance(result, Exception):
			raise result
		print("result is %s" % result)

	req.response(200, b'')
```

---

# suspend {#suspend}

### Synopsis

```python
kore.suspend(milliseconds)
```

### Description

Suspends the current coroutine for the specified amount of milliseconds.

| Parameter | Description |
| --- | --- |
| milliseconds | Number of milliseconds to suspend execution for. |

### Returns

Nothing

### Example

```python
import kore

async def request(req):
	await kore.suspend(1000)
	req.response(200, b'')
```

---

# websocket\_broadcast {#websocketbroadcast}

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

# register\_database {#registerdatabase}

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


# HTTP {#httpmodule}

Like the C API Kore will pass an http\_request data structure to your
Python page handler. This data structure is of type **kore.http\_request**.

## Constants {#httpmoduleconstants}

* HTTP\_METHOD\_GET
* HTTP\_METHOD\_PUT
* HTTP\_METHOD\_HEAD
* HTTP\_METHOD\_POST
* HTTP\_METHOD\_DELETE
* HTTP\_METHOD\_OPTIONS
* HTTP\_METHOD\_PATCH

## Getters {#httpmodulegetters}

* host - The domain as a unicode string.
* agent - The user agent as a unicode string.
* path - The requested path as a unicode string.
* body - The entire incoming HTTP body as a PyBuffer.
* method - The requested method as a PyLong. (kore.HTTP\_METHOD\_GET, etc).
* body\_path - The path to the HTTP body on disk (if enabled).
* connection - The underlying client connection as a **kore.connection** object.

## Functions {#httpmodulefunctions}

---

# pgsql {#pgsql}

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

# cookie {#cookie}

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

# response {#response}

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

# argument {#argument}

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

# body\_read {#bodyread}

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

# file\_lookup {#filelookup}

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

# populate\_get {#populateget}

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

# populate\_post {#populatepost}

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

# populate\_multipart {#populatemultipart}

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

# populate\_cookies {#populatecookies}

### Synopsis

```python
req.populate_cookies()
```

### Description

Instructs Kore to go ahead and parse the incoming cookie header (if any).

### Returns

Nothing

---

# request\_header {#requestheader}

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

	req.response(200, b'hello world')
```

---

# response\_header {#responseheader}

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

	req.response(200, b'hello world')
```

---

# websocket\_handshake {#websockethandshake}

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


# Async/await {#async}

Kore exposes several functions that can be awaited upon allowing you to
write concurrent page handlers without any callbacks.

Kore currently supports:

  * [Sockets](#asyncsocket)
  * [Locks](#asynclock)
  * [Queues](#asyncqueue)
  * [Processes](#asyncproc)

For example code covering all asynchronous features, please see the
[python-async](https://github.com/jorisvink/kore/tree/master/examples/python-async) example.

You may choose to create new coroutines using <a href="#taskcreate">task_create</a> or await several coroutines using <a href="#gather">gather</a>.

## Asynchronous sockets {#asyncsocket}

You can use an existing Python socket object and wrap it inside of a Kore
socket. When you have a wrapped socket you can connect/accept, send or recv
asynchronously.

* [recv](#socketrecv)
* [recvfrom](#socketrecvfrom)
* [send](#socketsend)
* [sendto](#socketsendto)
* [accept](#socketaccept)
* [connect](#socketconnect)
* [close](#socketclose)

Example:

```python
import kore
import socket

async def page(req):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM |
                                         socket.SOCK_NONBLOCK)

    # Wrap the socket in a Kore object.
    ksock = kore.socket_wrap(sock)

    # Connect and send some data.
    await ksock.connect("127.0.0.1", 3332)
    await ksock.send("hello world")

    # Read a response.
    response = await ksock.recv(1024)

    ksock.close()
    sock.close()
```

---

### sock.recv {#socketrecv}

### Synopsis

```python
data = await sock.recv(1024)
```

### Description

Reads up to **maxlen** bytes from the socket. The optional **timeout** argument
will throw a TimeoutError exception in case the timeout is hit before data is
read from the socket. If no timeout argument is specified, the call will wait
until data is available.


| Parameter | Description |
| --- | --- |
| maxlen | Maximum number of bytes to be read. |
| timeout | Optional timeout in milliseconds. |

### Returns

The bytes read as a byte object or None on EOF.

### Example

```python
data = await sock.recv(1024)
if data is None:
    printf("eof!")
else:
    print("received %s" % data)

# Use optional timeout argument
try:
    data = await sock.recv(1024, 1000)
except TimeoutError as e:
    print("timed out reading (%s)" % e)
```

---

### sock.recvfrom {#socketrecvfrom}

### Synopsis

```python
address, port, data = await sock.recvfrom(1024)
```

### Description

Reads up to **maxlen** bytes from the socket while returning the source
ip and source port.

| Parameter | Description |
| --- | --- |
| maxlen | Maximum number of bytes to be read. |

### Returns

A tuple of (adress, port, data) or None on EOF.

### Example

```python
ip, port, data = await sock.recvfrom(1024)
if data is None:
    printf("eof!")
else:
    sock.sendto(ip, port, data)
```

---

### sock.send {#socketsend}

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

### sock.sendto {#socketsendto}

### Synopsis

```python
await sock.sendto(address, port, data)
```

### Description

Sends the given data over the socket to the specified destination.

| Parameter | Description |
| --- | --- |
| address | The destination IP address. |
| port | The destination port. |
| data | The data to be sent (as bytes). |

### Returns

Nothing. In case the peer closes the socket before all data is written
a RuntimeException is thrown.

### Example

```python
# Will return when all bytes are sent.
await sock.send("127.0.0.1", "3311", "Hello world")
```

---

### sock.accept {#socketaccept}

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

### sock.connect {##socketconnect}

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

### sock.close {#socketclose}

### Synopsis

```python
sock.close()
```

### Description

Closes the underlying socket. If the socket was created from an
underlying Python socket, it is not closed.

### Returns

Nothing.

---

## Locks {#asynclock}

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

## Queues {#asyncqueue}

Kore provides a queue system that allows objects to be sent back and forth
between coroutines in a non-blocking fashion.

The queue is a FIFO queue.

* [push](#queuepush)
* [pop](#queuepop)

---

### queue.push {#queuepush}

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

### queue.pop {#queuepop}

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

## Asynchronous processes {#asyncproc}

For a more detailed example, see [this](https://git.kore.io/kore/file/examples/python-async/src/async_process.py) source file.

Or see the [kore.proc](https://docs.kore.io/3.2.0/api/python.html#proc) description.

* [kill](#prockill)
* [reap](#procreap)
* [recv](#procrecv)
* [send](#procsend)
* [close\_stdin](#procclosestdin)

---

### proc.kill {#prockill}

### Synopsis

```python
proc.kill()
```

### Description

Sends an immediate SIGKILL to the process.

### Returns

Nothing

---

### proc.reap {#procreap}

### Synopsis

```python
retcode = proc.reap()
```

### Description

Reaps the process and captures its exit code as the return value.

### Returns

The process exit code.

---

### proc.recv {#procrecv}

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

### proc.send {#procsend}

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

### proc.close\_stdin {#procclosestdin}

### Synopsis

```python
proc.close_stdin()
```

### Description

Closes the write pipe to the process, making the process see EOF on stdin.

### Returns

Nothing.

---
