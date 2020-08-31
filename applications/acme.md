# Automatic HTTPs using ACME

Kore can automatically provision certificates from ACME providers
such as Let's Encrypt and others.

## Enabling ACME on a domain

Enabling ACME is quite straight forward. Make sure Kore was built with
the ACME=1 directive set at compile time.

If ACME support was compiled in it can be verified by showing the
version string for the Kore binary installed.

```
$ kore -v
master-8097d22d curl-7.64.1 pgsql python-3.7.7 acme
$
```

In your configuration, under the domain context you set the acme
configuration option to yes.

```
domain kore.io {
	acme yes
	accesslog /var/log/kore.log
	route / serve_index
}
```

## ACME configuration

There are a few ACME related configuration options.

| Configuration option | Description |
| --- | --- |
| acme\_runas | The user the acme process will run as. If not set, the current user. |
| acme\_root | The root path for the acme process. If not set, inherited from the root option. |
| acme\_email | An email adress used for account registration. |
| acme\_provider | A URL to the directory for an ACME provider. Defaults to Let's Encrypt. |

The default ACME provider is "https://acme-v02.api.letsencrypt.org/directory".

## ACME architecture

When ACME is enabled, Kore will create a new acme process that stands
alone from your workers. It is this process that will talk to the
ACME servers and perform requests to them.

The acme process will communicate when needed with the keymgr who holds
all your private keys (even the ACME account key is only held by keymgr).

## ACME files

All certificates and private keys are stored under the directory that
was configured via the **keymgr_root** configuration option.

The RSA account key is stored as **account-key.pem** in the **keymgr_root**
directory while certificates and matching domain keys are stored under
the **certificates**  directory separated by hostnames.
