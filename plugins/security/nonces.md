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

</p>
/\*\*
 \* Generate a Delete link based on the homepage url.
 \*
 \* @param string $content   Existing content.
 \*
 \* @return string|null
 \*/
function wporg\_generate\_delete\_link( $content ) {
	// Run only for single post page.
	if ( is\_single() && in\_the\_loop() && is\_main\_query() ) {
		// Add query arguments: action, post, nonce
		$url = add\_query\_arg(
			\[
				'action' => 'wporg\_frontend\_delete',
				'post'   => get\_the\_ID(),
				'nonce'  => wp\_create\_nonce( 'wporg\_frontend\_delete' ),
			\], home\_url()
		);

		return $content . ' <a href="' . esc\_url( $url ) . '">' . esc\_html\_\_( 'Delete Post', 'wporg' ) . '</a>';
	}

	return null;
}


/\*\*
 \* Request handler
 \*/
function wporg\_delete\_post() {
	if ( isset( $\_GET\['action'\] )
         && isset( $\_GET\['nonce'\] )
         && 'wporg\_frontend\_delete' === $\_GET\['action'\]
         && wp\_verify\_nonce( $\_GET\['nonce'\], 'wporg\_frontend\_delete' ) ) {

		// Verify we have a post id.
		$post\_id = ( isset( $\_GET\['post'\] ) ) ? ( $\_GET\['post'\] ) : ( null );

		// Verify there is a post with such a number.
		$post = get\_post( (int) $post\_id );
		if ( empty( $post ) ) {
			return;
		}

		// Delete the post.
		wp\_trash\_post( $post\_id );

		// Redirect to admin page.
		$redirect = admin\_url( 'edit.php' );
		wp\_safe\_redirect( $redirect );

		// We are done.
		die;
	}
}


if ( current\_user\_can( 'edit\_others\_posts' ) ) {
	/\*\*
	 \* Add the delete link to the end of the post content.
	 \*/
	add\_filter( 'the\_content', 'wporg\_generate\_delete\_link' );

	/\*\*
	 \* Register our request handler with the init hook.
	 \*/
	add\_action( 'init', 'wporg\_delete\_post' );
}
<p>

[Expand full source code](#)[Collapse full source code](#)