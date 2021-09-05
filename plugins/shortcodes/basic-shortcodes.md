# Basic Shortcodes

## Add a Shortcode

It is possible to add your own shortcodes by using the Shortcode API. The process involves registering a callback `$func` to a shortcode `$tag` using `add_shortcode()`.

</p>
add\_shortcode(
    string $tag,
    callable $func
);
<p>

`[wporg]` is your new shortcode. The use of the shortcode will trigger the `wporg_shortcode` callback function.

</p>
add\_shortcode('wporg', 'wporg\_shortcode');
function wporg\_shortcode( $atts = \[\], $content = null) {
    // do something to $content
    // always return
    return $content;
}
<p>

## Remove a Shortcode

It is possible to remove shortcodes by using the Shortcode API. The process involves removing a registered `$tag` using [remove\_shortcode()](https://developer.wordpress.org/reference/functions/remove_shortcode/).

</p>
remove\_shortcode(
    string $tag
);
<p>

Make sure that the shortcode have been registered before attempting to remove. Specify a higher priority number for [add\_action()](https://developer.wordpress.org/reference/functions/add_action/) or hook into an action hook that is run later.

## Check if a Shortcode Exists

To check whether a shortcode has been registered use `shortcode_exists()`.