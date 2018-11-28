# The kodev tool

Kore provides a development tool called **kodev** to help you build and create applications.

```
kodev create
kodev build
kodev run
kodev flavor
kodev info
```

### Creating an application

Creating a new application is done via the _ kodev create_ tool. This tool will create a new directory and populate it with all required files to get hacking.

The tool will automatically generate a self-signed X.509 certificate for development purposes.

```
$ kodev create hello
created hello/src/hello.c
created hello/conf/hello.conf
created hello/conf/build.conf
created hello/.gitignore
created dh2048.pem
hello created successfully!
WARNING: DO NOT USE THE GENERATED DH PARAMETERS AND CERTIFICATES IN PRODUCTION
$
```

### Building an application

You can build a Kore application using the _ kodev build_ tool.

This tool will read your **conf/build.conf** file and build your application according to it.

```
$ kodev build
building hello (dev)
CFLAGS=-Wall -Wmissing-declarations -Wshadow -Wstrict-prototypes -Wmissing-prototypes -Wpointer-arith -Wcast-qual -Wsign-compare -fPIC -I./src -I./src/includes -I/usr/local/include -I/opt/local/include -I/usr/local/opt/openssl/include -g
LDFLAGS=-dynamiclib -undefined suppress -flat_namespace
compiling hello.c
hello built successfully!
$
```

### Running an application

You can run a Kore application in the foreground using the _ kodev run_ tool.

This tool will simply build the application \(if any building needs to happen\) and run it in the foreground. You can CTRL-C it to bring it back down.

```
$ kodev run
building hello (dev)
CFLAGS=-Wall -Wmissing-declarations -Wshadow -Wstrict-prototypes -Wmissing-prototypes -Wpointer-arith -Wcast-qual -Wsign-compare -fPIC -I./src -I./src/includes -I/usr/local/include -I/opt/local/include -I/usr/local/opt/openssl/include -g
LDFLAGS=-dynamiclib -undefined suppress -flat_namespace
nothing to be done!
[parent]: running on https://127.0.0.1:8888
[parent]: kore is starting up
[keymgr]: key manager started
[wrk 1]: worker 1 started (cpu#1)
^C[keymgr]: cleaning up keys
[keymgr]: parent gone, shutting down
[wrk 1]: parent gone, shutting down
[parent]: server shutting down
[parent]: waiting for workers to drain and shutdown
[parent]: worker 0 (21752)-> status 0
[parent]: worker 1 (21753)-> status 0
$
```

See the [Running](/applications/running.md) section on how to start your Kore application in the background for production purposes.

### Environment Variables

The **kodev** tool will pickup the following environment variables if set:

**CFLAGS**
> Any additional compiler flags.

**LDFLAGS**
> Any additional linker flags.

**KORE_PREFIX**
> The prefix where kore was installed.

**KORE_SOURCE**
> Path where to find kore source, overrides **kore_source** from build.conf.

**KORE_FLAVOR**
> Flavor to use, overrides **kore_flavor** from build.conf.

**KORE_OBJDIR**
> The directory where the .o files will be placed.

**KODEV_OUTPUT**
> The directory where the resulting binary is placed.



