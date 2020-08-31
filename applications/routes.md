# Configuring routes

Configuring routes in Kore happens in the Kore configuration inside
of the domain configuration block.

Routes are evaluated from top to bottom.
Routes can either contain a fixed string of be a regular expression.

```
domain * {
	route / root_page
	route /about/ about_page

	route ^.*$ redirect
}
```

In the example above the configuration specifies 2 fixed routes and one
regex route that captures all the other paths and sends them to some
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
