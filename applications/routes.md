# Configuring routes

Configuring routes in Kore happens in the Kore configuration inside
of the domain configuration block.

There are 2 type routes:

* static routes
* dynamic routes

Routes are evaluated from top to bottom.

A static route is a path that is simply matched to the incoming path
for the request with a straight forward string comparison.

A dynamic route is a regular expression that is matched against the
incoming path. This allows you to capture multiple routes towards
the same callback.

```
domain * {
	static / root_page
	static /about/ about_page

	dynamic ^.*$ redirect
}
```

In the example above the configuration specifies 2 static routes and one
dynamic route that captures all the other paths and sends them to some
redirection function.

# Parameter configuration

If you wish to receive parameters via a route you **must** define them
in the configuration **and** specify how they should be validated.

Validation is not optional.

After having defined the route you would continue with a param block:


```
# First we must define a validator (in the global config)
validator v_number regex ^[0-9]$

# Then inside the domain {} configuration we can add the parameter block
# on an existing route:
params qs:get / {
	validate id v_number
}
```

The params block syntax is as follows:

```
params method route {
	validate parameter validator
}
```

Where **method** is the lowercase method (eg: post, get) and optionally
prefixed with **qs:** indicating the query string should be validated with
this parameters.

If validation for a parameter fails it is filtered out.
