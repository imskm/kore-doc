# Build framework

Kore provides a flexible way of building your applications through the **conf/build.conf** file.

This file contains build flavors that dictate CFLAGS and LDFLAGS.

Flavors can be switched using the **flavor** command:

```
$ kodev flavor osx
changed build flavor to: osx
$
```

### Sources

Kore will automatically build any source files found under the **src** directory in your application.

### Assets

By default the build tool will automatically convert files found under **assets** into C files and compile them into your code allowing you to access these directly.

Each asset gets its own page handler as well that can be easily added to the configuration, reducing the work you have to do to configure static assets.
Additionally the assets also get an ETag that is automatically included in the response and Kore will honor if-none-match for these assets and send
a 304 if they were not modified.

The example below configures / to serve the static asset index.html.

```
domain * {
	static	/	asset_serve_index_html
}
```

You may specify media types for these static assets in **conf/build.conf**.

```
mime_add=jpg:imge/jpg
mime_add=css:text/css; charset=utf-8
mime_add=html:text/html; charset=utf-8
```

### Single binaries

Kore will normally produce a single dynamic library \(DSO\) file that contains your application logic. This DSO file is then loaded at run-time by the main Kore binary as instructed by your configuration file.

If DSO's are not your cup-o-tea you can opt for building a single binary instead. This requires you to update your **conf/build.conf** file and add the following settings globally:

```
single_binary = yes
kore_source = /path/to/kore/source/
kore_flavor = <flavors..>
```

This will cause _kodev build_ to build the original Kore source code and link your application logic directly into it. It will also add in the configuration file and of course any assets that are under the **assets** directory. The result is a single dynamically linked binary that can be run.

