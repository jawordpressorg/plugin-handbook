# Responses

## Overview

Responses in the API are what holds all of the data we want. If we made a mistake in our request, our response’s data should also inform us that an error occurred. Responses in the WordPress REST API should return the data we requested or an error message. Responses in the API are handled by the `WP_REST_Response` class, one of the three infrastructural classes for the API.

## [WP\_REST\_Response](https://developer.wordpress.org/reference/classes/wp_rest_response/)

The `WP_REST_Response` extends WordPress’s `WP_HTTP_Response` class, allowing us access to response headers, response status code, and response data.

// The following code will not do anything and just serves as a demonstration.
$response = new WP\_REST\_Response( 'This is some data' );

// To get the response data we can use this method. It should equal 'This is some data'.
$our\_data = $response->get\_data();

// To access the HTTP status code we can use this method. The most common status code is probably 200, which means OK!
$our\_status = $response->get\_status();

// To access the HTTP response headers we can use this method.
$our\_headers = $response->get\_headers();

The above is pretty straightforward and shows you how to get what you need out of a response. The `WP_REST_Response` takes things a bit further. You can access the matched route for the response to backtrack which endpoint the response came from with `$response->get_matched_route()`. `$response->get_matched_handler()` will return the options registered for the endpoint that produced our response. These could be useful for logging the API among other things. The response class also helps us with error handling.

### Error Handling

If something went terribly wrong in our request, we can return `WP_Error` objects in our endpoint callbacks explaining what went wrong, like this:

// Register our mock batch endpoint.
function prefix\_register\_broken\_route() {
    register\_rest\_route( 'my-namespace/v1', '/broken', array(
        // Supported methods for this endpoint. WP\_REST\_Server::READABLE translates to GET.
        'methods' => WP\_REST\_Server::READABLE,
        // Register the callback for the endpoint.
        'callback' => 'prefix\_get\_an\_error',
    ) );
}

add\_action( 'rest\_api\_init', 'prefix\_register\_broken\_route' );

/\*\*
 \* Our registered endpoint callback. Notice how we are passing in $request as an argument.
 \* By default, the WP\_REST\_Server will pass in the matched request object to our callback.
 \*
 \* @param WP\_REST\_Request $request The current matched request object.
 \*/
function prefix\_get\_an\_error( $request ) {
    return new WP\_Error( 'oops', esc\_html\_\_( 'This endpoint is currently broken, try another endpoint, I promise the API is cool! EEEK!!!!', 'my-textdomain' ), array( 'status' => 400 ) );
}

