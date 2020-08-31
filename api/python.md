# Python

This page contains a reference of all exported Kore functions to Python.

Kore must be compiled with **PYTHON=1** for this to work.
Python 3.6 or higher should be used.

---

# A Kore Python application

Since the 4.0.0 release you can use Kore as an asynchronous first
Python web development platform.

Completely configurable via code and easy to setup and write.

A Kore Python application must be setup in the following way:

* Import kore.
* Define a class with a configure() method.
* Instanciate the koreapp global with that class.

The configure() method is called at startup by the Kore platform and
will receive a list of arguments passed to the command-line.

From there you can configure Kore via the kore.config object, create
server contexts and setup domains and routes.

Note that the Python code is expected to run inside of the Kore worker
process, the kore module being imported is not available outside of
this worker process.

Example:

```python
import kore

class MyApp:
    def configure(self, args):
        kore.config.workers = 1
        kore.config.deployment = "dev"

        kore.server("notls", ip="127.0.0.1", port="8888", tls=False)

        domain = kore.domain("*", attach="notls")
        domain.route("/", self.index, methods=["get"])
        domain.route("^/(0-9)$", self.index_with_args, methods=["get"])

    def index(self, req):
        req.response(200, b'hello')

    def index_with_args(self, req, specified_id):
        req.response(200, specified_id)

koreapp = MyApp()
```

## Index

