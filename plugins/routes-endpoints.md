# Routes &amp; Endpoints

## Overview

The REST API provides us a way to match URIs to various resources in our WordPress install. By default, if you have pretty permalinks enabled, the WordPress REST API “lives” at `/wp-json/`. At our WordPress site `https://ourawesomesite.com`, we can access the REST API’s index by making a `GET` request to `https://ourawesomesite.com/wp-json/`. The index provides information regarding what routes are available for that particular WordPress install, along with what HTTP methods are supported and what endpoints are registered.

If we wanted to create an endpoint that would return the phrase “Hello World, this is the WordPress REST API”, we would first need to register the route for that endpoint. To register routes you should use the `register_rest_route()` function. It needs to be called on the `rest_api_init` action hook. `register_rest_route()` handles all of the mapping for routes to endpoints. Let’s try to create a “Hello World, this is the WordPress REST API” route.

/\*\*
 \* This is our callback function that embeds our phrase in a WP\_REST\_Response
 \*/
function prefix\_get\_endpoint\_phrase() {
    // rest\_ensure\_response() wraps the data we want to return into a WP\_REST\_Response, and ensures it will be properly returned.
    return rest\_ensure\_response( 'Hello World, this is the WordPress REST API' );
}

/\*\*
 \* This function is where we register our routes for our example endpoint.
 \*/
function prefix\_register\_example\_routes() {
    // register\_rest\_route() handles more arguments but we are going to stick to the basics for now.
    register\_rest\_route( 'hello-world/v1', '/phrase', array(
        // By using this constant we ensure that when the WP\_REST\_Server changes our readable endpoints will work as intended.
        'methods'  => WP\_REST\_Server::READABLE,
        // Here we register our callback. The callback is fired when this endpoint is matched by the WP\_REST\_Server class.
        'callback' => 'prefix\_get\_endpoint\_phrase',
    ) );
}

add\_action( 'rest\_api\_init', 'prefix\_register\_example\_routes' );

