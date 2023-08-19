# Testing of WP-Cron

## WP-CLI

Cron jobs can be tested using [WP-CLI](https://wp-cli.org/). It offers commands like `wp cron event list` and `wp cron event run {job name}`. [Check the documentation](https://developer.wordpress.org/cli/commands/cron/) for more details.

## WP-Cron Management Plugins

[Several plugins are available on the WordPress.org Plugin Directory for viewing, editing, and controlling the scheduled cron events and available schedules on your site.](https://wordpress.org/plugins/tags/cron/)

## `_get_cron_array()`

[The `_get_cron_array()` function](https://developer.wordpress.org/reference/functions/_get_cron_array/) returns an array of all currently scheduled cron events. Use this function if you need to inspect the raw list of events.

## `wp_get_schedules()`

[The `wp_get_schedules()` function](https://developer.wordpress.org/reference/functions/wp_get_schedules/) returns an array of available event recurrence schedules. Use this function if you need to inspect the raw list of available schedules.