[Expand full source code](#)[Collapse full source code](#)

That is kind of a silly example but it touches on some key things. The most important thing to understand is that the WordPress REST API will automatically handle changing the [WP\_Error](https://developer.wordpress.org/reference/classes/wp_error/) object into an HTTP Response containing your data. When you set the status code in the `WP_Error` object your HTTP response status code will take on that value. This comes in really handy when you need to use different error codes like 404 for content that wasn’t found, or 403 for forbidden access. All we have to do is have our endpoint callbacks return a request and the `WP_REST_Server` class will handle a lot of really important things for us.

There are other cool things the response class can help us with, like Linking.

## Linking

What if we wanted to get a post and the first comment for that post? Would we write a separate endpoint to handle this use case? If we did that, we would need to start adding a lot of endpoints to handle various small use cases and our API index would get bloated really fast. Response Linking provides us a way to form relations between our resources that the API can understand. The API implements a standard known as HAL for resource linking. Let’s look at our post and comment example, it would be better to have routes for each resource.

Let’s say we have post with ID = 1 and comment ID = 3. The comment is assigned to post 1, so realistically the two resources could live at the routes `/my-namespace/v1/posts/1` and `/my-namespace/v1/comments/3`. We would add links to the responses to create the relationships between them. Let’s look at this from the comment perspective first.

// Register our mock endpoints.
function prefix\_register\_my\_routes() {
    register\_rest\_route( 'my-namespace/v1', '/posts/(?P<id>\[\\d\]+)', array(
        // Supported methods for this endpoint. WP\_REST\_Server::READABLE translates to GET.
        'methods' => WP\_REST\_Server::READABLE,
        // Register the callback for the endpoint.
        'callback' => 'prefix\_get\_rest\_post',
    ) );
    register\_rest\_route( 'my-namespace/v1', '/comments', array(
        // Supported methods for this endpoint. WP\_REST\_Server::READABLE translates to GET.
        'methods' => WP\_REST\_Server::READABLE,
        // Register the callback for the endpoint.
        'callback' => 'prefix\_get\_rest\_comments',
        // Register the post argument to limit results to a specific post parent.
        'args' => array(
            'post' => array(
                'description' => esc\_html\_\_( 'The post ID that the comment is assigned to.', 'my-textdomain' ),
                'type'        => 'integer',
                'required'    => true,
            ),
        ),
    ) );
    register\_rest\_route( 'my-namespace/v1', '/comments/(?P<id>\[\\d\]+)', array(
        // Supported methods for this endpoint. WP\_REST\_Server::READABLE translates to GET.
        'methods' => WP\_REST\_Server::READABLE,
        // Register the callback for the endpoint.
        'callback' => 'prefix\_get\_rest\_comment',
    ) );
}

add\_action( 'rest\_api\_init', 'prefix\_register\_my\_routes' );

// Grab a post.
function prefix\_get\_rest\_post( $request ) {
    $id = (int) $request\['id'\];
    $post = get\_post( $id );

    $response = rest\_ensure\_response( array( $post ) );

    $response->add\_links( prefix\_prepare\_post\_links( $post ) );

    return $response;
}

// Prepare post links.
function prefix\_prepare\_post\_links( $post ) {
    $links = array();

    $replies\_url = rest\_url( 'my-namespace/v1/comments' );
    $replies\_url = add\_query\_arg( 'post', $post->ID, $replies\_url );
    $links\['replies'\] = array(
		'href'         => $replies\_url,
		'embeddable'   => true,
    );

    return $links;
}

// Grab a comments.
function prefix\_get\_rest\_comments( $request ) {
    if ( ! isset( $request\['post'\] ) ) {
        return new WP\_Error( 'rest\_bad\_request', esc\_html\_\_( 'You must specify the post parameter for this request.', 'my-text-domain' ), array( 'status' => 400 ) );
    }

    $data = array();

    $comments = get\_comments( array( 'post\_\_in' => $request\['post'\] ) );

    if ( empty( $comments ) ) {
        return array();
    }

    foreach( $comments as $comment ) {
        $response = rest\_ensure\_response( $comment );
        $response->add\_links( prefix\_prepare\_comment\_links( $comment ) );
        $data\[\] = prefix\_prepare\_for\_collection( $response );
    }

    $response = rest\_ensure\_response( $data );
    return $response;
}

// Grab a comment.
function prefix\_get\_rest\_comment( $request ) {
    $id = (int) $request\['id'\];
    $post = get\_comment( $id );

    $response = rest\_ensure\_response( $comment );

    $response->add\_links( prefix\_prepare\_comment\_links( $comment ) );

    return $response;
}

// Prepare comment links.
function prefix\_prepare\_comment\_links( $comment ) {
    $links = array();
    if ( 0 !== (int) $comment->comment\_post\_ID ) {
        $post = get\_post( $comment->comment\_post\_ID );
        if ( ! empty( $post->ID ) ) {
        $links\['up'\] = array(
                'href'       => rest\_url( 'my-namespace/v1/posts/' . $comment->comment\_post\_ID ),
                'embeddable' => true,
                'post\_type'  => $post->post\_type,
            );
        }
    }
    return $links;
}

/\*\*
 \* Prepare a response for inserting into a collection of responses.
 \*
 \* This is lifted from WP\_REST\_Controller class in the WP REST API v2 plugin.
 \*
 \* @param WP\_REST\_Response $response Response object.
 \* @return array Response data, ready for insertion into collection data.
 \*/
function prefix\_prepare\_for\_collection( $response ) {
	if ( ! ( $response instanceof WP\_REST\_Response ) ) {
		return $response;
	}

	$data = (array) $response->get\_data();
	$server = rest\_get\_server();

	if ( method\_exists( $server, 'get\_compact\_response\_links' ) ) {
		$links = call\_user\_func( array( $server, 'get\_compact\_response\_links' ), $response );
	} else {
		$links = call\_user\_func( array( $server, 'get\_response\_links' ), $response );
	}

	if ( ! empty( $links ) ) {
		$data\['\_links'\] = $links;
	}

	return $data;
}

[Expand full source code](#)[Collapse full source code](#)

As you can see in the example above we are using links to create the relations between our resources. If the post has comments, our endpoint callback will add a link to the comments route specifying the \`post\` parameter to match our current post ID. So if you were to follow that route you would now get the comments that have that assigned post ID. If you search for comments then each comment will have a link point \`up\` to the post. \`up\` has special meaning in links using the HAL spec. If we follow an up link for a comment then we will be returned the post that is the comment parent. Linking is pretty awesome, but it gets better.

The WordPress REST API also supports what is referred to as embedding. If you notice in both of the links we added, we specified that `embeddable => true`. This enables us to embed our linked data in our responses. So if we wanted to grab comment 3 and its assigned post we could make this request `https://ourawesomesite.com/wp-json/my-namespace/v1/comments/3?_embed`. The `_embed` parameter tells the API we want all of the embeddable resource links for our request to also be added to the API. Using embed is a performance gain as the multiple resources are all handled in one HTTP Request.

Smart use of embedding and links make the WordPress REST API incredibly flexible and powerful for interacting with WordPress.