# Server Side PHP and Enqueuing

There are two parts to the server side PHP script that are needed to implement AJAX communication. First we need to enqueue the jQuery script on the web page and localize any PHP values that the jQuery script needs. Second is the actual handling of the AJAX request.

## Enqueue Script

This section covers the two major quirks of AJAX in WordPress that trip up experienced coders new to WordPress. One is the need to enqueue scripts in order to get meta links to appear correctly in the page’s head section. The other is that **all** AJAX requests need to be sent through `wp-admin/admin-ajax.php`. Never send requests directly to your plugin pages.

### Enqueue

Use the function `[wp_enqueue_script()](https://developer.wordpress.org/reference/functions/wp_enqueue_script/)` to get WordPress to insert a meta link to your script in the page’s section. Never hardcode such links in the header template. As a plugin developer, you do not have ready access to the header template, but this rule bears mentioning anyway.

The enqueue function takes three parameters. The first is an arbitrary tag or handle that is used to refer to your script in other functions. The second is the complete URL to your script file. For portability, use `plugins_url()` to build the proper URL. If you are enqueuing the script for something besides a plugin, use some related function to create a proper URL – never hardcode it. The third parameter is an array of any script tags that your script is dependent on. Since we are using jQuery to send an AJAX request, you will at least need to list *‘jquery’* in the array. Always use an array even if it is for a single dependency. The enqueue call for our example looks like this:

</p>
wp\_enqueue\_script(
	'ajax-script',
	plugins\_url( '/js/myjquery.js', \_\_FILE\_\_ ),
	array( 'jquery' ),
	'1.0.,0',
	true
);
<p>

You cannot enqueue scripts directly from your plugin code page when it is loaded. Scripts must be enqueued from one of a few action hooks – which one depends on what sort of page the script needs to be linked to. For administration pages, use `admin_enqueue_scripts`. For front-end pages use `wp_enqueue_scripts`, except for the login page, in which case use `login_enqueue_scripts`.

The `admin_enqueue_scripts` hook passes the current page filename to your callback. Use this information to only enqueue your script on pages where it is needed. The front-end version does not pass anything. In that case, use template tags such as `is_home()`, `is_single()`, etc. to ensure that you only enqueue your script where it is needed. This is the complete enqueue code for our example:

</p>
add\_action( 'admin\_enqueue\_scripts', 'my\_enqueue' );
function my\_enqueue( $hook ) {
	if ( 'myplugin\_settings.php' !== $hook ) {
		return;
	}
	wp\_enqueue\_script(
		'ajax-script',
		plugins\_url( '/js/myjquery.js', \_\_FILE\_\_ ),
		array( 'jquery' ),
		'1.0.0',
		true
	);
}
<p>

