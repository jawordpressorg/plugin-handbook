# Custom Settings Page

Creating a custom settings page includes the combination of: [creating an administration menu](https://developer.wordpress.org/plugins/administration-menus/), [using Settings API](https://developer.wordpress.org/plugins/settings/using-settings-api/) and [Options API](https://developer.wordpress.org/plugins/settings/options-api/).

Alert:  
Please read these chapters before attempting to create your own settings page.

The example below can be used for quick reference on these topics by following the comments.

## Complete Example

Complete example which adds a Top-Level Menu named `WPOrg`, registers a custom option named `wporg_options` and performs the CRUD (create, read, update, delete) logic using Settings API and Options API (including showing error/update messages).

<?php
/\*\*
 \* @internal never define functions inside callbacks.
 \* these functions could be run multiple times; this would result in a fatal error.
 \*/

/\*\*
 \* custom option and settings
 \*/
function wporg\_settings\_init() {
	// Register a new setting for "wporg" page.
	register\_setting( 'wporg', 'wporg\_options' );

	// Register a new section in the "wporg" page.
	add\_settings\_section(
		'wporg\_section\_developers',
		\_\_( 'The Matrix has you.', 'wporg' ), 'wporg\_section\_developers\_callback',
		'wporg'
	);

	// Register a new field in the "wporg\_section\_developers" section, inside the "wporg" page.
	add\_settings\_field(
		'wporg\_field\_pill', // As of WP 4.6 this value is used only internally.
		                        // Use $args' label\_for to populate the id inside the callback.
			\_\_( 'Pill', 'wporg' ),
		'wporg\_field\_pill\_cb',
		'wporg',
		'wporg\_section\_developers',
		array(
			'label\_for'         => 'wporg\_field\_pill',
			'class'             => 'wporg\_row',
			'wporg\_custom\_data' => 'custom',
		)
	);
}

/\*\*
 \* Register our wporg\_settings\_init to the admin\_init action hook.
 \*/
add\_action( 'admin\_init', 'wporg\_settings\_init' );


/\*\*
 \* Custom option and settings:
 \*  - callback functions
 \*/


/\*\*
 \* Developers section callback function.
 \*
 \* @param array $args  The settings array, defining title, id, callback.
 \*/
function wporg\_section\_developers\_callback( $args ) {
	?>
	<p id="<?php echo esc\_attr( $args\['id'\] ); ?>"><?php esc\_html\_e( 'Follow the white rabbit.', 'wporg' ); ?></p>
	<?php
}

/\*\*
 \* Pill field callbakc function.
 \*
 \* WordPress has magic interaction with the following keys: label\_for, class.
 \* - the "label\_for" key value is used for the "for" attribute of the <label>.
 \* - the "class" key value is used for the "class" attribute of the <tr> containing the field.
 \* Note: you can add custom key value pairs to be used inside your callbacks.
 \*
 \* @param array $args
 \*/
function wporg\_field\_pill\_cb( $args ) {
	// Get the value of the setting we've registered with register\_setting()
	$options = get\_option( 'wporg\_options' );
	?>
	<select
			id="<?php echo esc\_attr( $args\['label\_for'\] ); ?>"
			data-custom="<?php echo esc\_attr( $args\['wporg\_custom\_data'\] ); ?>"
			name="wporg\_options\[<?php echo esc\_attr( $args\['label\_for'\] ); ?>\]">
		<option value="red" <?php echo isset( $options\[ $args\['label\_for'\] \] ) ? ( selected( $options\[ $args\['label\_for'\] \], 'red', false ) ) : ( '' ); ?>>
			<?php esc\_html\_e( 'red pill', 'wporg' ); ?>
		</option>
 		<option value="blue" <?php echo isset( $options\[ $args\['label\_for'\] \] ) ? ( selected( $options\[ $args\['label\_for'\] \], 'blue', false ) ) : ( '' ); ?>>
			<?php esc\_html\_e( 'blue pill', 'wporg' ); ?>
		</option>
	</select>
	<p class="description">
		<?php esc\_html\_e( 'You take the blue pill and the story ends. You wake in your bed and you believe whatever you want to believe.', 'wporg' ); ?>
	</p>
	<p class="description">
		<?php esc\_html\_e( 'You take the red pill and you stay in Wonderland and I show you how deep the rabbit-hole goes.', 'wporg' ); ?>
	</p>
	<?php
}

/\*\*
 \* Add the top level menu page.
 \*/
function wporg\_options\_page() {
	add\_menu\_page(
		'WPOrg',
		'WPOrg Options',
		'manage\_options',
		'wporg',
		'wporg\_options\_page\_html'
	);
}


/\*\*
 \* Register our wporg\_options\_page to the admin\_menu action hook.
 \*/
add\_action( 'admin\_menu', 'wporg\_options\_page' );


/\*\*
 \* Top level menu callback function
 \*/
function wporg\_options\_page\_html() {
	// check user capabilities
	if ( ! current\_user\_can( 'manage\_options' ) ) {
		return;
	}

	// add error/update messages

	// check if the user have submitted the settings
	// WordPress will add the "settings-updated" $\_GET parameter to the url
	if ( isset( $\_GET\['settings-updated'\] ) ) {
		// add settings saved message with the class of "updated"
		add\_settings\_error( 'wporg\_messages', 'wporg\_message', \_\_( 'Settings Saved', 'wporg' ), 'updated' );
	}

	// show error/update messages
	settings\_errors( 'wporg\_messages' );
	?>
	<div class="wrap">
		<h1><?php echo esc\_html( get\_admin\_page\_title() ); ?></h1>
		<form action="options.php" method="post">
			<?php
			// output security fields for the registered setting "wporg"
			settings\_fields( 'wporg' );
			// output setting sections and their fields
			// (sections are registered for "wporg", each field is registered to a specific section)
			do\_settings\_sections( 'wporg' );
			// output save settings button
			submit\_button( 'Save Settings' );
			?>
		</form>
	</div>
	<?php
}

[Expand full source code](#)[Collapse full source code](#)