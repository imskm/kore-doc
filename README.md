# Kore web framework

Kore is a web application framework written in C that allows you to create blazing fast web applications. It focuses on security, stability and rapid development.

The latest version is [3.0.0](https://github.com/jorisvink/kore/releases) and was released DD/MM/2018 \(release is upcoming\).

This gitbook serves as the main documentation for the project.

The documentation is correct for the latest 3.0.0 release but may not be correct for the latest master branch on github.

# Features

* Supports SNI.
* Supports HTTP/1.1.
* Websocket support.
* TLS enabled by default.
* Optional background tasks.
* Built-in parameter validation.
* Fully privilege separated by default.
* Optional asynchronous PostgreSQL support.
* Optional support for page handlers in Python.
* Private keys isolated in separate process \(RSA and ECDSA\).
* Default sane TLS ciphersuites \(PFS in all major browsers\).
* Modules can be reloaded on-the-fly, even while serving content.
* Event driven \(epoll/kqueue\) architecture with per CPU worker processes.
* Build your web application as a precompiled dynamic library or single binary.

# Architecture overview

![](arch.png)

