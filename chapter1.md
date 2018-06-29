# Installation

### Building and installing

Kore has been tested to run on the following platforms:

* Linux 3.2 or newer
* OpenBSD 5.3 or newer
* FreeBSD 10 or newer
* OSX 10.10.x or newer

Basic compilation requires the following libraries:

* openssl

Download the latest release tarball from [https://github.com/jorisvink/kore/releases](https://github.com/jorisvink/kore/releases) and build it:

```
$ cd kore
$ make
$ sudo make install
```

Kore has several build flavors available:

* TASKS=1 \(compiles in task support\)
* PGSQL=1 \(compiles in pgsql support\)
* DEBUG=1 \(enables use of -d for debug\)
* NOTLS=1 \(compiles Kore without TLS\)
* NOHTTP=1 \(compiles Kore without HTTP support\)
* NOOPT=1 \(disable compiler optimizations\)
* JSONRPC=1 \(compiles in JSONRPC support\)
* PYTHON=1 \(compiles in Python support\)

These build flavors can be passed on the command line when building. Note that enabling these flavors may require additional libraries to be present on your system.:

Requirements for background tasks \(optional\)

* pthreads

Requirements for pgsql \(optional\)

* libpq

Requirements for python \(optional\)

* Python 3.6+

For BSD-like systems you will need to install GNU make.

### OSx

Kore is available on Homebrew under OSx and can be installed with:

```
$ brew install kore
```

### Verification

All releases can be verified with [minisign](https://jedisct1.github.io/minisign/) and the following public key:

```
RWSxkEDc2y+whfKTmvhqs/YaFmEwblmvar7l6RXMjnu6o9tZW3LC0Hc9
```


