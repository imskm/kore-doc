# kore.conf

The configuration file of an application describes to Kore what modules to load, how validators work, what page handlers to map to which functions and more.

Therefore it is an integral part of Kore as a whole.

Below we will quickly go over all available quick toggle configuration options, their default settings and what they do.

There are more options than what is listed below, specifically for validators, authentication blocks and domains. Please find those in https://github.com/jorisvink/kore/blob/master/conf/kore.conf.example.

---

**socket_backlog**
> Maximum length to queue pending connections (see listen(2)). Must be set before any bind directives.

**bind**
> Bind to a given IP address and port number.

**chroot**
> The directory the worker processes will chroot() into.

**runas**
> The username the worker processes drop privileges to.

**workers** (default: 1)
> The number of worker processes to spawn and keep alive.

**worker_max_connections** (default: 250)
> The number of active connections each worker will accept.

**worker_rlimit_nofiles** (default: 1024)
> Limit of maximum open files per worker.

**worker_accept_threshold** (default: disabled)
> Limit the number of new connections a worker can accept in a single event loop.
By default Kore will accept as many new connections it can up to worker_max_connections.

**worker_set_affinity** (default: enabled)
> Workers bind themselves to a single CPU by default. Turn this off by setting this option to 0.

**pidfile** (default: none)
> Store the pid of the parent process in this file.

**http_header_max** (default: 4096)
> Maximum size of HTTP headers (in bytes).

**http_body_max** (default: 1024000)
> Maximum size of an HTTP body (in bytes).
> 
> If set to 0 disallows requests with a body all together.

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

**tls_version** (default: 1.2 only)
> The TLS versions allowed, by default this is set to TLSv1.2 only.

**tls_cipher** (default: A very sane set of ciphersuites preferring AEAD ciphers and ephemeral key exchanges, RSA key exchanges are not enabled).
> The server TLS ciphersuites that are allowed.

**tls_dhparam** (default: dh2048.pem)
> The DH parameters to load (**required**)