[Expand full source code](#)[Collapse full source code](#)

Why do we use a named function here but use anonymous functions with jQuery? Because closures are only recently supported by PHP. jQuery has supported them for quite some time. Since some people may still be running older versions of PHP, we always use named functions for maximum compatibility. If you have a recent PHP version and are developing only for your own installation, go ahead and use closures if you like.

#### Register vs. Enqueue

You will see examples in other tutorials that religiously use `[wp_register_script()](https://developer.wordpress.org/reference/functions/wp_register_script/)`. This is fine, but its use is optional. What is not optional is `wp_enqueue_script()`. This function must be called in order for your script file to be properly linked on the web page. So why register scripts? It creates a useful tag or handle with which you can easily reference the script in various parts of your code as needed. If you just need your script loaded and are not referencing it elsewhere in your code, there is no need to register it.

### Nonce

You need to create a nonce so that the jQuery AJAX request can be validated as a legitimate request instead of a potentially nefarious request from some unknown bad actor. Only your PHP script and your jQuery script will know this value. When the request is received, you can verify it is the same value created here. This is how to create a nonce for our example:

</p>
$title\_nonce = wp\_create\_nonce( 'title\_example' );
<p>

The parameter `title_example` can be any arbitrary string. It’s suggested the string be related to what the nonce is used for, but it can really be anything that suits you.

### Localize

If you recall from the [jQuery Section](https://developer.wordpress.org/plugins/javascript/jquery/), data created by PHP for use by jQuery was passed in a global object named `my_ajax_obj`. In our example, this data was a nonce and the complete URL to `admin-ajax.php`. The process of assigning object properties and creating the global jQuery object is called **localizing**. This is the localizing code used in our example which uses `[wp_localize_script()](https://developer.wordpress.org/reference/functions/wp_localize_script/)`.

</p>
wp\_localize\_script(
	'ajax-script',
	'my\_ajax\_obj',
	array(
		'ajax\_url' => admin\_url( 'admin-ajax.php' ),
		'nonce'    => $title\_nonce,
	)
);
<p>

Note how our script handle `ajax-script` is used so that the global object is assigned to the right script. The object is global to our script, not to all scripts. Localization can also be called from the same hook that is used to enqueue scripts. The same goes for creating a nonce, though that particular function can be called virtually anywhere. All of that combined together in a single hook callback looks like this:

</p>
add\_action( 'admin\_enqueue\_scripts', 'my\_enqueue' );

/\*\*
 \* Enqueue my scripts and assets.
 \*
 \* @param $hook
 \*/
function my\_enqueue( $hook ) {
	if ( 'myplugin\_settings.php' !== $hook ) {
		return;
	}
	wp\_enqueue\_script(
		'ajax-script',
		plugins\_url( '/js/myjquery.js', \_\_FILE\_\_ ),
		array( 'jquery' ),
		'1.0.0',
		true
	);

	wp\_localize\_script(
		'ajax-script',
		'my\_ajax\_obj',
		array(
			'ajax\_url' => admin\_url( 'admin-ajax.php' ),
			'nonce'    => wp\_create\_nonce( 'title\_example' ),
		)
	);
}
<p>

[Expand full source code](#)[Collapse full source code](#)

Note: Remember to only add this nonce localization to the needed pages, do not display a nonce to someone who sould not use it. And remember to use `current_user_can()` with a capability or role to complete the security.

## AJAX Action

The other major part of the server side PHP code is the actual AJAX handler that receives the POSTed data, does something with it, then sends an appropriate response back to the browser. This takes on the form of a WordPress [action hook](https://developer.wordpress.org/plugins/hooks/actions/). Which hook tag you use depends on whether the user is logged in or not and what value your jQuery script passed as the *action:* value.

Note: **$\_GET , $\_POST and $\_COOKIE vs $\_REQUEST**

You’ve probably used one or more of the PHP super globals such as `$_GET` or `$_POST` to retrieve values from forms or cookies (using `$_COOKIE`). Maybe you prefer `$_REQUEST` instead, or at least have seen it used. It’s kind of cool – regardless of the request method, `POST` or `GET`, it will have the form values. Works great for pages that use both methods. On top of that, it has cookie values as well. One stop shopping! Therein lies its tragic flaw. In the case of a name conflict, the cookie value will override any form values. Thus it is ridiculously easy for a bad actor to craft a counterfeit cookie on their browser, which will overwrite any form value you might be expecting from the request. `$_REQUEST` is an easy route for hackers to inject arbitrary data into your form values. To be extra safe, stick to the specific variables and avoid the one size fits all.

Since our AJAX exchange is for the plugin’s settings page, the user must be logged in. If you recall from the [jQuery section](https://developer.wordpress.org/plugins/javascript/jquery/), the `action:` value is `"my_tag_count"`. This means our action hook tag will be `wp_ajax_my_tag_count`. If our AJAX exchange were to be utilized by users who were not currently logged in, the action hook tag would be `wp_ajax_nopriv_my_tag_count` The basic code used to hook the action looks like this:

</p>
add\_action( 'wp\_ajax\_my\_tag\_count', 'my\_ajax\_handler' );

/\*\*
 \* Handles my AJAX request.
 \*/
function my\_ajax\_handler() {
	// Handle the ajax request here

	wp\_die(); // All ajax handlers die when finished
}
<p>

The first thing your AJAX handler should do is verify the nonce sent by jQuery with `[check_ajax_referer()](https://developer.wordpress.org/reference/functions/check_ajax_referer/)`, which should be the same value that was localized when the script was enqueued.

</p>
check\_ajax\_referer( 'title\_example' );
<p>

The provided parameter must be identical to the parameter provided [earlier](#php-nonce) to `wp_create_nonce()`. The function simply dies if the nonce does not check out. If this were a true nonce, now that it was used, the value is no longer any good. You would then generate a new one and send it to the callback script so that it can be used for the next request. But since WordPress nonces are good for twenty-four hours, you needn’t do anything but check it.

### Data

With the nonce out of the way, our handler can deal with the data sent by the jQuery script contained in `$_POST['title']`. We can save the user’s selection in user meta by using [update\_user\_meta()](https://developer.wordpress.org/reference/functions/update_user_meta/).

</p>
update\_user\_meta( get\_current\_user\_id(), 'title\_preference', sanitize\_post\_title( $\_POST\['title'\] ) );
<p>

Then we build a query in order to get the post count for the selected title tag.

</p>
$args      = array(
	'tag' => $\_POST\['title'\],
);
$the\_query = new WP\_Query( $args );
<p>

Finally we can send the response back to the jQuery script. There’s several ways to transmit data. Let’s look at some of the options before we deal with the specifics of our example.

#### XML

PHP support for XML leaves something to be desired. Fortunately, WordPress provides the `[WP_Ajax_Response](https://developer.wordpress.org/reference/classes/wp_ajax_response/)` class to make the task easier. The [WP\_Ajax\_Response](https://developer.wordpress.org/reference/classes/wp_ajax_response/) class will generate an XML-formatted response, set the correct content type for the header, output the response xml, then die — ensuring a proper XML response.

#### JSON

This format is lightweight and easy to use, and WordPress provides the `[wp_send_json](https://developer.wordpress.org/reference/functions/wp_send_json/)` function to json-encode your response, print it, and die — effectively replacing [WP\_Ajax\_Response](https://developer.wordpress.org/reference/classes/wp_ajax_response/). WordPress also provides the `[wp_send_json_success](https://developer.wordpress.org/reference/functions/wp_send_json_success/)` and `[wp_send_json_error](https://developer.wordpress.org/reference/functions/wp_send_json_error/)` functions, which allow the appropriate done() or fail() callbacks to fire in JS.

#### Other

You can transfer data any way you like, as long as the sender and receiver are coordinated. Text formats like comma delimited or tab delimited are one of many possibilities. For small amounts of data, sending the raw stream may be adequate. That is what we will do with our example – we will send the actual replacement HTML, nothing else.

</p>
echo esc\_html( $\_POST\['title'\] ) . ' (' . $the\_query->post\_count . ') ';
<p>

In a real world application, you must account for the possibility that the action could fail for some reason–for instance, maybe the database server is down. The response should allow for this contingency, and the jQuery script receiving the response should act accordingly, perhaps telling the user to try again later.

### Die

When the handler has finished all of its tasks, it needs to die. If you are using the [WP\_Ajax\_Response](https://developer.wordpress.org/reference/classes/wp_ajax_response/) or wp\_send\_json\* functions, this is automatically handled for you. If not, simply use the WordPress `[wp_die()](https://developer.wordpress.org/reference/functions/wp_die/)` function.

### AJAX Handler Summary

The complete AJAX handler for our example looks like this:

</p>
/\*\*
 \* AJAX handler using JSON
 \*/
function my\_ajax\_handler\_\_json() {
	check\_ajax\_referer( 'title\_example' );
	update\_user\_meta( get\_current\_user\_id(), 'title\_preference', sanitize\_post\_title( $\_POST\['title'\] ) );
	$args      = array(
		'tag' => $\_POST\['title'\],
	);
	$the\_query = new WP\_Query( $args );
	wp\_send\_json( esc\_html( $\_POST\['title'\] ) . ' (' . $the\_query->post\_count . ') ' );
}
<p>

</p>
/\*\*
 \* AJAX handler not using JSON.
 \*/
function my\_ajax\_handler() {
	check\_ajax\_referer( 'title\_example' );
	update\_user\_meta( get\_current\_user\_id(), 'title\_preference', sanitize\_post\_title( $\_POST\['title'\] ) );
	$args      = array(
		'tag' => $\_POST\['title'\],
	);
	$the\_query = new WP\_Query( $args );
	echo esc\_html( $\_POST\['title'\] ) . ' (' . $the\_query->post\_count . ') ';
	wp\_die(); // All ajax handlers should die when finished
}
<p>

[Expand full source code](#)[Collapse full source code](#)