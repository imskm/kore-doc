# Filemaps

A filemap is a configuration option in Kore that allows mapping
of on-disk files to a URI root.

A filemap must be declared inside of a domain block.

```
domain kore.io {
	filemap		/3.2.0/		docs_3_2_0
}
```

Where the first argument is the URI root and the second is the on-disk
path where the files will be served from.

Thus from the example above the URL /3.2.0/index.html will be served from
the file docs\_3\_2\_0/index.html.

Kore has the following builtin mime types:

| Extension | MIME type|
| --- | --- |
| gif | image/gif |
| png | image/png |
| jpeg | image/jpeg |
| jpg | image/jpeg |
| zip | application/zip |
| pdf | application/pdf |
| json | application/json |
| js | application/javascript |
| htm | text/html |
| txt | text/plain |
| css | text/css |
| html | text/html |

Additional mime types can be added via the configuration option **http\_media\_type**

```
http_media_type [mime type] ext1 ext2 extN
```
