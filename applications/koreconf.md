# kore.conf

The configuration file of an application describes to Kore what modules to load, how validators work, what page handlers to map to which functions and more.

Therefore it is an integral part of Kore as a whole.

# Server context

A server context sets up a one or more listeners for the Kore server. These
listeners can either be ipv4/ipv6 addresses or unix sockets.

The old bind and bind_unix configuration options have been migrated since
Kore 4.0.0 to these server contexts.

You can also turn off TLS in a server context by specifying the **tls no**
option inside of a context.

Example:

```
server tls {
	bind 127.0.0.1 443
	bind ::1 443
	bind_unix /var/run/socket.path
	bind_unix @linux-abstract-socket
}

server notls {
	bind 127.0.0.1 80
	tls no
}
```

# Configuration options.

Below we will quickly go over all available quick toggle configuration options, their default settings and what they do.

There are more options than what is listed below, specifically for validators, authentication blocks and domains. Please find those in https://github.com/jorisvink/kore/blob/master/conf/kore.conf.example.

---

**socket_backlog**
> Maximum length to queue pending connections (see listen(2)). Must be set before any bind directives.

**root**
> The directory the worker processes will chroot() or chdir() into.

**runas**
> The username the worker processes drop privileges to.

**workers** (default: number of cores on system)
> The number of worker processes to spawn and keep alive.

**worker_max_connections** (default: 748)
> The number of active connections each worker will accept.

**worker_rlimit_nofiles** (default: 1024)
> Limit of maximum open files per worker.

**worker_accept_threshold** (default: 16)
> The number of accepts a worker will do in one go before going up its
> accept lock to another worker.

**worker_death_policy** (default: restart)
> Workers are restarted when they unexpectedly exit. Setting this to "terminate" will instead bring down the entire Kore server.

**worker_set_affinity** (default: enabled)
> Workers bind themselves to a single CPU by default. Turn this off by setting this option to 0.

**rand_file** (default: none)
> The entropy file to be loaded at startup time.

**keymgr_root**
> The root path for the keymgr process (if not set, will use root from above).

**keymgr_runas**
> The user the keymgr process will drop privileges towards.

**filemap_index** (default: index.html)
> The default filemap index file.

**pidfile** (default: none)
> Store the pid of the parent process in this file.

**http_header_max** (default: 4096)
> Maximum size of HTTP headers (in bytes).

**http_header_timeout** (default: 10)
>  Timeout in seconds for receiving the HTTP headers before the connection is closed.

**http_body_max** (default: 1024000)
> Maximum size of an HTTP body (in bytes).
> 
> If set to 0 disallows requests with a body all together.

**http_body_timeout** (default: 60)
> Timeout in seconds for receiving the HTTP body in full before the connection is closed with an 408.

**http_body_disk_offload** (default: disabled)
> Number of bytes after which Kore will use a temporary file to hold the HTTP body instead of holding it in memory.
> 
> If set to 0 no disk offloading will be done. This is turned off by default.

**http_body_disk_path** (default: tmp_files)
> Path where Kore will store any temporary HTTP body files.

**http_keepalive_time** (default: 20 seconds)
> Maximum seconds an HTTP connection can be kept open by the browser.
> 
> Set to 0 to turn off keep-alive completely.

**http_hsts_enable** (default: 31536000 seconds)
> If not 0 the age of the HSTS header that is included in all responses.

**http_request_limit** (default: disabled)
> The number of HTTP requests Kore workers will process in one loop.

**websocket_maxframe** (default: 16384)
> The maximum number of bytes per websocket frame.

**websocket_timeout** (default: 120 seconds)
> The number of seconds a websocket connection is kept open without traffic.

**task_threads** (default: 2)
> The number of OS threads to use for background tasks.

**tls_version** (default: 1.2, 1.3)
> The TLS versions allowed, by default this is set to TLSv1.2 + TLSv1.3.

**tls_cipher** (default: A very sane set of ciphersuites preferring AEAD ciphers and ephemeral key exchanges, RSA key exchanges are not enabled).
> The server TLS ciphersuites that are allowed.

**tls_dhparam** (default: dh2048.pem)
> The DH parameters to load (**required**)

**curl_recv_max** (default: 2097152)
> Maximum incoming bytes for a response.

**curl_timeout** (default: 60)
> Timeout in seconds before a transfer is cancelled.

