# Configuring routes

Configuring routes in Kore happens in the Kore configuration inside
of the domain configuration block.

Routes are evaluated from top to bottom.
Routes can either contain a fixed string or a regular expression.

```
domain * {
	route / {
		handler root_page
	}
	route /about/ {
		handler about_page
	}

	route ^.*$ {
		handler redirect
	}
}
```

In the example above the configuration specifies 2 fixed routes and one
regex route that captures all the other paths and sends them to some
redirection function. By default route is for GET method if request
method is not specified.

If you need route for different request method you should use `methods`
directive to specify one or more methods as shown below:

```
domain * {
	route /comment {
		handler comment_page
		methods post
	}

	route /posts {
		handler post_create
		methods get post
	}
}
```

In the above example the first route only handles POST request but in
the second route two methods are handled by same route i.e. GET and POST.

# Parameter configuration

If you wish to receive parameters via a route you **must** define them
in the configuration **and** specify how they should be validated.

Validation is not optional.

You should define all the parameter validation of the route inside the
route block for all types of request methods as shown below:


```
# First we must define a validator (in the global config)
validator v_number regex ^[0-9]$
validator v_title  regex ^[0-9a-zA-Z]{1,128}$

# Then inside the route {} configuration we can add the validate directive
# for all parameters on an existing route:
route /posts {
	...

	validate post title v_title
}

# You can also validate more than one parameter in the same route block:
route /posts {
	...

	# Validate post parameter
	validate post title v_title

	# Validate "id" param in get query string
	validate qs:get id v_number

	# Validate "id" param in delete query string
	validate qs:delete id v_number
}

```

The route block syntax is as follows:

```
route route {
	handler page_handler_function

	methods get post ...

	# For validating parameters
	validate method parameter validator

	# For attaching authentication handler
	authenticate auth_handler
}
```

Where **method** is the lowercase method (eg: post, get) and optionally
prefixed with **qs:** indicating the query string should be validated with
this parameters.

If validation for a parameter fails it is filtered out.

**NOTE:**

Old route definition has been changed from 4.1.0 to new route configuration
definition and param block has been removed in this version all the validate
directive must be used inside `route` block as shown above.
