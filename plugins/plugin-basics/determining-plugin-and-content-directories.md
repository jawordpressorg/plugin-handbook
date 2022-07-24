# Determining Plugin and Content Directories

When coding WordPress plugins you often need to reference various files and folders throughout the WordPress installation and within your plugin or theme.

WordPress provides several functions for easily determining where a given file or directory lives. Always use these functions in your plugins instead of hard-coding references to the wp-content directory or using the WordPress internal constants.

Tip: WordPress allows users to place their wp-content directory anywhere they want and rename it whatever they want. Never assume that plugins will be in wp-content/plugins, uploads will be in wp-content/uploads, or that themes will be in wp-content/themes.

PHP’s `__FILE__` magic-constant resolves symlinks automatically, so if the `wp-content` or `wp-content/plugins` or even the individual plugin directory is symlinked, hardcoded paths will not work correctly.

## Common Usage

If your plugin includes JavaScript files, CSS files or other external files, then it’s likely you’ll need the URL to these files so you can load them into the page. To do this you should use the [plugins\_url()](https://developer.wordpress.org/reference/functions/plugins_url/) function like so:

```php
plugins_url( 'myscript.js', __FILE__ );
```

This will return the full URL to myscript.js, such as `example.com/wp-content/plugins/myplugin/myscript.js`.

To load your plugins’ JavaScript or CSS into the page you should use [`wp_enqueue_script()`](https://developer.wordpress.org/reference/functions/wp_enqueue_script/) or [`wp_enqueue_style()`](https://developer.wordpress.org/reference/functions/wp_enqueue_style/) respectively, passing the result of `plugins_url()` as the file URL.

## Available Functions

WordPress includes many other functions for determining paths and URLs to files or directories within plugins, themes, and WordPress itself. See the individual DevHub pages for each function for complete information on their use.

### Plugins

plugins\_url()  
plugin\_dir\_url()  
plugin\_dir\_path()  
plugin\_basename()

### Themes

get\_template\_directory\_uri()  
get\_stylesheet\_directory\_uri()  
get\_stylesheet\_uri()  
get\_theme\_root\_uri()  
get\_theme\_root()  
get\_theme\_roots()  
get\_stylesheet\_directory()  
get\_template\_directory()

### Site Home

home\_url()  
get\_home\_path()

### WordPress

admin\_url()  
site\_url()  
content\_url()  
includes\_url()  
wp\_upload\_dir()

### Multisite

get\_admin\_url()  
get\_home\_url()  
get\_site\_url()  
network\_admin\_url()  
network\_site\_url()  
network\_home\_url()

## Constants

WordPress makes use of the following constants when determining the path to the content and plugin directories. These should not be used directly by plugins or themes, but are listed here for completeness.

WP\_CONTENT\_DIR  // no trailing slash, full paths only  
WP\_CONTENT\_URL  // full url   
WP\_PLUGIN\_DIR  // full path, no trailing slash  
WP\_PLUGIN\_URL  // full url, no trailing slash

// Available per default in MS, not set in single site install  
// Can be used in single site installs (as usual: at your own risk)  
UPLOADS // (If set, uploads folder, relative to ABSPATH) (for e.g.: /wp-content/uploads)

## Related

****WordPress Directories****:

<table><tbody><tr><td><a href="https://developer.wordpress.org/reference/functions/home_url/">home_url()</a></td><td>Home URL</td><td><a href="http://www.example.com">http://www.example.com</a></td></tr><tr><td><a href="https://developer.wordpress.org/reference/functions/site_url/">site_url()</a></td><td>Site directory URL</td><td><a href="http://www.example.com">http://www.example.com</a> or <a href="http://www.example.com/wordpress">http://www.example.com/wordpress</a></td></tr><tr><td><a href="https://developer.wordpress.org/reference/functions/admin_url/">admin_url()</a></td><td>Admin directory URL</td><td><a href="http://www.example.com/wp-admin">http://www.example.com/wp-admin</a></td></tr><tr><td><a href="https://developer.wordpress.org/reference/functions/includes_url/">includes_url()</a></td><td>Includes directory URL</td><td><a href="http://www.example.com/wp-includes">http://www.example.com/wp-includes</a></td></tr><tr><td><a href="https://developer.wordpress.org/reference/functions/content_url/">content_url()</a></td><td>Content directory URL</td><td><a href="http://www.example.com/wp-content">http://www.example.com/wp-content</a></td></tr><tr><td><a href="https://developer.wordpress.org/reference/functions/plugins_url/">plugins_url()</a></td><td>Plugins directory URL</td><td><a href="http://www.example.com/wp-content/plugins">http://www.example.com/wp-content/plugins</a></td></tr><tr><td><a href="https://developer.wordpress.org/reference/functions/wp_upload_dir/">wp_upload_dir()</a></td><td>Upload directory URL (returns an array)</td><td><a href="http://www.example.com/wp-content/uploads">http://www.example.com/wp-content/uploads</a></td></tr></tbody></table>