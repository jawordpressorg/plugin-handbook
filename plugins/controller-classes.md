# Controller Classes

## Overview

When writing endpoints it can be helpful to use a controller class to handle the functionality of an endpoint. Controller classes will provide a standard way to interact with the API and also a more maintainable way to interact with the API. WordPress’s current minimum PHP version is 5.2, if you are developing endpoints that will be used by the WordPress ecosystem at large you should consider supporting WordPress’s minimum requirements.

PHP 5.2 does not have namespacing built into it. This means that every function you declare will be in the global scope. If you decide to use a common function name for endpoints like `get_items()` and another plugin also registers that function, PHP will fail with a fatal error. This is because the function `get_items()` is being declared twice. By wrapping our endpoints we can avoid these naming conflicts and also have a consistent way to interact with the API.

## Controllers

Controllers typically do one thing; they receive input, and generate output. For the WordPress REST API our controllers will handle request input as `WP_REST_Request` objects and generate response output as `WP_REST_Response` objects. Let’s look at an example controller class:

class My\_REST\_Posts\_Controller {

	// Here initialize our namespace and resource name.
	public function \_\_construct() {
		$this->namespace     = '/my-namespace/v1';
		$this->resource\_name = 'posts';
	}

	// Register our routes.
	public function register\_routes() {
		register\_rest\_route( $this->namespace, '/' . $this->resource\_name, array(
			// Here we register the readable endpoint for collections.
			array(
				'methods'   => 'GET',
				'callback'  => array( $this, 'get\_items' ),
				'permission\_callback' => array( $this, 'get\_items\_permissions\_check' ),
			),
			// Register our schema callback.
			'schema' => array( $this, 'get\_item\_schema' ),
		) );
		register\_rest\_route( $this->namespace, '/' . $this->resource\_name . '/(?P<id>\[\\d\]+)', array(
			// Notice how we are registering multiple endpoints the 'schema' equates to an OPTIONS request.
			array(
				'methods'   => 'GET',
				'callback'  => array( $this, 'get\_item' ),
				'permission\_callback' => array( $this, 'get\_item\_permissions\_check' ),
			),
			// Register our schema callback.
			'schema' => array( $this, 'get\_item\_schema' ),
		) );
	}

	/\*\*
	 \* Check permissions for the posts.
	 \*
	 \* @param WP\_REST\_Request $request Current request.
	 \*/
	public function get\_items\_permissions\_check( $request ) {
		if ( ! current\_user\_can( 'read' ) ) {
			return new WP\_Error( 'rest\_forbidden', esc\_html\_\_( 'You cannot view the post resource.' ), array( 'status' => $this->authorization\_status\_code() ) );
		}
		return true;
	}

	/\*\*
	 \* Grabs the five most recent posts and outputs them as a rest response.
	 \*
	 \* @param WP\_REST\_Request $request Current request.
	 \*/
	public function get\_items( $request ) {
		$args = array(
			'post\_per\_page' => 5,
		);
		$posts = get\_posts( $args );

		$data = array();

		if ( empty( $posts ) ) {
			return rest\_ensure\_response( $data );
		}

		foreach ( $posts as $post ) {
			$response = $this->prepare\_item\_for\_response( $post, $request );
			$data\[\] = $this->prepare\_response\_for\_collection( $response );
		}

		// Return all of our comment response data.
		return rest\_ensure\_response( $data );
	}

	/\*\*
	 \* Check permissions for the posts.
	 \*
	 \* @param WP\_REST\_Request $request Current request.
	 \*/
	public function get\_item\_permissions\_check( $request ) {
		if ( ! current\_user\_can( 'read' ) ) {
			return new WP\_Error( 'rest\_forbidden', esc\_html\_\_( 'You cannot view the post resource.' ), array( 'status' => $this->authorization\_status\_code() ) );
		}
		return true;
	}

	/\*\*
	 \* Grabs the five most recent posts and outputs them as a rest response.
	 \*
	 \* @param WP\_REST\_Request $request Current request.
	 \*/
	public function get\_item( $request ) {
		$id = (int) $request\['id'\];
		$post = get\_post( $id );

		if ( empty( $post ) ) {
			return rest\_ensure\_response( array() );
		}

		$response = prepare\_item\_for\_response( $post );

		// Return all of our post response data.
		return $response;
	}