* [Kore module](#koremodule)
  * [configuration](#koreconfig)
  * [constants](#koremoduleconstants)
  * [functions](#koremodulefunctions)
    * [server](#server)
    * [domain](#domain)
      * [route](#domainroute)
    * [log](#log)
    * [timer](#timer)
    * [proc](#proc)
    * [fatal](#fatal)
    * [tracer](#tracer)
    * [task\_create](#taskcreate)
    * [task\_kill](#taskkill)
    * [gather](#gather)
    * [suspend](#suspend)
    * [httpclient](#httpclient)
    * [dbsetup](#dbsetup)
    * [dbquery](#dbquery)
    * [websocket\_broadcast](#websocketbroadcast)
    * [worker](#worker) TODO
    * [setname](#setname) TODO
    * [coroname](#coroname) TODO
    * [corotrace](#corotrace) TODO


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


* [Asynchronous libcurl support](#curl)
 * [curl](#curlhandle)
 * [setopt](#curlsetopt)

# Kore {#koremodule}

## Configuration {#koreconfig}

Kore configuration options can be set via the kore.config module.

All options available in the traditional kore configuration be
set via the Python code.

| Configuration option | Description |
| --- | --- |
| root | The root path in which the Kore server runs (either via chroot or chdir). If not set, the current working directory. |
| runas | The user the worker processes will run as. If not set, the current user. |
| workers | The number of worker processes to use. If not set, the number of CPU cores in the system. |
| worker\_max\_connections | The maximum number of active connections a worker process holds before refusing to accept more. |
| worker\_rlimit\_nofiles | The maximum number of open file descriptor per worker. |
| worker\_accept\_threshold | The maximum number of new connections to accept in a single event loop. |
| worker\_death\_policy | The death policy for a worker, "restart" by default. If set to "terminate" will cause the Kore server to shutdown on abnormal worker termination. |
| worker\_set\_affinity | Worker CPU affinity (0 or 1, default 1). |
| pidfile | The path to a file in which the server will write the PID for the parent process. |
| socket\_backlog | The number of pending connections. |
| tls\_version | The TLS version to use (default: both, 1.2 for TLSv1.2 only and 1.3 for TLSv1.3 only). |
| tls\_cipher | OpenSSL ciphersuite list to use. Defaults to a very sane list with only AEAD ciphers and ephemeral key exchanges. |
| tls\_dhparam | Path to DH parameters for the server to use. |
| rand\_file | Path to a 2048 byte file containing entropy used to seed the PRNG. |
| keymgr\_runas | The user the keymgr process will run as. If not set, the current user. |
| keymgr\_root | The root path for the keymgr process. If not set, inherited from the root option. |
| acme\_runas | The user the acme process will run as. If not set, the current user. |
| acme\_root | The root path for the acme process. If not set, inherited from the root option. |
| acme\_email | An email adress used for account registration. |
| acme\_provider | A URL to the directory for an ACME provider. Defaults to Let's Encrypt. |
| pledge | OpenBSD only, pledge categories for the worker processes. |
| seccomp\_tracing | Linux only, seccomp violations will be logged and not cause the process to terminate. Either "yes" or "no". |
| filemap\_ext | The default extension for files in a filemap. |
| filemap\_index | The root file in a filemap. (eg index.html). |
| http\_media\_type | Add a new HTTP media type (in the form of "mediatype ext1 ext2 ext"). |
| http\_header\_max | The maximum number of bytes HTTP headers can consist of. If a request comes in with headers larger than this the connection is closed. Defaults to 4096 bytes. |
| http\_header\_timeout | The number of seconds after which Kore will close a connection if no HTTP headers were received. Defaults to 10. |
| http\_body\_max | The maximum number of bytes an HTTP body can consist of. If a request comes in with a body larger than this the connection is closed with a 413 response. Defaults to 1MB. |
| http\_body\_timeout | The number of seconds after which Kore will close a connection if no HTTP body was received in full. Defaults to 60. |
| http\_body\_disk\_offload | The number in bytes from which point Kore will offload incoming HTTP bodies onto a file on disk instead of keeping it in memory. Disabled by default. |
| http\_body\_disk\_path | A path where the temporary body files are written if the http\_body\_disk\_offload setting is enabled. |
| http\_server\_version | Allows you to override the Kore server header. |
| http\_pretty\_error | If set to "yes" will display HTML based HTTP error codes. Defaults to "no". |
| deployment | The deployment type of the application. Either "production" or "development". The production setting will cause Kore to chroot and drop privileges and run in the background. The development setting will run Kore in the foreground and only chdir into the root setting. |

Any configuration options should be set in your App.configure method.

Example

```python
import os
import kore

class MyApp:
    def configure(self, args):
        kore.config.workers = 1
        kore.config.http_header_max = 1024
        kore.config.http_header_timeout = 60
        kore.config.seccomp_tracing = "yes"
        kore.config.deployment = os.getenv("DEPLOYMENT", default="dev")

```

## Constants {#koremoduleconstants}

The kore module exports some constants that can be used by the Python code.

* LOG\_INFO
* LOG\_NOTICE
* LOG\_ERR
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

If Kore was built with CURL=1 all curl\_easy\_setopt CURLoption constants
are exported via the kore module as well.

## Functions {#koremodulefunctions}

---

# server {#server}

### Synopsis

```python
kore.server(name, ip="ip", port="port", path="/tmp/web.sock", tls=True)
```

### Description

Setup a new server with the given **name**.

The ip and path keywords are mutually exclusive.

| Parameter | Description |
| --- | --- |
| name | The name of this listener. |

| Keyword | Description |
| --- | --- |
| ip | This keyword specifies the IP address to bind to. |
| port | This keyword specifies the port to bind to. |
| path | This keyword specifies the UNIX path to bind to. |
| tls | This keyword specifies if TLS is enabled or not (True by default). |

### Returns

Nothing

### Example

```python
kore.server("default", ip="127.0.0.1", port="8888", tls=False)
```

---

# domain {#domain}

### Synopsis

```python
kore.domain(host, attach="server", cert="cert.pem", key="key.pem", acme=False,
    client_verify="ca.pem", verify_depth=1)
```

### Description

Setup a new domain for **host** attached to the given server.

| Parameter | Description |
| --- | --- |
| host | The hostname of this domain (eg: kore.io). |

The cert and key keywords should not be specified if acme is True.

| Keyword | Description |
| --- | --- |
| cert | The path to the certificate for this domain. |
| key | The path to the private key for this domain. |
| acme | If true will use the configured ACME provider (let's encrypt by default) to automaticall obtain an X509 for this domain. |
| client\_verify | If present points to a PEM file containing a Certificate Authority for which the client should present a certificate for. |
| verify\_depth | Maximum depth for the certificate chain. |

### Returns

A domain handle on which you can install routes.

### Example

```python
dom = kore.domain("kore.io", attach="server", acme=True)
dom.route("/", self.index, methods=["get"])
```

---

# domain.route {#domainroute}

### Synopsis

```python
domain.route(url, callback, methods=[], methodname={}, auth=None)
```

### Description

Setup a new route in the domain. The route is attached to the given
**url** and will call the **callback** function when hit.

| Parameter | Description |
| --- | --- |
| url | URL for this route, can contain regex and capture groups. Each capture group is passed as a seperate parameter to the callback after the initial request object. |
| callback | The callback to call for this route. This callback takes at least one parameter: the request object. |

| Keyword | Description |
| --- | --- |
| methods | A list of allowed methods. Any request to this route with an uncorrect method will automatically result in a 405. |
| key | The path to the private key for this domain. |
| methodname | For each supported method a dictionary containing parameters for the route and how they are validated. |
| auth | If set should be a dictionary containing the authentication information for the route. |

### Returns

Nothing

### Example

```python
async def validate_user(req, data):
    return True

async def validate_auth(req, data):
    return True

def route_gethash(req, hash):
    req.response(200, hash.encode())

dom = kore.domain("kore.io", attach="server", acme=True)

dom.route("/", route_index, methods=["get"])

dom.route("/update", route_update, methods=["post"],
    post={
        "id"   :   "^[0-9]+$",
        "user" : validate_user
    }
)

dom.route("^/([a-f0-9]{32})$", route_gethash, methods=["get"])

dom.route("/secret", route_secret, methods=["get"],
    auth={
        "type"      : "header",      # header or cookie are both supported.
        "value"     : "x-header",    # header name or cookie name.
        "redirect"  : "/",           # Redirect to / on auth failure, if not set
                                     # auth failure will result in a 403.
        "verify"    : validate_auth  # Callback to verify the authenticator.
    }
)
```

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
		stdout = await proc.recv(8192)
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

| Parameter | Description |
| --- | --- |
| method | The method to call from the new task. |

### Returns

The unique task identifier that can be passed to kore.task\_kill().

### Example

```python
import kore

async def coro(id):
	print("i am coro %d" % id)

def kore_worker_configure():
	kore.task_create(coro(1))
	kore.task_create(coro(2))
```

---

# task\_kill {#taskkill}

### Synopsis

```python
kore.task_kill(taskid)
```

### Description

Kills the coroutine task specified in the **taskid** identifier.

The coroutine will immediately be killed.

| Parameter | Description |
| --- | --- |
| taskid | The taskid identifier to be killed. |

### Returns

Nothing

### Example

```python
import kore

async def coro(id):
	while True:
		await kore.suspend(1000)
		print("i am coro %d" % id)

async def kill(taskid):
	await kore.suspend(5000)
	kore.task_kill(taskid)

def kore_worker_configure():
	task = kore.task_create(coro(1))

	kore.task_create(kill(task))
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

| Keywords | Description |
| --- | --- |
| concurrency | The number of coroutines to run at the same time. |

### Returns

Nothing

### Example

```python
import kore

async def coro(id):
	print("i am coro %d" % id)

async def request(req):
	results = await kore.gather(coro(1), coro(2))

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

This function must be awaited.

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

# dbsetup {#dbsetup}

### Synopsis

```python
kore.dbsetup(name, connection_string)
```

### Description

Associates a pgsql connection string with a shortname for a database that
can be used later via <a href="#dbquery">dbquery</a>.

| Parameter | Description |
| --- | --- |
| name | The friendly name for the database. |
| connection\_string | The pgsql connection string. |

### Returns

Nothing

### Example

```python
kore.dbsetup("db", "host=/tmp dbname=hello")
```

---

# dbquery {#dbquery}

### Synopsis

```python
result = await kore.dbquery(db, query, params=[v1, v2, ...])
```

### Description

Performs an asynchronous PostgreSQL query. The query should be a parameterized
and the arguments for each parameter are passed via the **params** keyword.

| Parameter | Description |
| --- | --- |
| db | A previously configured database name. |
| query | The query to be performed. |
| params | A list of strings or byte object arguments. |

### Returns

The result of the query as a dictionary where each key is the column name
and the matching value for that column.

If the returned data for a column was binary the returned value will be
of the **bytes** type, otherwise the returned value will be a python string.

### Example

```python
async def myquery(req):
	name = "bar"
	result = await kore.dbquery("db",
		"SELECT * FROM table WHERE foo = $1", params=[name])
	req.response(200, json.dumps(result).encode("utf-8"))
```

---

## Asynchronous HTTP client {#httpclient}

The Python API in Kore contains a full asynchronous HTTP client that allows
you to fire off HTTP requests from your Python code and await their result.

Before you can use the client you must set it up.

**note** Kore must be built with CURL=1 for the httpclient to be included.

```python
client = kore.httpclient(url, keywords)
```

| Parameter | Description |
| --- | --- |
| url | The URL to send the request to. |

Addiotnally, optional keyword parameters can be specified:

| Keywords | Description |
| --- | --- |
| tlscert | A path to an x509 certificate to be used to authenticate this request. |
| tlskey | A path to the private key to be used to authenticate this request. |
| tlsverify | Should the peer its x509 certificate be verified (default: True). |
| cabundle | A path to a PEM bundle containing the CAs you would like to use to verify the peer. |
| unix | Use this unix socket path instead (On Linux if prefixed with '@' this becomes an abstract socket path. |

Kore internally will re-use connections to the same hosts if they are
available, as a developer you can go ahead and create multiple httpclient
objects to the same host, they will all share underlying connections.

Once the httpclient object is created you can call any of the following
functions on it to perform the requested action (these calls return a coroutine and must be awaited):

* get
* put
* post
* head
* patch
* delete
* options

| Keywords | Description |
| --- | --- |
| body | For put, post and patch, add the given bytes as the HTTP body. |
| headers | A dictionary containing the headers to be added to the request. |
| return\_headers | Set to True to return the headers as well (default: False). |

Unless the return\_headers keyword is True, these functions return a tuple
containing the status and HTTP body. If return\_headers is True, the tuple
returned will be status, headers, HTTP body.

### Example

```python
client = kore.httpclient("https://kore.io")

# Do a normal GET request.
status, body = await client.get()

# Do a POST.
mybody = "Hello world"
status, body = await client.post(
	body=mybody.encode("utf-8"),
	headers={
		"x-header": "from-client"
	}
)

# Get the headers from a GET request.
status, headers, body = await client.get(return_headers=True)
```

---

## Asynchronous libcurl client {#curl}

The Python API in Kore contains a full asynchronous libcurl client that allows
you to fire off any protocol that libcurl supports from your Python code and
await their result.

Before you can use the client you must set it up.

**note** Kore must be built with CURL=1 for the curl handler to be included.

---

# Creating a new curl handle {#curlhandle}

### Synopsis

```python
handle = kore.curl(url)
```

| Parameter | Description |
| --- | --- |
| url | The URL to send the request to. |

Once the libcurl object is created you can call handle.run() to perform
the call to the specified url.

Additionally you can call handle.setopt() which behaves a lot like
the curl\_easy\_setopt() function.
### Example

```python
handle = kore.curl("https://kore.io")

# Run the libcurl handle.
data = await handle.run()
```

---

# Using libcurl setopt from Python {#curlsetopt}

### Synopsis

```python
handle.setopt(option, value)
```

| Parameter | Description |
| --- | --- |
| option | The libcurl option to be set. |
| value | The value to set the option to. |

All libcurl constants are exported under the kore module directly.

Example:
```python
handle = kore.curl("https://kore.io")
handle.setopt(kore.CURLOPT_TIMEOUT, 10)
```

Depending on what the original option in libcurl takes the value must
either be a string, an integer or a list.

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

A RuntimeError exception is thrown if the *length* parameter is greater than 1024.

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
tuple = await sock.recvfrom(1024)
```

### Description

Reads up to **maxlen** bytes from the socket while returning the source
and data.

| Parameter | Description |
| --- | --- |
| maxlen | Maximum number of bytes to be read. |

### Returns

A tuple of (address, port, data) for AF\_INET,
A tuple of (address, data) for AF\_UNIX.

None on EOF.

### Example

```python
ip, port, data = await sock.recvfrom(1024)
if data is None:
    printf("eof!")
else:
    await sock.sendto(ip, port, data)
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

for AF\_INET:

| Parameter | Description |
| --- | --- |
| address | The destination IP address. |
| port | The destination port. |
| data | The data to be sent (as bytes). |

For AF\_UNIX:

| Parameter | Description |
| --- | --- |
| address | The destination address path. |
| data | The data to be sent (as bytes). |

### Returns

Nothing. In case the peer closes the socket before all data is written
a RuntimeException is thrown.

### Example

```python
# AF_INET, will return when all bytes are sent.
await sock.sendto("127.0.0.1", "3311", "Hello world")

# AF_UNIX, will return when all bytes are sent.
await sock.sendto("/tmp/service.sock", "Hello world")
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
* [popnow](#queuepopnow)

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

### queue.popnow {#queuepopnow}

### Synopsis

```python
object = queue.popnow()
```

### Description

Attempts to pop an object from a queue immediately.

### Returns

A popped object from the queue or None if the queue was empty.

### Example

```python
d = queue.popnow()
if not d:
	continue
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

