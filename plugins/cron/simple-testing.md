# Simple Testing of WP-Cron

## WP-CLI

The simplest way to test cron jobs is with [WP-CLI](https://wp-cli.org/). It offers commands like `wp cron event list` and `wp cron event run {job name}`. [Check the documentation](https://developer.wordpress.org/cli/commands/cron/) for more details.

## Custom Plugin

You can also create a plugin to show all the currently scheduled WPCron tasks. In order to test this we will load the `wp-cron.php` file directly in our browser and output data to the display (otherwise we would have to perform some other action, maybe in the database, as the output is typically not shown on the site).

So let’s run through the initial steps to get this setup quickly.

### Create the plugin file

In the `wp-content/plugins` folder create a new folder called `my-wp-cron-test` and, inside this folder, a new file called `my-wp-cron-test.php`.

Open the PHP file for editing and insert the following lines:

<?php
/\*
Plugin Name: My WP-Cron Test
\*/

This text will setup the plugin for display and activation in the Plugins menu. Find the new plugin there, and activate it.

### View all currently scheduled tasks

WordPress has an undocumented function, [\_get\_cron\_array()](https://developer.wordpress.org/reference/functions/_get_cron_array/), that returns an array of all currently scheduled tasks. We are going to `var_dump` to show all the scheduled tasks.

Put the following code in the PHP file:

echo '<pre>'; print\_r( \_get\_cron\_array() ); echo '</pre>';

You can also put this code into an easy-to-call function like this:

function bl\_print\_tasks() {
    echo '<pre>'; print\_r( \_get\_cron\_array() ); echo '</pre>';
}
bl\_print\_tasks();

Note: Remember, the “bl\_” part of the function name is a *function prefix*. You can learn why prefixes are important [here](https://developer.wordpress.org/plugins/plugin-basics/best-practices/#prefix-everything).

The finished file looks like this:

<?php
/\*
Plugin Name: My WP-Cron Test
\*/

function bl\_print\_tasks() {
    echo '<pre>'; print\_r( \_get\_cron\_array() ); echo '</pre>';
}
bl\_print\_tasks();

### Testing the code

Open your browser and point it to `YOUR_SITE_URL/wp-cron.php`. You’ll see listed all the scheduled tasks for you installation.

This works because every time WordPress loads a page, it initializes all installed plugins by loading the main file for each plugin and running what it finds inside. In our case, the plugin is doing only one thing: showing the list of scheduled tasks.

Note: If you have installed WordPress into a subfolder, you’ll need to add that subfolder to the URL. This is something you would have to do intentionally, so if you don’t know what this means, you can probably ignore this :)