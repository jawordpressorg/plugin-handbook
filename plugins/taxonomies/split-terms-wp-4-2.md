# Term Splitting (WP 4.2)

## Prior to WP 4.2

Prior to WP 4.2, Terms in different Taxonomies with the same slug shared a single Term ID.

For instance, a Tag and a Category with the slug “news” had the same Term ID.

## WP 4.2+

Beginning with WP 4.2, when one of these shared Terms is updated, it will be split: the updated term will be assigned a new Term ID.

## What does it mean for you?

In the vast majority of situations, this update will be seamless and uneventful. However, some plugins and themes who store Term IDs in options, post meta, user meta, or elsewhere might be affected.

## Handling the Split

WP 4.2 includes two different tools to help authors of plugins and themes with the transition.

### The `split_shared_term` hook

When a shared term is assigned a new Term ID, a new `split_shared_term` action is fired.

Here are a few examples of how plugin and theme authors can leverage this hook to ensure that stored Term IDs are updated.

#### Term ID stored in an option

Let’s say your plugin stores an option called `featured_tags` that contains an array of Term IDs (`[4, 6, 10]`) that serve as the query parameter for your homepage featured posts section.

In this example, you’ll hook to `split_shared_term` action, check whether the updated Term ID is in the array, and update if necessary.

<br />
/\*\*<br />
\* Update featured\_tags option when a shared term gets split.<br />
\*<br />
\* @param int $term\_id ID of the formerly shared term.<br />
\* @param int $new\_term\_id ID of the new term created for the $term\_taxonomy\_id.<br />
\* @param int $term\_taxonomy\_id ID for the term\_taxonomy row affected by the split.<br />
\* @param string $taxonomy Taxonomy for the split term.<br />
\*/<br />
function wporg\_featured\_tags\_split($term\_id, $new\_term\_id, $term\_taxonomy\_id, $taxonomy)<br />
{<br />
// we only care about tags, so we'll first verify that the taxonomy is post\_tag.<br />
if ($taxonomy === 'post\_tag') {</p>
<p>// get the currently featured tags.<br />
$featured\_tags = get\_option('featured\_tags');</p>
<p>// if the updated term is in the array, note the array key.<br />
$found\_term = array\_search($term\_id, $featured\_tags);<br />
if ($found\_term !== false) {</p>
<p>// the updated term is a featured tag! replace it in the array, save the new array.<br />
$featured\_tags\[$found\_term\] = $new\_term\_id;<br />
update\_option('featured\_tags', $featured\_tags);<br />
}<br />
}<br />
}<br />
add\_action('split\_shared\_term', 'wporg\_featured\_tags\_split', 10, 4);<br />

[Expand full source code](#)[Collapse full source code](#)

#### Term ID stored in post meta

Let’s say your plugin stores a Term ID in post meta for pages so that you can show related posts for a certain page.

In this case, you need to use the [get\_posts()](https://developer.wordpress.org/reference/functions/get_posts/) function to get the pages with your `meta_key` and a update the `meta_value` matching the split term ID.

<br />
/\*\*<br />
\* Update related posts term ID for pages<br />
\*<br />
\* @param int $term\_id ID of the formerly shared term.<br />
\* @param int $new\_term\_id ID of the new term created for the $term\_taxonomy\_id.<br />
\* @param int $term\_taxonomy\_id ID for the term\_taxonomy row affected by the split.<br />
\* @param string $taxonomy Taxonomy for the split term.<br />
\*/<br />
function wporg\_page\_related\_posts\_split($term\_id, $new\_term\_id, $term\_taxonomy\_id, $taxonomy)<br />
{<br />
// find all the pages where meta\_value matches the old term ID.<br />
$page\_ids = get\_posts(\[<br />
'post\_type'  => 'page',<br />
'fields'     => 'ids',<br />
'meta\_key'   => 'meta\_key',<br />
'meta\_value' => $term\_id,<br />
\]);</p>
<p>// if such pages exist, update the term ID for each page.<br />
if ($page\_ids) {<br />
foreach ($page\_ids as $id) {<br />
update\_post\_meta($id, 'meta\_key', $new\_term\_id, $term\_id);<br />
}<br />
}<br />
}<br />
add\_action('split\_shared\_term', 'wporg\_page\_related\_posts\_split', 10, 4);<br />

[Expand full source code](#)[Collapse full source code](#)

### The `wp_get_split_term` function

Note:  
Using the `split_shared_term` hook is the preferred method for processing Term ID changes.

However, there may be cases where Terms are split without your plugin having a chance to hook to the `split_shared_term` action.

WP 4.2 stores information about Taxonomy Terms that have been split, and provides the [wp\_get\_split\_term()](https://developer.wordpress.org/reference/functions/wp_get_split_term/) utility function to help developers retrieve this information.

Consider the case above, where your plugin stores Term IDs in an option named `featured_tags`.  
You may want to build a function that validates these tag IDs (perhaps to be run on plugin update), to be sure that none of the featured tags has been split:

<br />
function wporg\_featured\_tags\_check\_split()<br />
{<br />
$featured\_tag\_ids = get\_option('featured\_tags', \[\]);</p>
<p>// check to see whether any IDs correspond to post\_tag terms that have been split.<br />
foreach ($featured\_tag\_ids as $index => $featured\_tag\_id) {<br />
$new\_term\_id = wp\_get\_split\_term($featured\_tag\_id, 'post\_tag');</p>
<p>if ($new\_term\_id) {<br />
$featured\_tag\_ids\[$index\] = $new\_term\_id;<br />
}<br />
}</p>
<p>// save<br />
update\_option('featured\_tags', $featured\_tag\_ids);<br />
}<br />

[Expand full source code](#)[Collapse full source code](#)

Note that [wp\_get\_split\_term()](https://developer.wordpress.org/reference/functions/wp_get_split_term/) takes two parameters, `$old_term_id` and `$taxonomy` and returns an integer.

If you need to retrieve a list of all split terms associated with an old Term ID, regardless of taxonomy, use [wp\_get\_split\_terms()](https://developer.wordpress.org/reference/functions/wp_get_split_terms/).