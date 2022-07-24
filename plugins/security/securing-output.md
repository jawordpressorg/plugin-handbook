# Securing (escaping) Output

Securing output is the process of *escaping* output data.

Escaping means stripping out unwanted data, like malformed HTML or script tags.

**Whenever you’re rendering data, make sure to properly escape it. Escaping output prevents XSS (Cross-site scripting) attacks.**

Note:  
[Cross-site scripting (XSS)](https://en.wikipedia.org/wiki/Cross-site_scripting) is a type of computer security vulnerability typically found in web applications. XSS enables attackers to inject client-side scripts into web pages viewed by other users. A cross-site scripting vulnerability may be used by attackers to bypass access controls such as the same-origin policy.

## Escaping

Escaping helps secure your data prior to rendering it for the end user. WordPress has many helper functions you can use for most common scenarios.

*   [esc\_attr()](https://developer.wordpress.org/reference/functions/esc_attr/) – Use on everything else that’s printed into an HTML element’s attribute.
*   [esc\_html()](https://developer.wordpress.org/reference/functions/esc_html/) – Use anytime an HTML element encloses a section of data being displayed. This **WILL NOT** display HTML as HTML (so `<strong>` would be output as is, instead of making something bold), it is meant for being used inside HTML and will remove your HTML.
*   [esc\_js()](https://developer.wordpress.org/reference/functions/esc_js/) – Use for inline Javascript, it escapes javascript for use in `<script>` tags.
*   [esc\_textarea()](https://developer.wordpress.org/reference/functions/esc_textarea/) – Use this to encode text for use inside a textarea element.
*   [esc\_url()](https://developer.wordpress.org/reference/functions/esc_url/) – Use on all URLs, including those in the `src` and `href` attributes of an HTML element.
*   [esc\_url\_raw()](https://developer.wordpress.org/reference/functions/esc_url_raw/) – Use when storing a URL in the database or in other cases where non-encoded URLs are needed.
*   [esc\_xml()](https://developer.wordpress.org/reference/functions/esc_xml/) – Use to escape XML block.
*   [wp\_kses()](https://developer.wordpress.org/reference/functions/wp_kses/) – Use to safely escape for all non-trusted HTML (post text, comment text, etc.). This **WILL** display HTML as HTML (so `<em>` will show as emphasized text)
*   [wp\_kses\_post()](https://developer.wordpress.org/reference/functions/wp_kses_post/) – Alternative version of `wp_kses()` that automatically allows all HTML that is permitted in post content.
*   [wp\_kses\_data()](https://developer.wordpress.org/reference/functions/wp_kses_data/) – Alternative version of `wp_kses()` that allows only the HTML permitted in post comments.

Pay close attention to what each function does, as some will *remove* HTML while others will permit it. You must use the *most* appropriate function to the content and context of what you’re echoing. You always want to escape **when** you echo, not before.

Alert:  
Most WordPress functions properly prepare data for output, so you don’t need to escape the data again. For example, you can safely call [the\_title()](https://developer.wordpress.org/reference/functions/the_title/) without escaping.

## Escaping with Localization

Rather than using `echo` to output data, it’s common to use the WordPress localization functions, such as `_e()` or `__()`.

These functions simply wrap a localization function inside an escaping function:

```php
esc_html_e( 'Hello World', 'text_domain' );
// Same as
echo esc_html( __( 'Hello World', 'text_domain' ) );
```

These helper functions combine localization and escaping:

*   [esc\_html\_\_()](https://developer.wordpress.org/reference/functions/esc_html__/)
*   [esc\_html\_e()](https://developer.wordpress.org/reference/functions/esc_html_e/)
*   [esc\_html\_x()](https://developer.wordpress.org/reference/functions/esc_html_x/)
*   [esc\_attr\_\_()](https://developer.wordpress.org/reference/functions/esc_attr__/)
*   [esc\_attr\_e()](https://developer.wordpress.org/reference/functions/esc_attr_e/)
*   [esc\_attr\_x()](https://developer.wordpress.org/reference/functions/esc_attr_x/)

## Custom Escaping

In the case that you need to escape your output in a specific way, the function [wp\_kses()](https://developer.wordpress.org/reference/functions/wp_kses/) (pronounced “kisses”) will come in handy.

This function makes sure that only the specified HTML elements, attributes, and attribute values will occur in your output, and normalizes HTML entities.

```php
$allowed_html = array(
	'a'      => array(
		'href'  => array(),
		'title' => array(),
	),
	'br'     => array(),
	'em'     => array(),
	'strong' => array(),
);
echo wp_kses( $custom_content, $allowed_html );

```

`wp_kses_post()` is a wrapper function for `wp_kses` where `$allowed_html` is a set of rules used by post content.