# build.conf

The **build.conf** file dictates build flavors, their CFLAGS and LDFLAGS and whether or not a single binary should be constructed or not.

A default **build.conf** looks like this:

```
# integers build config
# You can switch flavors using: kore flavor [newflavor]

# Set to yes if you wish to produce a single binary instead
# of a dynamic library. If you set this to yes you must also
# set kore_source together with kore_flavor and update ldflags
# to include the appropriate libraries you will be linking with.
#single_binary=no
#kore_source=/home/joris/src/kore
#kore_flavor=

# The cflags below are shared between flavors
cflags=-Wall -Wmissing-declarations -Wshadow
cflags=-Wstrict-prototypes -Wmissing-prototypes
cflags=-Wpointer-arith -Wcast-qual -Wsign-compare

dev {
  # These cflags are added to the shared ones when
  # you build the "dev" flavor.
  cflags=-g
}

#prod {
# You can specify additional CFLAGS here which are only
# included if you build with the "prod" flavor.
#}
```

There are global directives and per flavor directives as you can see in the example above. The cflags and ldflags directives are merged between global and the current active flavor.