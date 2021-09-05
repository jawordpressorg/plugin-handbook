# Adding the Personal Data Eraser to Your Plugin

In WordPress 4.9.6, new tools were added to make compliance easier with laws like the European Union’s General Data Protection Regulation, or GDPR for short. Among the tools added is a Personal Data Removal tool which supports erasing/anonymizing personal data for a given user. It does NOT delete registered user accounts – that is still a separate step the admin can choose whether or not to do.

In addition to the personal data stored in things like WordPress comments, plugins can also hook into the eraser feature to erase the personal data they collect, whether it be in something like postmeta or even an entirely new Custom Post Type (CPT).

Like the exporters, the “key” for all the erasers is the user’s email address – this was chosen because it supports erasing personal data for both full-fledged registered users and also unregistered users (e.g. like a logged out commenter).

However, since performing a personal data erase is a destructive process, we don’t want to just do it without confirming the request, so the admin-facing user interface starts all requests by having the admin enter the username or email address making the request and then sends then a link to click to confirm their request. Once a request has been confirmed, the admin can kick off personal data erasure for the user, or force one if the need arises.

The way the personal data export is erased is similar to how the personal data exporters – and relies on hooking “eraser” callbacks to do the dirty work of erasing the data. When the admin clicks on the remove personal data link, an AJAX loop begins that iterates over all the erasers registered in the system, one at a time. In addition to erasers built into core, plugins can register their own eraser callbacks.

The eraser callback interface is designed to be as simple as possible. An eraser callback receives the email address we are working with, and a page parameter as well. The page parameter (which starts at 1) is used to avoid plugins potentially causing timeouts by attempting to erase all the personal data they’ve collected at once. A well behaved plugin will limit the amount of data it attempts to erase per page (e.g. 100 posts, 200 comments, etc.)

The eraser callback replies whether items containing personal data were erased, whether any items containing personal data were retained, an array of messages to present to the admin (explaining why items that were retained were retained) and whether it is done or not. If an eraser callback reports that it is not done, it will be called again (in a separate request) with the page parameter incremented by 1.

When all the exporters have been called to completion, the admin user interface is updated to show whether or not all personal data found was erased, and any messages explaining why personal data was retained.

Let’s work on a hypothetical plugin which adds commenter location data to comments. Let’s assume the plugin has used `add_comment_meta` to add location data using `meta_ke`ys of `latitude` and `longitude`

The first thing the plugin needs to do is to create an eraser function that accepts an email address and a page, e.g.:

</p>
/\*\*
 \* Removes any stored location data from a user's comment meta for the supplied email address.
 \*
 \* @param string $email\_address   email address to manipulate
 \* @param int    $page            pagination
 \*
 \* @return array
 \*/
function wporg\_remove\_location\_meta\_from\_comments\_for\_email( $email\_address, $page = 1 ) {
	$number = 500; // Limit us to avoid timing out
	$page   = (int) $page;

	$comments = get\_comments(
		array(
			'author\_email' => $email\_address,
			'number'       => $number,
			'paged'        => $page,
			'order\_by'     => 'comment\_ID',
			'order'        => 'ASC',
		)
	);

	$items\_removed = false;

	foreach ( (array) $comments as $comment ) {
		$latitude  = get\_comment\_meta( $comment->comment\_ID, 'latitude', true );
		$longitude = get\_comment\_meta( $comment->comment\_ID, 'longitude', true );

		if ( ! empty( $latitude ) ) {
			delete\_comment\_meta( $comment->comment\_ID, 'latitude' );
			$items\_removed = true;
		}

		if ( ! empty( $longitude ) ) {
			delete\_comment\_meta( $comment->comment\_ID, 'longitude' );
			$items\_removed = true;
		}
	}

	// Tell core if we have more comments to work on still
	$done = count( $comments ) < $number;
	return array(
		'items\_removed'  => $items\_removed,
		'items\_retained' => false, // always false in this example
		'messages'       => array(), // no messages in this example
		'done'           => $done,
	);
}
<p>

[Expand full source code](#)[Collapse full source code](#)

The next thing the plugin needs to do is to register the callback by filtering the eraser array using the \`wp\_privacy\_personal\_data\_erasers\`  
filter.

When registering you provide a friendly name for the eraser (to aid in debugging – this friendly name is not shown to anyone at this time)  
and the callback, e.g.

</p>
/\*\*
 \* Registers all data erasers.
 \*
 \* @param array $exporters
 \*
 \* @return mixed
 \*/
function wporg\_register\_privacy\_erasers( $erasers ) {
	$erasers\['my-plugin-slug'\] = array(
		'eraser\_friendly\_name' => \_\_( 'Comment Location Plugin', 'text-domain' ),
		'callback'             => 'wporg\_remove\_location\_meta\_from\_comments\_for\_email',
	);
	return $erasers;
}

add\_filter( 'wp\_privacy\_personal\_data\_erasers', 'wporg\_register\_privacy\_erasers' );
<p>

[Expand full source code](#)[Collapse full source code](#)

And that’s all there is to it! Your plugin will now clean up its personal data!