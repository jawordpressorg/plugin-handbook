# Activation / Deactivation Hooks

Activation and deactivation hooks provide ways to perform actions when plugins are activated or deactivated.

*   On *activation*, plugins can run a routine to add rewrite rules, add custom database tables, or set default option values.
*   On *deactivation*, plugins can run a routine to remove temporary data such as cache and temp files and directories.

Alert:  
The deactivation hook is sometimes confused with the [uninstall hook](https://developer.wordpress.org/plugins/the-basics/uninstall-methods/). The uninstall hook is best suited to **delete all data permanently** such as deleting plugin options and custom tables, etc. Read more about the uninstall hooks [here](https://developer.wordpress.org/plugins/plugin-basics/uninstall-methods/).  

## Activation

To set up an activation hook, use the [register\_activation\_hook()](https://developer.wordpress.org/reference/functions/register_activation_hook/) function:

register\_activation\_hook( \_\_FILE\_\_, 'pluginprefix\_function\_to\_run' );

## Deactivation

To set up a deactivation hook, use the [register\_deactivation\_hook()](https://developer.wordpress.org/reference/functions/register_deactivation_hook/) function:

register\_deactivation\_hook( \_\_FILE\_\_, 'pluginprefix\_function\_to\_run' );

The first parameter in each of these functions refers to your main plugin file, which is the file in which you have placed the [plugin header comment](https://developer.wordpress.org/plugins/the-basics/header-requirements/). Usually these two functions will be triggered from within the main plugin file; however, if the functions are placed in any other file, you must update the first parameter to correctly point to the main plugin file.

## Example

One of the most common uses for an activation hook is to refresh WordPress permalinks when a plugin registers a custom post type. This gets rid of the nasty 404 errors.

Let’s look at an example of how to do this:

/\*\*
 \* Register the "book" custom post type
 \*/
function pluginprefix\_setup\_post\_type() {
	register\_post\_type( 'book', \['public' => true \] ); 
} 
add\_action( 'init', 'pluginprefix\_setup\_post\_type' );


/\*\*
 \* Activate the plugin.
 \*/
function pluginprefix\_activate() { 
	// Trigger our function that registers the custom post type plugin.
	pluginprefix\_setup\_post\_type(); 
	// Clear the permalinks after the post type has been registered.
	flush\_rewrite\_rules(); 
}
register\_activation\_hook( \_\_FILE\_\_, 'pluginprefix\_activate' );

[Expand full source code](#)[Collapse full source code](#)

If you are unfamiliar with registering custom post types, don’t worry – this will be covered later. This example is used simply because it’s very common.

Using the example from above, the following is how to reverse this process and deactivate a plugin:

/\*\*
 \* Deactivation hook.
 \*/
function pluginprefix\_deactivate() {
	// Unregister the post type, so the rules are no longer in memory.
	unregister\_post\_type( 'book' );
	// Clear the permalinks to remove our post type's rules from the database.
	flush\_rewrite\_rules();
}
register\_deactivation\_hook( \_\_FILE\_\_, 'pluginprefix\_deactivate' );

For further information regarding activation and deactivation hooks, here are some excellent resources:

*   [register\_activation\_hook()](https://developer.wordpress.org/reference/functions/register_activation_hook/) in the WordPress function reference.
*   [register\_deactivation\_hook()](https://developer.wordpress.org/reference/functions/register_deactivation_hook/) in the WordPress function reference.