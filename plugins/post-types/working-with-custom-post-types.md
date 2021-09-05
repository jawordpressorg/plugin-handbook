# Working with Custom Post Types

## Custom Post Type Templates

You can create custom [templates](https://make.wordpress.org/docs/theme-developer-handbook/theme-basics/theme-files/) for your custom post types. In the same way posts and their archives can be displayed using `single.php` and `archive.php`, you can create the templates:

*   `single-{post_type}.php` – for single posts of a custom post type
*   `archive-{post_type}.php` – for the archive

Where `{post_type}` is the post type identifier, used as the `$post_type` argument of the `register_post_type()` function.

Building upon what we’ve learned previously, you could create `single-wporg_product.php` and `archive-wporg_product.php` template files for single product posts and the archive.

Alternatively, you can use the [is\_post\_type\_archive()](https://developer.wordpress.org/reference/functions/is_post_type_archive/) function in any template file to check if the query shows an archive page of a given post type, and the [post\_type\_archive\_title()](https://developer.wordpress.org/reference/functions/post_type_archive_title/) function to display the post type title.

## Querying by Post Type

You can query posts of a specific type by passing the `post_type` key in the arguments array of the `WP_Query` class constructor.

<?php
$args = array(
	'post\_type'      => 'product',
	'posts\_per\_page' => 10,
);
$loop = new WP\_Query($args);
while ( $loop->have\_posts() ) {
	$loop->the\_post();
	?>
	<div class="entry-content">
		<?php the\_title(); ?>
		<?php the\_content(); ?>
	</div>
	<?php
}

[Expand full source code](#)[Collapse full source code](#)

This loops through the latest ten product posts and displays the title and content of them one by one.

## Altering the Main Query

Registering a custom post type does not mean it gets added to the main query automatically.

If you want your custom post type posts to show up on standard archives or include them on your home page mixed up with other post types, use the `[pre_get_posts](https://developer.wordpress.org/reference/hooks/pre_get_posts/)` action hook.

The next example will show posts from `post`, `page` and `movie` post types on the home page:

function wporg\_add\_custom\_post\_types($query) {
	if ( is\_home() && $query->is\_main\_query() ) {
		$query->set( 'post\_type', array( 'post', 'page', 'movie' ) );
	}
	return $query;
}
add\_action('pre\_get\_posts', 'wporg\_add\_custom\_post\_types');