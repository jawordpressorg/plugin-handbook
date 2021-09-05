# Using Settings API

## Adding Settings

You must define a new setting using [register\_setting()](https://developer.wordpress.org/reference/functions/register_setting/), it will create an entry in the `{$wpdb->prefix}_options` table.

You can add new sections on existing pages using [add\_settings\_section()](https://developer.wordpress.org/reference/functions/add_settings_section/).

You can add new fields to existing sections using [add\_settings\_field()](https://developer.wordpress.org/reference/functions/add_settings_field/).

Alert:  
[register\_setting()](https://developer.wordpress.org/reference/functions/register_setting/) as well as the mentioned `add_settings_*()` functions should all be added to the `admin_init` action hook.

### Add a Setting

</p>
register\_setting(
    string $option\_group,
    string $option\_name,
    callable $sanitize\_callback = ''
);
<p>

Please refer to the Function Reference about [register\_setting()](https://developer.wordpress.org/reference/functions/register_setting/) for full explanation about the used parameters.

### Add a Section

</p>
add\_settings\_section(
    string $id,
    string $title,
    callable $callback,
    string $page
);
<p>

Sections are the groups of settings you see on WordPress settings pages with a shared heading. In your plugin you can add new sections to existing settings pages rather than creating a whole new page. This makes your plugin simpler to maintain and creates fewer new pages for users to learn.

Please refer to the Function Reference about [add\_settings\_section()](https://developer.wordpress.org/reference/functions/add_settings_section/) for full explanation about the used parameters.

### Add a Field

</p>
add\_settings\_field(
    string $id,
    string $title,
    callable $callback,
    string $page,
    string $section = 'default',
    array $args = \[\]
);
<p>

Please refer to the Function Reference about [add\_settings\_field()](https://developer.wordpress.org/reference/functions/add_settings_field/) for full explanation about the used parameters.

### Example

</p>
function wporg\_settings\_init() {
	// register a new setting for "reading" page
	register\_setting('reading', 'wporg\_setting\_name');

	// register a new section in the "reading" page
	add\_settings\_section(
		'wporg\_settings\_section',
		'WPOrg Settings Section', 'wporg\_settings\_section\_callback',
		'reading'
	);

	// register a new field in the "wporg\_settings\_section" section, inside the "reading" page
	add\_settings\_field(
		'wporg\_settings\_field',
		'WPOrg Setting', 'wporg\_settings\_field\_callback',
		'reading',
		'wporg\_settings\_section'
	);
}

/\*\*
 \* register wporg\_settings\_init to the admin\_init action hook
 \*/
add\_action('admin\_init', 'wporg\_settings\_init');

/\*\*
 \* callback functions
 \*/

// section content cb
function wporg\_settings\_section\_callback() {
	echo '<p>WPOrg Section Introduction.</p>';
}

// field content cb
function wporg\_settings\_field\_callback() {
	// get the value of the setting we've registered with register\_setting()
	$setting = get\_option('wporg\_setting\_name');
	// output the field
	?>
	<input type="text" name="wporg\_setting\_name" value="<?php echo isset( $setting ) ? esc\_attr( $setting ) : ''; ?>">
    <?php
}
<p>

[Expand full source code](#)[Collapse full source code](#)

## Getting Settings

</p>
get\_option(
    string $option,
    mixed $default = false
);
<p>

Getting settings is accomplished with the [get\_option()](https://developer.wordpress.org/reference/functions/get_option/) function.  
The function accepts two parameters: the name of the option and an optional default value for that option.

### Example

</p>
// Get the value of the setting we've registered with register\_setting()
$setting = get\_option('wporg\_setting\_name');
<p>