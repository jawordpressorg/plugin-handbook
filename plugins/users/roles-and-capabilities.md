# Roles and Capabilities

Roles and capabilities are two important aspects of WordPress that allow you to control user privileges.

WordPress stores the Roles and their Capabilities in the `options` table under the `user_roles` key.

## Roles

A role defines a set of capabilities for a user. For example, what the user may see and do in his dashboard.

**By default, WordPress have six roles:**

*   Super Admin
*   Administrator
*   Editor
*   Author
*   Contributor
*   Subscriber

More roles can be added and the default roles can be removed.

![](https://developer.wordpress.org/files/2014/09/wp-roles.png)

### Adding Roles

Add new roles and assign capabilities to them with [add\_role()](https://developer.wordpress.org/reference/functions/add_role/).

function wporg\_simple\_role() {
	add\_role(
		'simple\_role',
		'Simple Role',
		array(
			'read'         => true,
			'edit\_posts'   => true,
			'upload\_files' => true,
		),
	);
}

// Add the simple\_role.
add\_action( 'init', 'wporg\_simple\_role' );

[Expand full source code](#)[Collapse full source code](#)

Alert:  
After the first call to [add\_role()](https://developer.wordpress.org/reference/functions/add_role/), the Role and it’s Capabilities will be stored in the database!

Sequential calls will do nothing: including altering the capabilities list, which might not be the behavior that you’re expecting.

Note:  
To alter the capabilities list in bulk: remove the role using [remove\_role()](https://developer.wordpress.org/reference/functions/remove_role/) and add it again using [add\_role()](https://developer.wordpress.org/reference/functions/add_role/) with the new capabilities.

Make sure to do it only if the capabilities differ from what you’re expecting (i.e. condition this) or you’ll degrade performance considerably!

### Removing Roles

Remove roles with [remove\_role()](https://developer.wordpress.org/reference/functions/remove_role/).

function wporg\_simple\_role\_remove() {
	remove\_role( 'simple\_role' );
}

// Remove the simple\_role.
add\_action( 'init', 'wporg\_simple\_role\_remove' );

Alert:  
After the first call to [remove\_role()](https://developer.wordpress.org/reference/functions/remove_role/), the Role and it’s Capabilities will be removed from the database!

Sequential calls will do nothing.

Note:  
If you’re removing the default roles:

*   We advise **against** removing the Administrator and Super Admin roles!
*   Make sure to keep the code in your plugin/theme as future WordPress updates may add these roles again.
*   Run  
    `update_option('default_role', YOUR_NEW_DEFAULT_ROLE)`  
    since you’ll be deleting `subscriber` which is WP’s default role.

## Capabilities

Capabilities define what a **role** can and can not do: edit posts, publish posts, etc.

Note:  
Custom post types can require a certain set of Capabilities.

### Adding Capabilities

You may define new capabilities for a role.

Use [get\_role()](https://developer.wordpress.org/reference/functions/get_role/) to get the role object, then use the `add_cap()` method of that object to add a new capability.

function wporg\_simple\_role\_caps() {
	// Gets the simple\_role role object.
	$role = get\_role( 'simple\_role' );

	// Add a new capability.
	$role->add\_cap( 'edit\_others\_posts', true );
}

// Add simple\_role capabilities, priority must be after the initial role definition.
add\_action( 'init', 'wporg\_simple\_role\_caps', 11 );

Note:  
It’s possible to add custom capabilities to any role.

Under the default WordPress admin, they would have no effect, but they can be used for custom admin screen and front-end areas.

### Removing Capabilities

You may remove capabilities from a role.

The implementation is similar to Adding Capabilities with the difference being the use of `remove_cap()` method for the role object.

## Using Roles and Capabilities

### Get Role

Get the role object including all of it’s capabilities with [get\_role()](https://developer.wordpress.org/reference/functions/get_role/).

get\_role( $role );

### User Can

Check if a user have a specified **role** or **capability** with [user\_can()](https://developer.wordpress.org/reference/functions/user_can/).

user\_can( $user, $capability );

Warning:  
There is an undocumented, third argument, $args, that may include the object against which the test should be performed.

E.g. Pass a post ID to test for the capability of that specific post.

### Current User Can

[current\_user\_can()](https://developer.wordpress.org/reference/functions/current_user_can/) is a wrapper function for [user\_can()](https://developer.wordpress.org/reference/functions/user_can/) using the current user object as the `$user` parameter.

Use this in scenarios where back-end and front-end areas should require a certain level of privileges to access and/or modify.

current\_user\_can( $capability );

### Example

Here’s a practical example of adding an Edit link on the in a template file if the user has the proper capability:

if ( current\_user\_can( 'edit\_posts' ) ) {
	edit\_post\_link( esc\_html\_\_( 'Edit', 'wporg' ), '<p>', '</p>' );
}

## Multisite

The [current\_user\_can\_for\_blog()](https://developer.wordpress.org/reference/functions/current_user_can_for_blog/) function is used to test if the current user has a certain **role** or **capability** on a specific blog.

current\_user\_can\_for\_blog( $blog\_id, $capability );

## Reference

Codex Reference for [User Roles and Capabilities](https://wordpress.org/support/article/roles-and-capabilities/).