# Client Authentication

Kore supports client authentication if turned on for a given domain.

If turned on Kore will request an X509 certificate from the client
and verify it against the configured list of trusted certificate authorities.

## Enabling client authentication (via config)

In order to turn on client authentication add the **client_verify** and
**client_verify_depth** configuration directives to the domain you wish
to enable it on.

```
domain needsauth.example.com {
	certfile	cert/example.com/server.pem
	certkey		cert/example.com/key.pem

	# Bundle of trusted certificate authorities and an optional CRL 
	client_verify	cert/cabundle.pem cert/crloptional.pem

	# The verification depth
	client_verify_depth	1
}
```

## Enabling client authentication (via Python API)

You can enable client authentication via the Python API as well by
passing the **client_verify** and **verify_depth** keyword to the domain setup.

```python
dom = kore.domain("needsauth.example.com",
    attach="server",
    cert="cert/example.com/server.pem",
    key="cert/example.com/key.pem",
    client_verify="cert/cabundle.pem",
    verify_depth=1
)
```

You currently cannot set CRLs via the Python API.
