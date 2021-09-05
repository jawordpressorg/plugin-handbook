# Summary

Here are all the example code snippets from the preceding discussion, assembled into two complete code pages: one for jQuery and the other for PHP.

#### PHP

This code resides on one of your plugin pages.

add\_action( 'admin\_enqueue\_scripts', 'my\_enqueue' );  
function my\_enqueue( $hook ) {  
   if ( 'myplugin\_settings.php' !== $hook ) {  
      return;  
   }  
   wp\_enqueue\_script(  
      'ajax-script',  
      plugins\_url( '/js/myjquery.js', *\_\_FILE\_\_* ),  
      array( 'jquery' ),  
      '1.0.0',  
      true  
   );  
   $title\_nonce = wp\_create\_nonce( 'title\_example' );  
   wp\_localize\_script(  
      'ajax-script',  
      'my\_ajax\_obj',  
      array(  
         'ajax\_url' => admin\_url( 'admin-ajax.php' ),  
         'nonce'    => $title\_nonce,  
      )  
   );  
}  
  
add\_action( 'wp\_ajax\_my\_tag\_count', 'my\_ajax\_handler' );  
function my\_ajax\_handler() {  
   check\_ajax\_referer( 'title\_example' );  
   update\_user\_meta( get\_current\_user\_id(), 'title\_preference', $\_POST\['title'\] );  
   $args      = array(  
      'tag' => $\_POST\['title'\],  
   );  
   $the\_query = new WP\_Query( $args );  
   echo $\_POST\['title'\] . ' (' . $the\_query->post\_count . ') ';  
   wp\_die(); *// all ajax handlers should die when finished  
*}

#### jQuery

This code is in the file `js/myjquery.js` below your plugin folder.

</p>
jQuery(document).ready(function($) { 	   //wrapper
	$(".pref").change(function() { 		   //event
		var this2 = this; 		           //use in callback
		$.post(my\_ajax\_obj.ajax\_url, { 	   //POST request
	       \_ajax\_nonce: my\_ajax\_obj.nonce, //nonce
			action: "my\_tag\_count",        //action
	  		title: this.value 	           //data
  		}, function(data) {		           //callback
			this2.nextSibling.remove();    //remove the current title
			$(this2).after(data); 	       //insert server response
		});
	});
});
<p>

[Expand full source code](#)[Collapse full source code](#)

And after storing the preference, the resulting post count is added to the selected title.

## More Information

*   [How To Use AJAX In WordPress](http://wp.smashingmagazine.com/2011/10/18/how-to-use-ajax-in-wordpress/ "External Site")
*   [AJAX for WordPress](http://www.glennmessersmith.com/pages/wpajax.html "External Site")