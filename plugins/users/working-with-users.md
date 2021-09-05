# Working with Users

## Adding Users

To add a user you can use `wp_create_user()` or `wp_insert_user()`.

`wp_create_user()` creates a user using only the username, password and email parameters while while `wp_insert_user()` accepts an array or object describing the user and its properties.

### Create User

`wp_create_user()` allows you to create a new WordPress user.

Note:  
It uses [wp\_slash()](https://developer.wordpress.org/reference/functions/wp_slash/) to escape the values. The PHP compact() function to create an array with these values. The [wp\_insert\_user()](https://developer.wordpress.org/reference/functions/wp_insert_user/) to perform the insert operation.

Please refer to the Function Reference about `wp_create_user()` for full explanation about the used parameters.

#### Example Create

</p>
// check if the username is taken
$user\_id = username\_exists( $user\_name );

// check that the email address does not belong to a registered user
if ( ! $user\_id && email\_exists( $user\_email ) === false ) {
	// create a random password
	$random\_password = wp\_generate\_password( 12, false );
	// create the user
	$user\_id = wp\_create\_user(
		$user\_name,
		$random\_password,
		$user\_email
	);
}
<p>

[Expand full source code](#)[Collapse full source code](#)

### Insert User

</p>
wp\_insert\_user( $userdata );
<p>

Note:  
The function calls a filter for most predefined properties.

The function performs the action `user_register` when creating a user (user ID does not exist).

The function performs the action `profile_update` when updating the user (user ID exists).

Please refer to the Function Reference about `wp_insert_user()` for full explanation about the used parameters.

#### Example Insert

Below is an example showing how to insert a new user with the website profile field filled in.

</p>
$username  = $\_POST\['username'\];
$password  = $\_POST\['password'\];
$website   = $\_POST\['website'\];
$user\_data = \[
	'user\_login' => $username,
	'user\_pass'  => $password,
	'user\_url'   => $website,
\];

$user\_id = wp\_insert\_user( $user\_data );

// success
if ( ! is\_wp\_error( $user\_id ) ) {
	echo 'User created: ' . $user\_id;
}
<p>

[Expand full source code](#)[Collapse full source code](#)

## Updating Users

`wp_update_user()` Updates a single user in the database. The update data is passed along in the `$userdata` array/object.

To update a single piece of user meta data, use `update_user_meta()` instead. To create a new user, use `wp_insert_user()` instead.

Note:  
If current user’s password is being updated, then the cookies will be cleared!

Please refer to the Function Reference about `wp_update_user()` for full explanation about the used parameters.

#### Example Update

Below is an example showing how to update a user’s website profile field.

</p>
$user\_id = 1;
$website = 'https://wordpress.org';

$user\_id = wp\_update\_user(
	array(
		'ID'       => $user\_id,
		'user\_url' => $website,
	)
);

if ( is\_wp\_error( $user\_id ) ) {
	// error
} else {
	// success
}
<p>

[Expand full source code](#)[Collapse full source code](#)

## Deleting Users

`wp_delete_user()` deletes the user and optionally reassign associated entities to another user ID.

Note:  
The function performs the action `deleted_user` after the user have been deleted.

Alert:  
If the $reassign parameter is not set to a valid user ID, then all entities belonging to the deleted user will be deleted!

Please refer to the Function Reference about `wp_delete_user()` for full explanation about the used parameters.