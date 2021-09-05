# Custom Hooks

**An important, but often overlooked practice is using custom hooks in your plugin so that other developers can extend and modify it.**

Custom hooks are created and called in the same way that WordPress Core hooks are.

## Create a Hook

To create a custom hook, use [do\_action()](https://developer.wordpress.org/reference/functions/do_action/) for [Actions](https://developer.wordpress.org/plugins/hooks/actions/) and [apply\_filters()](https://developer.wordpress.org/reference/functions/apply_filters/) for [Filters](https://developer.wordpress.org/plugins/hooks/filters/).

Note:  
We recommend using [apply\_filters()](https://developer.wordpress.org/reference/functions/apply_filters/) on any text that is output to the browser. Particularly on the frontend.

This makes it easier for plugins to be modified according to the user’s needs.

## Add a Callback to the Hook

To add a callback function to a custom hook, use [add\_action()](https://developer.wordpress.org/reference/functions/add_action/) for [Actions](https://developer.wordpress.org/plugins/hooks/actions/) and [add\_filter()](https://developer.wordpress.org/reference/functions/add_filter/) for [Filters](https://developer.wordpress.org/plugins/hooks/filters/).

## Naming Conflicts

Since any plugin can create a custom hook, it’s important to prefix your hook names to avoid collisions with other plugins.

For example, a filter named `email_body` would be less useful because it’s likely that another developer will choose that same name. If the user installs both plugins, it could lead to bugs that are difficult to track down.

Naming the function `wporg_email_body` (where `wporg_` is a unique prefix for your plugin) would avoid any collisions.

## Examples

### Extensible Action: Settings Form

If your plugin adds a settings form to the Administrative Panels, you can use Actions to allow other plugins to add their own settings to it.

</p>
<p>    Foo:<br />
    Bar:<br />
   &lt;?php<br />
    do\_action( &#039;wporg\_after\_settings\_page\_html&#039; );<br />
}<br />

Now another plugin can register a callback function for the `wporg_after_settings_page_html` hook and inject new settings:

</p>
<p>    New 1:<br />
    &lt;?php<br />
}<br />
add\_action( &#039;wporg\_after\_settings\_page\_html&#039;, &#039;myprefix\_add\_settings&#039; );<br />

### Extensible Filter: Custom Post Type

In this example, when the new post type is registered, the parameters that define it are passed through a filter, so another plugin can change them before the post type is created.

<br />
&lt;?php<br />
function wporg\_create\_post\_type()<br />
{<br />
    $post\_type\_params = \[/\* ... \*/\];</p>
<p>    register\_post\_type(<br />
        &#039;post\_type\_slug&#039;,<br />
        apply\_filters( &#039;wporg\_post\_type\_params&#039;, $post\_type\_params )<br />
    );<br />
}<br />

Now another plugin can register a callback function for the `wporg_post_type_params` hook and change post type parameters:

<br />
&lt;?php<br />
function myprefix\_change\_post\_type\_params( $post\_type\_params ) {<br />
	$post\_type\_params\[&#039;hierarchical&#039;\] = true;<br />
	return $post\_type\_params;<br />
}<br />
add\_filter( &#039;wporg\_post\_type\_params&#039;, &#039;myprefix\_change\_post\_type\_params&#039; );<br />

## External Resources

*   [Extendable Extensions](http://wordpress.tv/2012/08/27/michael-fields-extendable-extensions/) by Michael Fields
*   [The Pluggable Plugin](http://wordpress.tv/2010/12/03/brandon-dove-the-pluggable-plugin/) by Brandon Dove
*   [WordPress Plugin Pet Peeves #3: Not Being Extensible](http://willnorris.com/2009/06/wordpress-plugin-pet-peeve-3-not-being-extensible) by Will Norris