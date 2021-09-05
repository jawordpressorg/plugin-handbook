# Advanced Topics

## Removing Actions and Filters

Sometimes you want to remove a callback function from a hook that another plugin, theme or even WordPress Core has registered.

To remove a callback function from a hook, you need to call `remove_action()` or `remove_filter()`, depending whether the callback function was added as an Action or a Filter.

The parameters passed to `remove_action()` / `remove_filter()` must be identical to the parameters passed to `add_action()` / `add_filter()` that registered it, or the removal won’t work.

Alert:  
To successfully remove a callback function you must perform the removal after the callback function was registered. The order of execution is important.

### Example

Lets say we want to improve the performance of a large theme by removing unnecessary functionality.

Let’s analyze the theme’s code by looking into `functions.php`.

</p>
function wporg\_setup\_slider() {
	// ...
}
add\_action( 'template\_redirect', 'wporg\_setup\_slider', 9 );
<p>

The `wporg_setup_slider` function is adding a slider that we don’t need, which probably loads a huge CSS file followed by a JavaScript initialization file which uses a custom written library the size of 1MB. We can can get rid of that.

Since we want to hook into WordPress after the `wporg_setup_slider` callback function was registered (`functions.php` executed) our best chance would be the `[after_setup_theme](https://developer.wordpress.org/reference/hooks/after_setup_theme/)` hook.

</p>
function wporg\_disable\_slider() {
	// Make sure all parameters match the add\_action() call exactly.
	remove\_action( 'template\_redirect', 'wporg\_setup\_slider', 9 );
}
// Make sure we call remove\_action() after add\_action() has been called.
add\_action( 'after\_setup\_theme', 'wporg\_disable\_slider' );
<p>

## Removing All Callbacks

You can also remove all of the callback functions associated with a hook by using `remove_all_actions()` / `remove_all_filters()`.

## Determining the Current Hook

Sometimes you want to run an Action or a Filter on multiple hooks, but behave differently based on which one is currently calling it.

You can use the `current_action()` / `current_filter()` to determine the current Action / Filter.

</p>
function wporg\_modify\_content( $content ) {
	switch ( current\_filter() ) {
		case 'the\_content':
			// Do something.
			break;
		case 'the\_excerpt':
			// Do something.
			break;
	}
	return $content;
}

add\_filter( 'the\_content', 'wporg\_modify\_content' );
add\_filter( 'the\_excerpt', 'wporg\_modify\_content' );
<p>

[Expand full source code](#)[Collapse full source code](#)

## Checking How Many Times a Hook Has Run

Some hooks are called multiple times in the course of execution, but you may only want your callback function to run once.

In this situation, you can check how many times the hook has run with the [did\_action()](https://developer.wordpress.org/reference/functions/did_action/).

function wporg\_custom() {  
   *// If save\_post has been run more than once, skip the rest of the code.  
*   if ( did\_action( 'save\_post' ) !== 1 ) {  
      return;  
   }  
   *// ...  
*}  
add\_action( 'save\_post', 'wporg\_custom' );

## Debugging with the “all” Hook

If you want a callback function to fire on every single hook, you can register it to the `all` hook. Sometimes this is useful in debugging situations to help determine when a particular event is happening or when a page is crashing.

</p>
function wporg\_debug() {
	echo '<p>' . current\_action() . '</p>';
}
add\_action( 'all', 'wporg\_debug' );
<p>