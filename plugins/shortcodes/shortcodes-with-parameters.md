# Shortcodes with Parameters

Now that we know how to create a [basic shortcode](https://developer.wordpress.org/plugins/shortcodes/basic-shortcodes/) and how to use it as [self-closing and enclosing](https://developer.wordpress.org/plugins/shortcodes/enclosing-shortcodes/), we will look at using parameters in shortcode `[$tag]` and handler function.

Shortcode `[$tag]` can accept parameters, known as attributes:

\[wporg title="WordPress.org"\]
Having fun with WordPress.org shortcodes.
\[/wporg\]

Shortcode handler function can accept 3 parameters:

*   `$atts` – array – \[$tag\] attributes
*   `$content` – string – post content
*   `$tag` – string – the name of the \[$tag\] (i.e. the name of the shortcode)

function wporg\_shortcode( $atts = array(), $content = null, $tag = '' ) {}

## Parsing Attributes

For the user, shortcodes are just strings with square brackets inside the post content. The user have no idea which attributes are available and what happens behind the scenes.

For plugin developers, there is no way to enforce a policy on the use of attributes. The user may include one attribute, two or none at all.

To gain control of how the shortcodes are used:

*   Declare default parameters for the handler function
*   Performing normalization of the key case for the attributes array with [array\_change\_key\_case()](http://php.net/manual/en/function.array-change-key-case.php)
*   Parse attributes using [shortcode\_atts()](https://developer.wordpress.org/reference/functions/shortcode_atts/) providing default values array and user `$atts`
*   [Secure the output](https://developer.wordpress.org/plugins/security/securing-output/) before returning it

## Complete Example

Complete example using a basic shortcode structure, taking care of self-closing and enclosing scenarios, shortcodes within shortcodes and securing output.

A `[wporg]` shortcode that will accept a title and will display a box that we can style with CSS.

/\*\*
 \* The \[wporg\] shortcode.
 \*
 \* Accepts a title and will display a box.
 \*
 \* @param array  $atts    Shortcode attributes. Default empty.
 \* @param string $content Shortcode content. Default null.
 \* @param string $tag     Shortcode tag (name). Default empty.
 \* @return string Shortcode output.
 \*/
function wporg\_shortcode( $atts = \[\], $content = null, $tag = '' ) {
	// normalize attribute keys, lowercase
	$atts = array\_change\_key\_case( (array) $atts, CASE\_LOWER );

	// override default attributes with user attributes
	$wporg\_atts = shortcode\_atts(
		array(
			'title' => 'WordPress.org',
		), $atts, $tag
	);

	// start box
	$o = '<div class="wporg-box">';

	// title
	$o .= '<h2>' . esc\_html\_\_( $wporg\_atts\['title'\], 'wporg' ) . '</h2>';

	// enclosing tags
	if ( ! is\_null( $content ) ) {
		// secure output by executing the\_content filter hook on $content
		$o .= apply\_filters( 'the\_content', $content );

		// run shortcode parser recursively
		$o .= do\_shortcode( $content );
	}

	// end box
	$o .= '</div>';

	// return output
	return $o;
}

/\*\*
 \* Central location to create all shortcodes.
 \*/
function wporg\_shortcodes\_init() {
	add\_shortcode( 'wporg', 'wporg\_shortcode' );
}

add\_action( 'init', 'wporg\_shortcodes\_init' );

[Expand full source code](#)[Collapse full source code](#)