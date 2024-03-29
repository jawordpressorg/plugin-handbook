# Enclosing Shortcodes

The are two scenarios for using shortcodes:

*   The shortcode is a self-closing tag like we seen in the [Basic Shortcodes](https://developer.wordpress.org/plugins/shortcodes/basic-shortcodes/) section.
*   The shortcode is enclosing content.

## Enclosing Content

Enclosing content with a shortcode allows manipulations on the enclosed content.

```php
[wporg]content to manipulate[/wporg]
```

As seen above, all you need to do in order to enclose a section of content is add a beginning `[$tag]` and an end `[/$tag]`, similar to HTML.

## Processing Enclosed Content

Lets get back to our original \[wporg\] shortcode code:

```php
function wporg_shortcode( $atts = array(), $content = null ) {
    // do something to $content
    // always return
    return $content;
}
add_shortcode( 'wporg', 'wporg_shortcode' );
```

Looking at the callback function we see that we chose to accept two parameters, `$atts` and `$content`. The `$content` parameter is going to hold our enclosed content. We will talk about `$atts` later.

The default value of `$content` is set to `null` so we can differentiate between a self-closing tag and enclosing tags by using PHP function [is\_null()](http://php.net/is_null).

The shortcode `[$tag]`, including its content and the end `[/$tag]` will be replaced with the **return value** of the handler function.

Alert:  
It is the responsibility of the handler function to [secure the output](https://developer.wordpress.org/plugins/security/securing-output/).

## Shortcode-ception

The shortcode parser performs a **single pass** on the content of the post.

This means that if the `$content` parameter of a shortcode handler contains another shortcode, it won’t be parsed. In this example, `[shortcode]` will not be processed:

```php
[wporg]another [shortcode] is included[/wporg]
```

Using shortcodes inside other shortcodes is possible by calling `do_shortcode()` on the **final return value** of the handler function.

```php
function wporg_shortcode( $atts = array(), $content = null ) {
	// do something to $content
	// run shortcode parser recursively
	$content = do_shortcode( $content );
	// always return
	return $content;
}
add_shortcode( 'wporg', 'wporg_shortcode' );
```