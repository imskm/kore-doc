# kore.conf

The configuration file of an application describes to Kore what modules to load, how validators work, what page handlers to map to which functions and more.

Therefore it is an integral part of Kore as a whole.

# Server context

A server context sets up a one or more listeners for the Kore server. These
listeners can either be ipv4/ipv6 addresses or unix sockets.

The old bind and bind_unix configuration options have been migrated since
Kore 4.0.0 to these server contexts. Note that the bind_unix option has been
renamed to unix.

You can also turn off TLS in a server context by specifying the **tls no**
option inside of a context.

Example:

```
server tls {
	bind 127.0.0.1 443
	bind ::1 443
	unix /var/run/socket.path
	unix @linux-abstract-socket
}

server notls {
	bind 127.0.0.1 80
	tls no
}
```

# Configuration options.

There are more options than what is listed below, specifically for validators, authentication blocks and domains. Please find those in https://github.com/jorisvink/kore/blob/master/conf/kore.conf.example.

---

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
| acme\_email | An email address used for account registration. |
| acme\_provider | A URL to the directory for an ACME provider. Defaults to Let's Encrypt. |
| pledge | OpenBSD only, pledge categories for the worker processes. |
| seccomp\_tracing | Linux only, seccomp violations will be logged and not cause the process to terminate. Either "yes" or "no". |
| filemap\_ext | The default extension for files in a filemap. |
| filemap\_index | The root file in a filemap. (eg index.html). |
| http\_keepalive\_time | The time an HTTP connection is kept-alive server side. Defaults to 20 seconds. |
| http\_media\_type | Add a new HTTP media type (in the form of "mediatype ext1 ext2 ext"). |
| http\_header\_max | The maximum number of bytes HTTP headers can consist of. If a request comes in with headers larger than this the connection is closed. Defaults to 4096 bytes. |
| http\_header\_timeout | The number of seconds after which Kore will close a connection if no HTTP headers were received. Defaults to 10. |
| http\_body\_max | The maximum number of bytes an HTTP body can consist of. If a request comes in with a body larger than this the connection is closed with a 413 response. Defaults to 1MB. |
| http\_body\_timeout | The number of seconds after which Kore will close a connection if no HTTP body was received in full. Defaults to 60. |
| http\_body\_disk\_offload | The number in bytes from which point Kore will offload incoming HTTP bodies onto a file on disk instead of keeping it in memory. Disabled by default. |
| http\_body\_disk\_path | A path where the temporary body files are written if the http\_body\_disk\_offload setting is enabled. |
| http\_server\_version | Allows you to override the Kore server header. |
| http\_pretty\_error | If set to "yes" will display HTML based HTTP error codes. Defaults to "no". |
