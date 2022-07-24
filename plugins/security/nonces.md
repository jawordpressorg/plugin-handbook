# Nonces

“Nonce” is a portmanteau of “*N*umber used *ONCE*”. It is essentially a unique hexadecimal serial number used to verify the origin and intent of requests for security purposes. As the name suggests, each nonce can only be used once.

If your plugin allows users to submit data; be it on the Admin or the Public side; you have to make sure that the user is who they say they are and that they [have the necessary capability](https://developer.wordpress.org/plugins/security/checking-user-capabilities/) to perform the action. Doing both in tandem means that data is only changing when the user *expects* it to be changing.

## Using Nonces

Following our [checking user capabilities example](https://developer.wordpress.org/plugins/security/checking-user-capabilities/#restricted-to-a-specific-capability), the next step in user data submission security is using nonces.

The capability check ensures that only users who have permission to delete a post are able to delete a post. But what if someone were to trick you into clicking that link? You have the necessary capability, so you could unwittingly delete a post.

Nonces can be used to check that the current user actually intends to perform the action.

When you generate the delete link, you’ll want to use [wp\_create\_nonce()](https://developer.wordpress.org/reference/functions/wp_create_nonce/) function to add a nonce to the link, the argument passed to the function ensures that the nonce being created is unique to that particular action.

Then, when you’re processing a request to delete a link, you can check that the nonce is what you expect it to be.

For more information, Mark Jaquith’s [post on WordPress nonces](http://markjaquith.wordpress.com/2006/06/02/wordpress-203-nonces/) is a great resource.

## Complete Example

Complete example using capability checks, data validation, secure input, secure output and nonces:

```php
/**
 * Generate a Delete link based on the homepage url.
 *
 * @param string $content   Existing content.
 *
 * @return string|null
 */
function wporg_generate_delete_link( $content ) {
	// Run only for single post page.
	if ( is_single() && in_the_loop() && is_main_query() ) {
		// Add query arguments: action, post, nonce
		$url = add_query_arg(
			[
				'action' => 'wporg_frontend_delete',
				'post'   => get_the_ID(),
				'nonce'  => wp_create_nonce( 'wporg_frontend_delete' ),
			], home_url()
		);

		return $content . ' <a href="' . esc_url( $url ) . '">' . esc_html__( 'Delete Post', 'wporg' ) . '</a>';
	}

	return null;
}


/**
 * Request handler
 */
function wporg_delete_post() {
	if ( isset( $_GET['action'] )
         && isset( $_GET['nonce'] )
         && 'wporg_frontend_delete' === $_GET['action']
         && wp_verify_nonce( $_GET['nonce'], 'wporg_frontend_delete' ) ) {

		// Verify we have a post id.
		$post_id = ( isset( $_GET['post'] ) ) ? ( $_GET['post'] ) : ( null );

		// Verify there is a post with such a number.
		$post = get_post( (int) $post_id );
		if ( empty( $post ) ) {
			return;
		}

		// Delete the post.
		wp_trash_post( $post_id );

		// Redirect to admin page.
		$redirect = admin_url( 'edit.php' );
		wp_safe_redirect( $redirect );

		// We are done.
		die;
	}
}


/**
 * Add delete post ability
 */
add_action('plugins_loaded', 'wporg_add_delete_post_ability');
 
function wporg_add_delete_post_ability() {    
    if ( current_user_can( 'edit_others_posts' ) ) {
        /**
         * Add the delete link to the end of the post content.
         */
        add_filter( 'the_content', 'wporg_generate_delete_link' );
      
        /**
         * Register our request handler with the init hook.
         */
        add_action( 'init', 'wporg_delete_post' );
    }
}
```