	/\*\*
	 \* Matches the post data to the schema we want.
	 \*
	 \* @param WP\_Post $post The comment object whose response is being prepared.
	 \*/
	public function prepare\_item\_for\_response( $post, $request ) {
		$post\_data = array();

		$schema = $this->get\_item\_schema( $request );

		// We are also renaming the fields to more understandable names.
		if ( isset( $schema\['properties'\]\['id'\] ) ) {
			$post\_data\['id'\] = (int) $post->ID;
		}

		if ( isset( $schema\['properties'\]\['content'\] ) ) {
			$post\_data\['content'\] = apply\_filters( 'the\_content', $post->post\_content, $post );
		}

		return rest\_ensure\_response( $post\_data );
	}

	/\*\*
	 \* Prepare a response for inserting into a collection of responses.
	 \*
	 \* This is copied from WP\_REST\_Controller class in the WP REST API v2 plugin.
	 \*
	 \* @param WP\_REST\_Response $response Response object.
	 \* @return array Response data, ready for insertion into collection data.
	 \*/
	public function prepare\_response\_for\_collection( $response ) {
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

	/\*\*
	 \* Get our sample schema for a post.
	 \*
	 \* @param WP\_REST\_Request $request Current request.
	 \*/
	public function get\_item\_schema( $request ) {
		$schema = array(
			// This tells the spec of JSON Schema we are using which is draft 4.
			'$schema'              => 'http://json-schema.org/draft-04/schema#',
			// The title property marks the identity of the resource.
			'title'                => 'post',
			'type'                 => 'object',
			// In JSON Schema you can specify object properties in the properties attribute.
			'properties'           => array(
				'id' => array(
					'description'  => esc\_html\_\_( 'Unique identifier for the object.', 'my-textdomain' ),
					'type'         => 'integer',
					'context'      => array( 'view', 'edit', 'embed' ),
					'readonly'     => true,
				),
				'content' => array(
					'description'  => esc\_html\_\_( 'The content for the object.', 'my-textdomain' ),
					'type'         => 'string',
				),
			),
		);

		return $schema;
	}

	// Sets up the proper HTTP status code for authorization.
	public function authorization\_status\_code() {

		$status = 401;

		if ( is\_user\_logged\_in() ) {
			$status = 403;
		}

		return $status;
	}
}

// Function to register our new routes from the controller.
function prefix\_register\_my\_rest\_routes() {
	$controller = new My\_REST\_Posts\_Controller();
	$controller->register\_routes();
}

add\_action( 'rest\_api\_init', 'prefix\_register\_my\_rest\_routes' );

[Expand full source code](#)[Collapse full source code](#)

## Overview & The Future

Controller classes tackle two big problems for us while developing endpoints; lack of namespacing and consistent structures. It is important to note that you should not abuse inheritance of your endpoints. For example: if you wrote a controller class for a posts endpoint, like the above example, and wanted to support custom post types as well, you should **NOT** extend your `My_REST_Posts_Controller` like this `class My_CPT_REST_Controller extends My_REST_Posts_Controller`.

Instead you should either create an entirely separate controller class or make `My_REST_Posts_Controller` handle all available post types. When you start go down the dark chasm of inheritance, it is important to understand that if the parent classes ever have to change at any point and your subclasses are dependent on them, you will have a major headache. In most cases, you will want to create a base controller class as either an `interface` or `abstract class`, that each of your endpoint controllers can implement or extend. The `abstract class` approach is being taken by the WP REST API team for the potential inclusion to core for the `WP_REST_Controller` class.

Currently, “core endpoints” supporting posts, post types, post statuses, revisions, taxonomies, terms, users, comments, and attachments/media resources, are being developed in a feature plugin that will hopefully be moved into WordPress core at some point. Within the plugin is a proposed `WP_REST_Controller` class that can be used to build your own controllers for your endpoints. `WP_REST_Controller` features a lot of advantages and a consistent way to create endpoints for the API.