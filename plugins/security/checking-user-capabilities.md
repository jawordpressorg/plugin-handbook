# Checking User Capabilities

If your plugin allows users to submit data—be it on the Admin or the Public side—it should check for User Capabilities.

## User Roles and Capabilities

The most important step in creating an efficient security layer is having a user permission system in place. WordPress provides this in the form of [User Roles and Capabilities](https://developer.wordpress.org/plugins/users/roles-and-capabilities/).

Every user logged into WordPress is automatically assigned specific User capabilities depending on their User role.

**User roles** is just a fancy way of saying which group the user belongs to. Each group has a specific set of predefined capabilities.

For example, the main user of your website will have the User role of an Administrator while other users might have roles like Editor or Author. You could have more than one user assigned to a role, i.e. there might be two Administrators for a website.

**User capabilities** are the specific permissions that you assign to each user or to a User role.

For example, Administrators have the “manage\_options” capability which allows them to view, edit and save options for the website. Editors on the other hand lack this capability which will prevent them from interacting with options.

These capabilities are then checked at various points within the Admin. Depending on the capabilities assigned to a role; menus, functionality, and other aspects of the WordPress experience may be added or removed.

**As you build a plugin, make sure to run your code only when the current user has the necessary capabilities.**

### Hierarchy

The higher the user role, the more capabilities the user has. Each user role inherits the previous roles in the hierarchy.

For example, the “Administrator”, which is the highest user role on a single site installation, inherits the following roles and their capabilities: “Subscriber”, “Contributor”, “Author” and “Editor”.

## Examples

### No Restrictions

The example below creates a link on the frontend which gives the ability to trash posts. Because this code does not check user capabilities, **it allows any visitor to the site to trash posts!**

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
		// Add query arguments: action, post.
		$url = add\_query\_arg(
			\[
				'action' => 'wporg\_frontend\_delete',
				'post'   => get\_the\_ID(),
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
	if ( isset( $\_GET\['action'\] ) && 'wporg\_frontend\_delete' === $\_GET\['action'\] ) {

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


/\*\*
 \* Add the delete link to the end of the post content.
 \*/
add\_filter( 'the\_content', 'wporg\_generate\_delete\_link' );

/\*\*
 \* Register our request handler with the init hook.
 \*/
add\_action( 'init', 'wporg\_delete\_post' );

[Expand full source code](#)[Collapse full source code](#)

### Restricted to a Specific Capability

The example above allows any visitor to the site to click on the “Delete” link and trash the post. However, we only want Editors and above to be able to click on the “Delete” link.

To accomplish this, we will check that the current user has the capability `edit_others_posts`, which only Editors or above would have:

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
		// Add query arguments: action, post.
		$url = add\_query\_arg(
			\[
				'action' => 'wporg\_frontend\_delete',
				'post'   => get\_the\_ID(),
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
	if ( isset( $\_GET\['action'\] ) && 'wporg\_frontend\_delete' === $\_GET\['action'\] ) {

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

[Expand full source code](#)[Collapse full source code](#)