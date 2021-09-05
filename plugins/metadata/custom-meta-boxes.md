# Custom Meta Boxes

## What is a Meta Box?

When a user edits a post, the edit screen is composed of several default boxes: Editor, Publish, Categories, Tags, etc. These boxes are meta boxes. Plugins can add custom meta boxes to an edit screen of any post type.

The content of custom meta boxes are usually HTML form elements where the user enters data related to a Plugin’s purpose, but the content can be practically any HTML you desire.

## Why Use Meta Boxes?

Meta boxes are handy, flexible, modular edit screen elements that can be used to collect information related to the post being edited. Your custom meta box will be on the same screen as all the other post related information; so a clear relationship is established.

Meta boxes are easily hidden from users that do not need to see them, and displayed to those that do. Meta boxes can be user-arranged on the edit screen. The users are free to arrange the edit screen in a way that suits them, giving users control over their editing environment.

Alert:  
All examples on this page are for illustration purposes only. The code is not suitable for production environments.

Operations such as [securing input](../../plugin-security/securing-input/), [user capabilities](https://developer.wordpress.org/plugins/security/checking-user-capabilities/), [nonces](https://developer.wordpress.org/plugins/security/nonces/), and [internationalization](../internationalization/) have been intentionally omitted. Be sure to always address these important operations.

## Adding Meta Boxes

To create a meta box use the [add\_meta\_box()](https://developer.wordpress.org/reference/functions/add_meta_box/) function and plug its execution to the `[add_meta_boxes](https://developer.wordpress.org/reference/hooks/add_meta_boxes/)` action hook.

The following example is adding a meta box to the `post` edit screen and the `wporg_cpt` edit screen.

function wporg\_add\_custom\_box() {
	$screens = \[ 'post', 'wporg\_cpt' \];
	foreach ( $screens as $screen ) {
		add\_meta\_box(
			'wporg\_box\_id',                 // Unique ID
			'Custom Meta Box Title',      // Box title
			'wporg\_custom\_box\_html',  // Content callback, must be of type callable
			$screen                            // Post type
		);
	}
}
add\_action( 'add\_meta\_boxes', 'wporg\_add\_custom\_box' );

The `wporg_custom_box_html` function will hold the HTML for the meta box.

The following example is adding form elements, labels, and other HTML elements.

function wporg\_custom\_box\_html( $post ) {
	?>
	<label for="wporg\_field">Description for this field</label>
	<select name="wporg\_field" id="wporg\_field" class="postbox">
		<option value="">Select something...</option>
		<option value="something">Something</option>
		<option value="else">Else</option>
	</select>
	<?php
}

Note:  
**Note there are no submit buttons in meta boxes.** The meta box HTML is included inside the edit screen’s form tags, all the post data including meta box values are transfered via `POST` when the user clicks on the Publish or Update buttons.

The example shown here only contains one form field, a drop down list. You may create as many as needed in any particular meta box. If you have a lot of fields to display, consider using multiple meta boxes, grouping similar fields together in each meta box. This helps keep the page more organized and visually appealing.

### Getting Values

To retrieve saved user data and make use of it, you need to get it from wherever you saved it initially. If it was stored in the `postmeta` table, you may get the data with [get\_post\_meta()](https://developer.wordpress.org/reference/functions/get_post_meta/).

The following example enhances the previous form elements with pre-populated data based on saved meta box values. You will learn how to save meta box values in the next section.

function wporg\_custom\_box\_html( $post ) {
	$value = get\_post\_meta( $post->ID, '\_wporg\_meta\_key', true );
	?>
	<label for="wporg\_field">Description for this field</label>
	<select name="wporg\_field" id="wporg\_field" class="postbox">
		<option value="">Select something...</option>
		<option value="something" <?php selected( $value, 'something' ); ?>>Something</option>
		<option value="else" <?php selected( $value, 'else' ); ?>>Else</option>
	</select>
	<?php
}

More on the [selected()](https://developer.wordpress.org/reference/functions/selected/) function.

### Saving Values

When a post type is saved or updated, several actions fire, any of which might be appropriate to hook into in order to save the entered values. In this example we use the `save_post` action hook but other hooks may be more appropriate for certain situations. Be aware that `save_post` may fire more than once for a single update event. Structure your approach to saving data accordingly.

You may save the entered data anywhere you want, even outside WordPress. Since you are probably dealing with data related to the post, the `postmeta` table is often a good place to store data.

The following example will save the `wporg_field` field value in the `_wporg_meta_key` meta key, which is hidden.

function wporg\_save\_postdata( $post\_id ) {
	if ( array\_key\_exists( 'wporg\_field', $\_POST ) ) {
		update\_post\_meta(
			$post\_id,
			'\_wporg\_meta\_key',
			$\_POST\['wporg\_field'\]
		);
	}
}
add\_action( 'save\_post', 'wporg\_save\_postdata' );

In production code, remember to follow the security measures outlined in the info box!

## Behind the Scenes

You don’t normally need to be concerned about what happens behind the scenes. This section was added for completeness.

When a post edit screen wants to display all the meta boxes that were added to it, it calls the [do\_meta\_boxes()](https://developer.wordpress.org/reference/functions/do_meta_boxes/) function. This function loops through all meta boxes and invokes the `callback` associated with each.  
In between each call, intervening markup (such as divs, titles, etc.) is added.

## Removing Meta Boxes

To remove an existing meta box from an edit screen use the [remove\_meta\_box()](https://developer.wordpress.org/reference/functions/remove_meta_box/) function. The passed parameters must exactly match those used to add the meta box with [add\_meta\_box()](https://developer.wordpress.org/reference/functions/add_meta_box/).

To remove default meta boxes check the source code for the parameters used. The default [add\_meta\_box()](https://developer.wordpress.org/reference/functions/add_meta_box/) calls are made from `wp-includes/edit-form-advanced.php`.

## Implementation Variants

So far we’ve been using the procedural technique of implementing meta boxes. Many plugin developers find the need to implement meta boxes using various other techniques.

### OOP

Adding meta boxes using OOP is easy and saves you from having to worry about naming collisions in the global namespace.  
To save memory and allow easier implementation, the following example uses an abstract Class with static methods.

abstract class WPOrg\_Meta\_Box {


	/\*\*
	 \* Set up and add the meta box.
	 \*/
	public static function add() {
		$screens = \[ 'post', 'wporg\_cpt' \];
		foreach ( $screens as $screen ) {
			add\_meta\_box(
				'wporg\_box\_id',          // Unique ID
				'Custom Meta Box Title', // Box title
				\[ self::class, 'html' \],   // Content callback, must be of type callable
				$screen                  // Post type
			);
		}
	}


	/\*\*
	 \* Save the meta box selections.
	 \*
	 \* @param int $post\_id  The post ID.
	 \*/
	public static function save( int $post\_id ) {
		if ( array\_key\_exists( 'wporg\_field', $\_POST ) ) {
			update\_post\_meta(
				$post\_id,
				'\_wporg\_meta\_key',
				$\_POST\['wporg\_field'\]
			);
		}
	}


	/\*\*
	 \* Display the meta box HTML to the user.
	 \*
	 \* @param \\WP\_Post $post   Post object.
	 \*/
	public static function html( $post ) {
		$value = get\_post\_meta( $post->ID, '\_wporg\_meta\_key', true );
		?>
		<label for="wporg\_field">Description for this field</label>
		<select name="wporg\_field" id="wporg\_field" class="postbox">
			<option value="">Select something...</option>
			<option value="something" <?php selected( $value, 'something' ); ?>>Something</option>
			<option value="else" <?php selected( $value, 'else' ); ?>>Else</option>
		</select>
		<?php
	}
}

add\_action( 'add\_meta\_boxes', \[ 'WPOrg\_Meta\_Box', 'add' \] );
add\_action( 'save\_post', \[ 'WPOrg\_Meta\_Box', 'save' \] );

[Expand full source code](#)[Collapse full source code](#)

### AJAX

Since the HTML elements of the meta box are inside the `form` tags of the edit screen, the default behavior is to parse meta box values from the `$_POST` super global *after the user have submitted the page*.

You can enhance the default experience with AJAX; this allows to perform actions based on user input and behavior; regardless if they’ve submitted the page.

#### Define a Trigger

First, you must define the trigger, this can be a link click, a change of a value or any other JavaScript event.

In the example below we will define `change` as our trigger for performing an AJAX request.

/\*jslint browser: true, plusplus: true \*/
(function ($, window, document) {
    'use strict';
    // execute when the DOM is ready
    $(document).ready(function () {
        // js 'change' event triggered on the wporg\_field form field
        $('#wporg\_field').on('change', function () {
            // our code
        });
    });
}(jQuery, window, document));

#### Client Side Code

Next, we need to define what we want the trigger to do, in other words we need to write our client side code.

In the example below we will make a `POST` request, the response will either be success or failure, this will indicate wither the value of the `wporg_field` is valid.

/\*jslint browser: true, plusplus: true \*/
(function ($, window, document) {
    'use strict';
    // execute when the DOM is ready
    $(document).ready(function () {
        // js 'change' event triggered on the wporg\_field form field
        $('#wporg\_field').on('change', function () {
            // jQuery post method, a shorthand for $.ajax with POST
            $.post(wporg\_meta\_box\_obj.url,                        // or ajaxurl
                   {
                       action: 'wporg\_ajax\_change',               // POST data, action
                       wporg\_field\_value: $('#wporg\_field').val() // POST data, wporg\_field\_value
                   }, function (data) {
                        // handle response data
                        if (data === 'success') {
                            // perform our success logic
                        } else if (data === 'failure') {
                            // perform our failure logic
                        } else {
                            // do nothing
                        }
                    }
            );
        });
    });
}(jQuery, window, document));

[Expand full source code](#)[Collapse full source code](#)

We took the WordPress AJAX file URL dynamically from the `wporg_meta_box_obj` JavaScript custom object that we will create in the next step.

Note:  
If your meta box only requires the WordPress AJAX file URL; instead of creating a new custom JavaScript object you could use the `ajaxurl` predefined JavaScript variable.  
**Available only in the WordPress Administration.** Make sure it is not empty before performing any logic.

#### Enqueue Client Side Code

Next step is to put our code into a script file and enqueue it on our edit screens.

In the example below we will add the AJAX functionality to the edit screens of the following post types: post, wporg\_cpt.

The script file will reside at `/plugin-name/admin/meta-boxes/js/admin.js`,  
`plugin-name` being the main plugin folder,  
`/plugin-name/plugin.php` the file calling the function.

function wporg\_meta\_box\_scripts()
{
    // get current admin screen, or null
    $screen = get\_current\_screen();
    // verify admin screen object
    if (is\_object($screen)) {
        // enqueue only for specific post types
        if (in\_array($screen->post\_type, \['post', 'wporg\_cpt'\])) {
            // enqueue script
            wp\_enqueue\_script('wporg\_meta\_box\_script', plugin\_dir\_url(\_\_FILE\_\_) . 'admin/meta-boxes/js/admin.js', \['jquery'\]);
            // localize script, create a custom js object
            wp\_localize\_script(
                'wporg\_meta\_box\_script',
                'wporg\_meta\_box\_obj',
                \[
                    'url' => admin\_url('admin-ajax.php'),
                \]
            );
        }
    }
}
add\_action('admin\_enqueue\_scripts', 'wporg\_meta\_box\_scripts');

[Expand full source code](#)[Collapse full source code](#)

#### Server Side Code

The last step is to write our server side code that is going to handle the request.

function wporg\_meta\_box\_scripts() {
	 // Get current admin screen, or null.
	$screen = get\_current\_screen();
	// Verify admin screen object before use.
	if ( is\_object( $screen ) ) {
		// Enqueue only for specific post types.
		if ( in\_array( $screen->post\_type, \[ 'post', 'wporg\_cpt' \], true ) ) {
			wp\_enqueue\_script( 'wporg\_meta\_box\_script', plugin\_dir\_url( \_\_FILE\_\_ ) . 'admin/meta-boxes/js/admin.js', \[ 'jquery' \], '1.0.0', true );
			wp\_localize\_script(
				'wporg\_meta\_box\_script',
				'wporg\_meta\_box\_obj',
				\[
					'url' => admin\_url( 'admin-ajax.php' ),
				\]
			);
		}
	}
}
add\_action( 'admin\_enqueue\_scripts', 'wporg\_meta\_box\_scripts' );

[Expand full source code](#)[Collapse full source code](#)

As a final reminder, the code illustrated on this page lacks important operations that take care of security. Be sure your production code includes such operations.

See [Handbook’s AJAX Chapter](https://developer.wordpress.org/plugins/javascript/ajax/) and the [Codex](https://codex.wordpress.org/AJAX_in_Plugins) for more on AJAX.

## More Information

*   [Complex Meta Boxes in WordPress](http://www.wproots.com/complex-meta-boxes-in-wordpress/)
*   [How To Create Custom Post Meta Boxes In WordPress](http://wp.smashingmagazine.com/2011/10/04/create-custom-post-meta-boxes-wordpress/)
*   [WordPress Meta Boxes: a Comprehensive Developer’s Guide](http://themefoundation.com/wordpress-meta-boxes-guide/)