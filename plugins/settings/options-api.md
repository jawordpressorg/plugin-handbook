# Options API

The Options API, added in WordPress 1.0, allows creating, reading, updating and deleting of WordPress options. In combination with the [Settings API](https://developer.wordpress.org/plugins/settings/settings-api/) it allows controlling of options defined in settings pages.

## Where Options are Stored?

Options are stored in the `{$wpdb->prefix}_options` table. `$wpdb->prefix` is defined by the `$table_prefix` variable set in the `wp-config.php` file.

## How Options are Stored?

Options may be stored in the WordPress database in one of two ways: as a single value or as an array of values.

### Single Value

When saved as a single value, the option name refers to a single value.

```php
// add a new option
add_option('wporg_custom_option', 'hello world!');
// get an option
$option = get_option('wporg_custom_option');
```

### Array of Values

When saved as an array of values, the option name refers to an array, which itself may be comprised key/value pairs.

```php
// array of options
$data_r = array('title' => 'hello world!', 1, false );
// add a new option
add_option('wporg_custom_option', $data_r);
// get an option
$options_r = get_option('wporg_custom_option');
// output the title
echo esc_html($options_r['title']);
```

If you are working with a large number of related options, storing them as an array can have a positive impact on overall performance.

Note:  
Accessing data as individual options may result in many individual database transactions, and as a rule, database transactions are expensive operations (in terms of time and server resources). When you store or retrieve an array of options, it happens in a single transaction, which is ideal.

## Function Reference

| Add Option | Get Option | Update Option | Delete Option |
| --- | --- | --- | --- |
| [add\_option()](https://developer.wordpress.org/reference/functions/add_option/) | [get\_option()](https://developer.wordpress.org/reference/functions/get_option/) | [update\_option()](https://developer.wordpress.org/reference/functions/update_option/) | [delete\_option()](https://developer.wordpress.org/reference/functions/delete_option/) |
| [add\_site\_option()](https://developer.wordpress.org/reference/functions/add_site_option/) | [get\_site\_option()](https://developer.wordpress.org/reference/functions/get_site_option/) | [update\_site\_option()](https://developer.wordpress.org/reference/functions/update_site_option/) | [delete\_site\_option()](https://developer.wordpress.org/reference/functions/delete_site_option/) |