[Expand full source code](#)[Collapse full source code](#)

The first argument passed into `register_rest_route()` is the namespace, which provides us a way to group our routes. The second argument passed in is the resource path, or resource base. For our example, the resource we are retrieving is the “Hello World, this is the WordPress REST API” phrase. The third argument is an array of options. We specify what methods the endpoint can use and what callback should happen when the endpoint is matched (more things can be done but these are the fundamentals).

The third argument also allows us to provide a permissions callback, which can restrict access for the endpoint to only certain users. The third argument also offers a way to register arguments for the endpoint so that requests can modify the response of our endpoint. We will get into those concepts in the endpoints section of this guide.

When we go to `https://ourawesomesite.com/wp-json/hello-world/v1/phrase` we can now see our REST API greeting us kindly. Let’s take a look at routes a bit more in depth.

## Routes

Routes in the REST API are represented by URIs. The route itself is what is tacked onto the end of `https://ourawesomesite.com/wp-json`. The index route for the API is `'/'` which is why `https://ourawesomesite.com/wp-json/` returns all of the available information for the API. All routes should be built onto this route, the `wp-json` portion can be changed, but in general, it is advised to keep it the same.

We want to make sure that our routes are unique. For instance we could have a route for books like this: `/books`. Our books route would now live at `https://ourawesomesite.com/wp-json/books`. However, this is not a good practice as we would end up polluting potential routes for the API. What if another plugin we wanted to register a books route as well? We would be in big trouble in that case, as the two routes would conflict with each other and only one could be used. The fourth parameter to `register_rest_field()` is a boolean for whether the route should override an existing route.

The override parameter does not really solve our problem either, as both routes could override or we would want to use both routes for different things. This is where using namespaces for our routes comes in.

### Namespaces

It is extremely important to add namespaces to your routes. The “core” endpoints, which are awaiting to be merged into WordPress core, use the `/wp/v2` namespace.

Note: **DO NOT PLACE ANYTHING INTO THE `/wp` NAMESPACE UNLESS YOU ARE MAKING ENDPOINTS WITH THE INTENTION OF MERGING THEM INTO CORE.**

There are some key things to take notice of in the core endpoint namespace. The first part of the namespace is `/wp`, which represents the vendor name; WordPress. For our plugins we will want to come up with unique names for what we call the vendor portion of the namespace. In the example above we used `hello-world`.

Following the vendor portion is the version portion of the namespace. The “core” endpoints utilize `v2` to represent version 2 of the WordPress REST API. If you are writing a plugin, you can maintain backwards compatibility of your REST API endpoints by simply creating new endpoints and bumping up the version number you provide. This way both the original `v1` and `v2` endpoints can be accessed.

The part of the route that follows the namespace is the resource path.

### Resource Paths

The resource path should signify what resource the endpoint is associated with. In the example we used above, we used the word `phrase` to signify that the resource we are interacting with is a phrase. To avoid any collisions, each resource path we register should also be unique within a namespace. Resource paths should be used to define different resource routes within a given namespace.

Let’s say we have a plugin that handles some basic eCommerce functionality. We will have two main resource types orders, and products. Orders are a request for product(s) but they are not the product themselves. The same concept applies to products. Although these resources are related they are not the same thing and each should live in a separate resource paths. Our routes will end up looking something like this for our eCommerce plugin: `/my-shop/v1/orders` and `/my-shop/v1/products`.

Using routes like this, we would want each to return a collection of orders or products. What if we wanted to grab a specific product by ID, we would need to use path variables in our routes.

### Path Variables

Path variables enable us to add dynamic routes. To expand on our eCommerce routes, we could register a route to grab individual products.

/\*\*
 \* This is our callback function to return our products.
 \*
 \* @param WP\_REST\_Request $request This function accepts a rest request to process data.
 \*/
function prefix\_get\_products( $request ) {
    // In practice this function would fetch the desired data. Here we are just making stuff up.
    $products = array(
        '1' => 'I am product 1',
        '2' => 'I am product 2',
        '3' => 'I am product 3',
    );
    
    return rest\_ensure\_response( $products );
}

/\*\*
 \* This is our callback function to return a single product.
 \*
 \* @param WP\_REST\_Request $request This function accepts a rest request to process data.
 \*/
function prefix\_get\_product( $request ) {
    // In practice this function would fetch the desired data. Here we are just making stuff up.
    $products = array(
        '1' => 'I am product 1',
        '2' => 'I am product 2',
        '3' => 'I am product 3',
    );

    // Here we are grabbing the 'id' path variable from the $request object. WP\_REST\_Request implements ArrayAccess, which allows us to grab properties as though it is an array.
    $id = (string) $request\['id'\];

    if ( isset( $products\[ $id \] ) ) {
        // Grab the product.
        $product = $products\[ $id \];

        // Return the product as a response.
        return rest\_ensure\_response( $product );
    } else {
        // Return a WP\_Error because the request product was not found. In this case we return a 404 because the main resource was not found.
        return new WP\_Error( 'rest\_product\_invalid', esc\_html\_\_( 'The product does not exist.', 'my-text-domain' ), array( 'status' => 404 ) );
    }

    // If the code somehow executes to here something bad happened return a 500.
    return new WP\_Error( 'rest\_api\_sad', esc\_html\_\_( 'Something went horribly wrong.', 'my-text-domain' ), array( 'status' => 500 ) );
}

/\*\*
 \* This function is where we register our routes for our example endpoint.
 \*/
function prefix\_register\_product\_routes() {
    // Here we are registering our route for a collection of products.
    register\_rest\_route( 'my-shop/v1', '/products', array(
        // By using this constant we ensure that when the WP\_REST\_Server changes our readable endpoints will work as intended.
        'methods'  => WP\_REST\_Server::READABLE,
        // Here we register our callback. The callback is fired when this endpoint is matched by the WP\_REST\_Server class.
        'callback' => 'prefix\_get\_products',
    ) );
    // Here we are registering our route for single products. The (?P<id>\[\\d\]+) is our path variable for the ID, which, in this example, can only be some form of positive number.
    register\_rest\_route( 'my-shop/v1', '/products/(?P<id>\[\\d\]+)', array(
        // By using this constant we ensure that when the WP\_REST\_Server changes our readable endpoints will work as intended.
        'methods'  => WP\_REST\_Server::READABLE,
        // Here we register our callback. The callback is fired when this endpoint is matched by the WP\_REST\_Server class.
        'callback' => 'prefix\_get\_product',
    ) );
}

add\_action( 'rest\_api\_init', 'prefix\_register\_product\_routes' );

[Expand full source code](#)[Collapse full source code](#)

The above example covers a lot. The important part to note is that in the second route we register, we add on a path variable `/(?P[\d]+)` to our resource path `/products`. The path variable is a regular expression. In this case it uses `[\d]+` to signify that should be any numerical character at least once. If you are using numeric IDs for your resources, then this is a great example of how to use a path variable. When using path variables, we now have to be careful around what can be matched as it is user input.

The regex luckily will filter out anything that is not numerical. However, what if the product for the requested ID doesn’t exist. We need to do error handling. You can see the basic way we are handling errors in the code example above. When you return a `WP_Error` in your endpoint callbacks the API server will automatically handle serving the error to the client.

Although this section is about routes, we have covered quite a bit about endpoints. Endpoints and routes are interrelated, but they definitely have distinctions.

## Endpoints

Endpoints are the destination that a route needs to map to. For any given route, you could have a number of different endpoints registered to it. We will expand on our fictitious eCommerce plugin, to better show the distinction between routes and endpoints. We are going to create two endpoints that exist at the `/wp-json/my-shop/v1/products/` route. One endpoint uses the HTTP verb `GET` to get products, and the other endpoint uses the HTTP verb `POST` to create a new product.

/\*\*
 \* This is our callback function to return our products.
 \*
 \* @param WP\_REST\_Request $request This function accepts a rest request to process data.
 \*/
function prefix\_get\_products( $request ) {
    // In practice this function would fetch the desired data. Here we are just making stuff up.
    $products = array(
        '1' => 'I am product 1',
        '2' => 'I am product 2',
        '3' => 'I am product 3',
    );
    
    return rest\_ensure\_response( $products );
}

/\*\*
 \* This is our callback function to return a single product.
 \*
 \* @param WP\_REST\_Request $request This function accepts a rest request to process data.
 \*/
function prefix\_create\_product( $request ) {
    // In practice this function would create a product. Here we are just making stuff up.
   return rest\_ensure\_response( 'Product has been created' );
}

/\*\*
 \* This function is where we register our routes for our example endpoint.
 \*/
function prefix\_register\_product\_routes() {
    // Here we are registering our route for a collection of products and creation of products.
    register\_rest\_route( 'my-shop/v1', '/products', array(
        array(
            // By using this constant we ensure that when the WP\_REST\_Server changes, our readable endpoints will work as intended.
            'methods'  => WP\_REST\_Server::READABLE,
            // Here we register our callback. The callback is fired when this endpoint is matched by the WP\_REST\_Server class.
            'callback' => 'prefix\_get\_products',
        ),
        array(
            // By using this constant we ensure that when the WP\_REST\_Server changes, our create endpoints will work as intended.
            'methods'  => WP\_REST\_Server::CREATABLE,
            // Here we register our callback. The callback is fired when this endpoint is matched by the WP\_REST\_Server class.
            'callback' => 'prefix\_create\_product',
        ),
    ) );
}

add\_action( 'rest\_api\_init', 'prefix\_register\_product\_routes' );

[Expand full source code](#)[Collapse full source code](#)

Depending on what HTTP Method we use for the route `/wp-json/my-shop/v1/products`, we are matched to a different endpoint and a different callback is fired. When we use `POST` we trigger the `prefix_create_product()` callback, and when we use `GET` we trigger the `prefix_get_products()` callback.

There are a number of different HTTP methods and the REST API can make use of any of them.

### HTTP Methods

HTTP methods are sometimes referred to as HTTP verbs. They are simply just different ways to communicate via HTTP. The main ones used by the WordPress REST API are:

*   `GET` should be used for retrieving data from the API.
*   `POST` should be used for creating new resources (i.e users, posts, taxonomies).
*   `PUT` should be used for updating resources.
*   `DELETE` should be used for deleting resources.
*   `OPTIONS` should be used to provide context about our resources.

It is important to note that these methods are not supported by every client, as they were introduced in HTTP 1.1. Luckily, the API provides a workaround for these unfortunate cases. If you want to delete a resource but can’t send a `DELETE` request, then you can use the `_method` parameter or the `X-HTTP-Method-Override` header in your request. How this works is you will send a `POST` request to `https://ourawesomesite.com/wp-json/my-shop/v1/products/1?_method=DELETE`. Now you will have deleted product number 1, even though your client could not send the proper HTTP method in the request, or maybe there was a firewall in place that blocks out DELETE requests.

The HTTP method, in combination with the route and callbacks, are what make up the core of an endpoint.

### Callbacks

There are currently only two types of callbacks for endpoints supported by the REST API; `callback` and `permissions_callback`. The main callback should handle the interaction with the resource. The permissions callback should handle what users have access to the endpoint. You can add additional callbacks by adding additional information when registering an endpoint. You can then hook into `rest_pre_dispatch`, `rest_dispatch_request`, or `rest_post_dispatch` hooks to fire your new custom callbacks.

#### Endpoint Callback

The main callback for a delete endpoint should only delete the resource and return a copy of it in the response. The main callback for a creation endpoint should only create the resource and return a response matching the newly created data. An update callback should only modify resources that actually exist. A reading callback should only retrieve data that already exists. It is important to take into account the concept of idempotence.

Idempotence, in the context of a REST API, means that if you make the same request to an endpoint the server will process the request the same way. Imagine if our read endpoint was not idempotent. Whenever we made a request to it the state of our server would be modified by the request, even though we were only trying to get data. This could be catastrophic. Any time someone fetched data from your server something would change internally. It is important to make sure that read, update, and delete endpoints do not have nasty side effects and just stick to what they are intended to do.

In a REST API, the concept of idempotence is tied to HTTP methods instead of endpoint callbacks. Any callback using `GET`, `HEAD`, `TRACE`, `OPTIONS`, `PUT`, or `DELETE`, should not produce any side effects. `POST` requests are not idempotent, and are typically used for creating resources. If you created an idempotent creation method then you would only ever create one resource because when you make the same request there would be no more side effects to the server. For creating, if you make the same request over and over the server should generate new resources each time.

To restrict usage of endpoints we need to register a permissions callback.

#### Permissions Callback

Permissions callbacks are extremely important for security with the WordPress REST API. If you have any private data that should not be displayed publicly, then you need to have permissions callbacks registered for your endpoints. Below is an example of how to register permissions callbacks.

/\*\*
 \* This is our callback function that embeds our resource in a WP\_REST\_Response
 \*/
function prefix\_get\_private\_data() {
    // rest\_ensure\_response() wraps the data we want to return into a WP\_REST\_Response, and ensures it will be properly returned.
    return rest\_ensure\_response( 'This is private data.' );
}

/\*\*
 \* This is our callback function that embeds our resource in a WP\_REST\_Response
 \*/
function prefix\_get\_private\_data\_permissions\_check() {
    // Restrict endpoint to only users who have the edit\_posts capability.
    if ( ! current\_user\_can( 'edit\_posts' ) ) {
        return new WP\_Error( 'rest\_forbidden', esc\_html\_\_( 'OMG you can not view private data.', 'my-text-domain' ), array( 'status' => 401 ) );
    }

    // This is a black-listing approach. You could alternatively do this via white-listing, by returning false here and changing the permissions check.
    return true;
}

/\*\*
 \* This function is where we register our routes for our example endpoint.
 \*/
function prefix\_register\_example\_routes() {
    // register\_rest\_route() handles more arguments but we are going to stick to the basics for now.
    register\_rest\_route( 'my-plugin/v1', '/private-data', array(
        // By using this constant we ensure that when the WP\_REST\_Server changes our readable endpoints will work as intended.
        'methods'  => WP\_REST\_Server::READABLE,
        // Here we register our callback. The callback is fired when this endpoint is matched by the WP\_REST\_Server class.
        'callback' => 'prefix\_get\_private\_data',
        // Here we register our permissions callback. The callback is fired before the main callback to check if the current user can access the endpoint.
        'permissions\_callback' => 'prefix\_get\_private\_data\_permissions\_check',
    ) );
}

add\_action( 'rest\_api\_init', 'prefix\_register\_example\_routes' );

[Expand full source code](#)[Collapse full source code](#)

If you try out this endpoint without any Authentication enabled then you will also be returned the error response, preventing you from seeing the data. Authentication is a huge topic and eventually a portion of this chapter will be created to show you how to create your own authentication processes.

### Arguments

When making requests to an endpoint you might need to specify extra parameters to change the response. These extra parameters can be added while registering endpoints. Let’s look at an example of how to use arguments with an endpoint.

/\*\*
 \* This is our callback function that embeds our resource in a WP\_REST\_Response
 \*/
function prefix\_get\_colors( $request ) {
    // In practice this function would fetch the desired data. Here we are just making stuff up.
    $colors = array(
        'blue',
        'blue',
        'red',
        'red',
        'green',
        'green',
    );

    if ( isset( $request\['filter'\] ) ) {
       $filtered\_colors = array();
       foreach ( $colors as $color ) {
           if ( $request\['filter'\] === $color ) {
               $filtered\_colors\[\] = $color;
           }
       }
       return rest\_ensure\_response( $filtered\_colors );
    }
    return rest\_ensure\_response( $colors );
}

/\*\*
 \* We can use this function to contain our arguments for the example product endpoint.
 \*/
function prefix\_get\_color\_arguments() {
    $args = array();
    // Here we are registering the schema for the filter argument.
    $args\['filter'\] = array(
        // description should be a human readable description of the argument.
        'description' => esc\_html\_\_( 'The filter parameter is used to filter the collection of colors', 'my-text-domain' ),
        // type specifies the type of data that the argument should be.
        'type'        => 'string',
        // enum specified what values filter can take on.
        'enum'        => array( 'red', 'green', 'blue' ),
    );
    return $args;
}

/\*\*
 \* This function is where we register our routes for our example endpoint.
 \*/
function prefix\_register\_example\_routes() {
    // register\_rest\_route() handles more arguments but we are going to stick to the basics for now.
    register\_rest\_route( 'my-colors/v1', '/colors', array(
        // By using this constant we ensure that when the WP\_REST\_Server changes our readable endpoints will work as intended.
        'methods'  => WP\_REST\_Server::READABLE,
        // Here we register our callback. The callback is fired when this endpoint is matched by the WP\_REST\_Server class.
        'callback' => 'prefix\_get\_colors',
        // Here we register our permissions callback. The callback is fired before the main callback to check if the current user can access the endpoint.
        'args' => prefix\_get\_color\_arguments(),
    ) );
}

add\_action( 'rest\_api\_init', 'prefix\_register\_example\_routes' );

[Expand full source code](#)[Collapse full source code](#)

We have now specified a `filter` argument for this example. We can specify the argument as a query parameter when we request the endpoint. If we make a `GET` request to `https://ourawesomesitem.com/my-colors/v1/colors?filter=blue`, we will be returned only the blue colors in our collection. You could also pass these as body parameters in the request body, instead of in the query string. To understand the distinction between query parameters and body parameters you should read about the HTTP spec. Query parameters live in the query string tacked onto the URL and body parameters are directly embedded in the body of an HTTP request.

We have created an argument for our endpoint, but how do we verify that the argument is a string and tell whether it matches the value red, green, or blue. To do this we need to specify a validation callback for our argument.

#### Validation

Validation and sanitization are extremely important for security in the API. The validate callback (in WP 4.6+), fires before the sanitize callback. You should use the `validate_callback` for your arguments to verify whether the input you are receiving is valid. The `sanitize_callback` should be used to transform the argument input or clean out unwanted parts out of the argument, before the argument is processed by the main callback.

In the example above, we need to verify that the `filter` parameter is a string, and it matches the value red, green, or blue. Let’s look at what the code looks like after adding in a `validate_callback`.

/\*\*
 \* This is our callback function that embeds our resource in a WP\_REST\_Response
 \*/
function prefix\_get\_colors( $request ) {
    // In practice this function would fetch more practical data. Here we are just making stuff up.
    $colors = array(
        'blue',
        'blue',
        'red',
        'red',
        'green',
        'green',
    );

    if ( isset( $request\['filter'\] ) ) {
       $filtered\_colors = array();
       foreach ( $colors as $color ) {
           if ( $request\['filter'\] === $color ) {
               $filtered\_colors\[\] = $color;
           }
       }
       return rest\_ensure\_response( $filtered\_colors );
    }
    return rest\_ensure\_response( $colors );
}
/\*\*
 \* Validate a request argument based on details registered to the route.
 \*
 \* @param  mixed            $value   Value of the 'filter' argument.
 \* @param  WP\_REST\_Request  $request The current request object.
 \* @param  string           $param   Key of the parameter. In this case it is 'filter'.
 \* @return WP\_Error|boolean
 \*/
function prefix\_filter\_arg\_validate\_callback( $value, $request, $param ) {
    // If the 'filter' argument is not a string return an error.
    if ( ! is\_string( $value ) ) {
        return new WP\_Error( 'rest\_invalid\_param', esc\_html\_\_( 'The filter argument must be a string.', 'my-text-domain' ), array( 'status' => 400 ) );
    }

    // Get the registered attributes for this endpoint request.
    $attributes = $request->get\_attributes();

    // Grab the filter param schema.
    $args = $attributes\['args'\]\[ $param \];

    // If the filter param is not a value in our enum then we should return an error as well.
    if ( ! in\_array( $value, $args\['enum'\], true ) ) {
        return new WP\_Error( 'rest\_invalid\_param', sprintf( \_\_( '%s is not one of %s' ), $param, implode( ', ', $args\['enum'\] ) ), array( 'status' => 400 ) );
    }
}

/\*\*
 \* We can use this function to contain our arguments for the example product endpoint.
 \*/
function prefix\_get\_color\_arguments() {
    $args = array();
    // Here we are registering the schema for the filter argument.
    $args\['filter'\] = array(
        // description should be a human readable description of the argument.
        'description' => esc\_html\_\_( 'The filter parameter is used to filter the collection of colors', 'my-text-domain' ),
        // type specifies the type of data that the argument should be.
        'type'        => 'string',
        // enum specified what values filter can take on.
        'enum'        => array( 'red', 'green', 'blue' ),
        // Here we register the validation callback for the filter argument.
        'validate\_callback' => 'prefix\_filter\_arg\_validate\_callback',
    );
    return $args;
}

/\*\*
 \* This function is where we register our routes for our example endpoint.
 \*/
function prefix\_register\_example\_routes() {
    // register\_rest\_route() handles more arguments but we are going to stick to the basics for now.
    register\_rest\_route( 'my-colors/v1', '/colors', array(
        // By using this constant we ensure that when the WP\_REST\_Server changes our readable endpoints will work as intended.
        'methods'  => WP\_REST\_Server::READABLE,
        // Here we register our callback. The callback is fired when this endpoint is matched by the WP\_REST\_Server class.
        'callback' => 'prefix\_get\_colors',
        // Here we register our permissions callback. The callback is fired before the main callback to check if the current user can access the endpoint.
        'args' => prefix\_get\_color\_arguments(),
    ) );
}

add\_action( 'rest\_api\_init', 'prefix\_register\_example\_routes' );

[Expand full source code](#)[Collapse full source code](#)

#### Sanitizing

In the above example, we do not need to use a `sanitize_callback`, because we are restricting input to only values in our enum. If we did not have strict validation and accepted any string as a parameter, we would definitely need to register a `sanitize_callback`. What if we wanted to update a content field and the user entered something like `alert('ZOMG Hacking you');`. The field value could potentially be a executable script. To strip out unwanted data or to transform data into a desired format we need to register a `sanitize_callback` for our arguments. Here is an example of how to use WordPress’s `sanitize_text_field()` for a sanitize callback:

/\*\*
 \* This is our callback function that embeds our resource in a WP\_REST\_Response.
 \*
 \* The parameter is already sanitized by this point so we can use it without any worries.
 \*/
function prefix\_get\_item( $request ) {
    if ( isset( $request\['data'\] ) ) {
        return rest\_ensure\_response( $request\['data'\] );
    }

    return new WP\_Error( 'rest\_invalid', esc\_html\_\_( 'The data parameter is required.', 'my-text-domain' ), array( 'status' => 400 ) );
}

/\*\*
 \* Validate a request argument based on details registered to the route.
 \*
 \* @param  mixed            $value   Value of the 'filter' argument.
 \* @param  WP\_REST\_Request  $request The current request object.
 \* @param  string           $param   Key of the parameter. In this case it is 'filter'.
 \* @return WP\_Error|boolean
 \*/
function prefix\_data\_arg\_validate\_callback( $value, $request, $param ) {
    // If the 'data' argument is not a string return an error.
    if ( ! is\_string( $value ) ) {
        return new WP\_Error( 'rest\_invalid\_param', esc\_html\_\_( 'The filter argument must be a string.', 'my-text-domain' ), array( 'status' => 400 ) );
    }
}

/\*\*
 \* Sanitize a request argument based on details registered to the route.
 \*
 \* @param  mixed            $value   Value of the 'filter' argument.
 \* @param  WP\_REST\_Request  $request The current request object.
 \* @param  string           $param   Key of the parameter. In this case it is 'filter'.
 \* @return WP\_Error|boolean
 \*/
function prefix\_data\_arg\_sanitize\_callback( $value, $request, $param ) {
    // It is as simple as returning the sanitized value.
    return sanitize\_text\_field( $value );
}

/\*\*
 \* We can use this function to contain our arguments for the example product endpoint.
 \*/
function prefix\_get\_data\_arguments() {
    $args = array();
    // Here we are registering the schema for the filter argument.
    $args\['data'\] = array(
        // description should be a human readable description of the argument.
        'description' => esc\_html\_\_( 'The data parameter is used to be sanitized and returned in the response.', 'my-text-domain' ),
        // type specifies the type of data that the argument should be.
        'type'        => 'string',
        // Set the argument to be required for the endpoint.
        'required'    => true,
        // We are registering a basic validation callback for the data argument.
        'validate\_callback' => 'prefix\_data\_arg\_validate\_callback',
        // Here we register the validation callback for the filter argument.
        'sanitize\_callback' => 'prefix\_data\_arg\_sanitize\_callback',
    );
    return $args;
}

/\*\*
 \* This function is where we register our routes for our example endpoint.
 \*/
function prefix\_register\_example\_routes() {
    // register\_rest\_route() handles more arguments but we are going to stick to the basics for now.
    register\_rest\_route( 'my-plugin/v1', '/sanitized-data', array(
        // By using this constant we ensure that when the WP\_REST\_Server changes our readable endpoints will work as intended.
        'methods'  => WP\_REST\_Server::READABLE,
        // Here we register our callback. The callback is fired when this endpoint is matched by the WP\_REST\_Server class.
        'callback' => 'prefix\_get\_item',
        // Here we register our permissions callback. The callback is fired before the main callback to check if the current user can access the endpoint.
        'args' => prefix\_get\_data\_arguments(),
    ) );
}

add\_action( 'rest\_api\_init', 'prefix\_register\_example\_routes' );

[Expand full source code](#)[Collapse full source code](#)

## Summary

We have covered the basics of registering endpoints for the WordPress REST API. Routes are the URIs that our endpoints live at. Endpoints are a collection of callbacks, methods, args, and other options. Each endpoint is mapped to a route when using `register_rest_route()`. An endpoint by default can support various HTTP methods, a main callback, a permissions callback, and registered arguments. We can register endpoints to cover any of our use cases for interacting with WordPress. The endpoints serve as the core interaction point with the REST API, but there are many other topics to explore and understand, to fully utilize this powerful API.