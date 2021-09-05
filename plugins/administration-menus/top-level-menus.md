# Top-Level Menus

## Add a Top-Level Menu

To add a new Top-level menu to WordPress Administration, use the [add\_menu\_page()](https://developer.wordpress.org/reference/functions/add_menu_page/) function.

<?php
add\_menu\_page(
    string $page\_title,
    string $menu\_title,
    string $capability,
    string $menu\_slug,
    callable $function = '',
    string $icon\_url = '',
    int $position = null
);

### Example

Lets say we want to add a new Top-level menu called “WPOrg”.

**The first step** will be creating a function which will output the HTML. In this function we will perform the necessary security checks and render the options we’ve registered using the [Settings API](https://developer.wordpress.org/plugins/settings/).

Note:

We recommend wrapping your HTML using a `<div>` with a class of `wrap`.

<?php
function wporg\_options\_page\_html() {
    ?>
    <div class="wrap">
      <h1><?php echo esc\_html( get\_admin\_page\_title() ); ?></h1>
      <form action="options.php" method="post">
        <?php
        // output security fields for the registered setting "wporg\_options"
        settings\_fields( 'wporg\_options' );
        // output setting sections and their fields
        // (sections are registered for "wporg", each field is registered to a specific section)
        do\_settings\_sections( 'wporg' );
        // output save settings button
        submit\_button( \_\_( 'Save Settings', 'textdomain' ) );
        ?>
      </form>
    </div>
    <?php
}
?>

[Expand full source code](#)[Collapse full source code](#)

**The second step** will be registering our WPOrg menu. The registration needs to occur during the `admin_menu` action hook.

<?php
add\_action( 'admin\_menu', 'wporg\_options\_page' );
function wporg\_options\_page() {
    add\_menu\_page(
        'WPOrg',
        'WPOrg Options',
        'manage\_options',
        'wporg',
        'wporg\_options\_page\_html',
        plugin\_dir\_url(\_\_FILE\_\_) . 'images/icon\_wporg.png',
        20
    );
}
?>

[Expand full source code](#)[Collapse full source code](#)

For a list of parameters and what each do please see the [add\_menu\_page()](https://developer.wordpress.org/reference/functions/add_menu_page/) in the reference.

### Using a PHP File for HTML

The best practice for portable code would be to create a Callback that requires/includes your PHP file.

For the sake of completeness and helping you understand legacy code, we will show another way: passing a `PHP file path` as the `$menu_slug` parameter with an `null` `$function` parameter.

<?php
add\_action( 'admin\_menu', 'wporg\_options\_page' );
function wporg\_options\_page() {
    add\_menu\_page(
        'WPOrg',
        'WPOrg Options',
        'manage\_options',
        plugin\_dir\_path(\_\_FILE\_\_) . 'admin/view.php',
        null,
        plugin\_dir\_url(\_\_FILE\_\_) . 'images/icon\_wporg.png',
        20
    );
}
?>

[Expand full source code](#)[Collapse full source code](#)

## Remove a Top-Level Menu

To remove a registered menu from WordPress Administration, use the [remove\_menu\_page()](https://developer.wordpress.org/reference/functions/remove_menu_page/) function.

<?php
remove\_menu\_page(
    string $menu\_slug
);
?>

Warning:  
Removing menus won’t prevent users accessing them directly.  
This should never be used as a way to restrict [user capabilities](https://developer.wordpress.org/plugins/users/roles-and-capabilities/).

### Example

Lets say we want to remove the “Tools” menu from.

<?php
add\_action( 'admin\_menu', 'wporg\_remove\_options\_page', 99 );
function wporg\_remove\_options\_page() {
    remove\_menu\_page( 'tools.php' );
}
?>

Make sure that the menu have been registered with the `admin_menu` hook before attempting to remove, specify a higher priority number for [add\_action()](https://developer.wordpress.org/reference/functions/add_action/).

## Submitting forms

To process the submissions of forms on options pages, you will need two things:

1.  Use the URL of the page as the `action` attribute of the form.
2.  Add a hook with the slug, returned by `add_menu_page`.

Note:  
You only need to follow those steps if you are manually creating forms in the back-end. The [Settings API](https://developer.wordpress.org/plugins/settings/) is the recommended way to do this.

### Form action attribute

Use the `$menu_slug` parameter of the options page as the first parameter of `[menu_page_url()](https://developer.wordpress.org/reference/functions/menu_page_url/)`. By the function will automatically escape URL and echo it by default, so you can directly use it within the `<form>` tag:

<form action="<?php menu\_page\_url( 'wporg' ) ?>" method="post">

### Processing the form

The `$function` you specify while adding the page will only be called once it is time to display the page, which makes it inappropriate if you need to send headers (ex. redirects) back to the browser.

`add_menu_page` returns a `$hookname`, and WordPress triggers the `"load-$hookname"` action before any HTML output. You can use this to assign a function, which could process the form.

Note:  
`"load-$hookname"` will be executed every time before an options page will be displayed, even when the form is not being submitted.

With the return parameter and action in mind, the example from above would like this:

add\_action( 'admin\_menu', 'wporg\_options\_page' );
function wporg\_options\_page() {
	$hookname = add\_menu\_page(
		'WPOrg',
		'WPOrg Options',
		'manage\_options',
		'wporg',
		'wporg\_options\_page\_html',
		plugin\_dir\_url(\_\_FILE\_\_) . 'images/icon\_wporg.png',
		20
	);

	add\_action( 'load-' . $hookname, 'wporg\_options\_page\_submit' );
}

[Expand full source code](#)[Collapse full source code](#)

You can program `wporg_options_page_submit` according to your needs, but keep in mind that you must manually perform all necessary checks, including:

1.  Whether the form is being submitted (`'POST' === $_SERVER['REQUEST_METHOD']`).
2.  [CSRF verification](https://developer.wordpress.org/plugins/security/nonces/)
3.  Validation
4.  Sanitization