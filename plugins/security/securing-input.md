# Securing (sanitizing) Input

Securing input is the process of *sanitizing* (cleaning, filtering) input data.

You use sanitizing when you don’t know what to expect or you don’t want to be strict with [data validation](https://developer.wordpress.org/plugins/security/data-validation/).

**Any time you’re accepting potentially unsafe data, it is important to validate or sanitize it.**

Remember: Even admins are users, and users will enter incorrect data on purpose or on accident. It’s your job to protect them from themselves.

## Sanitizing the Data

The easiest way to sanitize data is with built-in WordPress functions.

The `sanitize_*()` series of helper functions are super nice, as they ensure you’re ending up with safe data, and they require minimal effort on your part:

*   [sanitize\_email()](https://developer.wordpress.org/reference/functions/sanitize_email/)
*   [sanitize\_file\_name()](https://developer.wordpress.org/reference/functions/sanitize_file_name/)
*   [sanitize\_hex\_color()](https://developer.wordpress.org/reference/functions/sanitize_hex_color/)
*   [sanitize\_hex\_color\_no\_hash()](https://developer.wordpress.org/reference/functions/sanitize_hex_color_no_hash/)
*   [sanitize\_html\_class()](https://developer.wordpress.org/reference/functions/sanitize_html_class/)
*   [sanitize\_key()](https://developer.wordpress.org/reference/functions/sanitize_key/)
*   [sanitize\_meta()](https://developer.wordpress.org/reference/functions/sanitize_meta/)
*   [sanitize\_mime\_type()](https://developer.wordpress.org/reference/functions/sanitize_mime_type/)
*   [sanitize\_option()](https://developer.wordpress.org/reference/functions/sanitize_option/)
*   [sanitize\_sql\_orderby()](https://developer.wordpress.org/reference/functions/sanitize_sql_orderby/)
*   [sanitize\_text\_field()](https://developer.wordpress.org/reference/functions/sanitize_text_field/)
*   [sanitize\_textarea\_field()](https://developer.wordpress.org/reference/functions/sanitize_textarea_field/)
*   [sanitize\_title()](https://developer.wordpress.org/reference/functions/sanitize_title/)
*   [sanitize\_title\_for\_query()](https://developer.wordpress.org/reference/functions/sanitize_title_for_query/)
*   [sanitize\_title\_with\_dashes()](https://developer.wordpress.org/reference/functions/sanitize_title_with_dashes/)
*   [sanitize\_user()](https://developer.wordpress.org/reference/functions/sanitize_user/)
*   [esc\_url\_raw()](https://developer.wordpress.org/reference/functions/esc_url_raw/)
*   [wp\_kses()](https://developer.wordpress.org/reference/functions/wp_kses/)
*   [wp\_kses\_post()](https://developer.wordpress.org/reference/functions/wp_kses_post/)

## Example

Let’s say we have an input field named title.

<input id="title" type="text" name="title">

You can sanitize the input data with the [sanitize\_text\_field()](https://developer.wordpress.org/reference/functions/sanitize_text_field/) function:

$title = sanitize\_text\_field( $\_POST\['title'\] );
update\_post\_meta( $post->ID, 'title', $title );

Behind the scenes, `sanitize_text_field()` does the following:

*   Checks for invalid UTF-8
*   Converts single less-than characters (<) to entity
*   Strips all tags
*   Removes line breaks, tabs and extra white space
*   Strips octets