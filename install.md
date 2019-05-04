# Installation

### Building and installing

Kore has been tested to run on the following platforms:

* Linux 3.2 or newer
* OpenBSD 5.3 or newer
* FreeBSD 10 or newer
* macOS 10.10.x or newer

Basic compilation requires the following libraries:

* OpenSSL 1.1.0, 1.1.1 / LibreSSL

Get the 3.2.0 release tarball and signature from [https://kore.io/releases/3.2.0](https://kore.io/releases/3.2.0) and verify it using minisign:

```
minisign -Vm kore-3.2.0.tar.gz -P RWSxkEDc2y+whfKTmvhqs/YaFmEwblmvar7l6RXMjnu6o9tZW3LC0Hc9
```

If verification is successful, build it. Do not build distributions that
cannot be verified by the minisign key seen above.

```
$ cd kore
$ make
$ sudo make install
```

Kore has several build flavors available:

* CURL=1 \(compiles in asynchronous curl support\)
* TASKS=1 \(compiles in task support\)
* PGSQL=1 \(compiles in pgsql support\)
* DEBUG=1 \(enables use of -d for debug\)
* NOTLS=1 \(compiles Kore without TLS\)
* NOHTTP=1 \(compiles Kore without HTTP support\)
* NOOPT=1 \(disable compiler optimizations\)
* JSONRPC=1 \(compiles in JSONRPC support\)
* PYTHON=1 \(compiles in Python support\)

These build flavors can be passed on the command line when building. Note that enabling these flavors may require additional libraries to be present on your system:

Requirements for background tasks \(optional\)

* pthreads

Requirements for pgsql \(optional\)

* libpq

Requirements for python \(optional\)

* Python 3.6+

For BSD-like systems you will need to install GNU Make.

### macOS

Kore is available on Homebrew under macOS and can be installed with:

```
$ brew install kore
```



