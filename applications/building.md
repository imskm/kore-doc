# Build framework

Kore provides a flexible way of building your applications through the **conf/build.conf** file.

This file contains build flavors that dictate CFLAGS and LDFLAGS.

Flavors can be switched using _ kodev flavor_:

```
$ kodev flavor osx
changed build flavor to: osx
$
```

### Sources

Kore will automatically build any source files found under the **src** directory in your application.

### Assets

By default the build tool will automatically convert files found under **assets** into C files and compile them into your code allowing you to access these directly.

### Single binaries

Kore will normally produce a single dynamic library \(DSO\) file that contains your application logic. This DSO file is then loaded at run-time by the main Kore binary as instructed by your configuration file.

If DSO's are not your cup-o-tea you can opt for building a single binary instead. This requires you to update your **conf/build.conf** file and add the following settings globally:

```
single_binary = yes
kore_source = /path/to/kore/source/
kore_flavor = <flavors..>
```

This will cause _kore build_ to build the original Kore source code and link your application logic directly into it. It will also add in the configuration file and of course any assets that are under the **assets** directory. The result is a single dynamically linked binary that can be run.

