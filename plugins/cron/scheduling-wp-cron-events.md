# Scheduling WP Cron Events

The WP Cron system uses hooks to add new scheduled tasks.

## Adding the Hook

In order to get your task to run you must create your own custom hook and give that hook the name of a function to execute. This is a very important step. Forget it and your task will never run.

The following example will create a hook. The first parameter is the name of the hook you are creating, and the second is the name of the function to call.

```php
add_action( 'bl_cron_hook', 'bl_cron_exec' );
```

Note: Remember, the “bl\_” part of the function name is a *function prefix*. You can learn why prefixes are important [here](https://developer.wordpress.org/plugins/plugin-basics/best-practices/#prefix-everything).

Note: You can read more about actions [here](https://developer.wordpress.org/plugins/hooks/actions/).

## Scheduling the Task

An important note is that WP-Cron is a simple task scheduler. As we know, tasks are added by the hook created to call the function that runs the desired task. However if you call `wp_schedule_event()` multiple times, even with the same hook name, the event will be scheduled multiple times. If your code adds the task on each page load this could result in the task being scheduled several thousand times. This is not what you want.

WordPress provides a convenient function called [wp\_next\_scheduled()](https://developer.wordpress.org/reference/functions/wp_next_scheduled/) to check if a particular hook is already scheduled. `wp_next_scheduled()` takes one parameter, the hook name. It will return either a string containing the timestamp of the next execution or false, signifying the task is not scheduled. It is used like so:

```php
wp_next_scheduled( 'bl_cron_hook' )
```

Scheduling a recurring task is accomplished with [wp\_schedule\_event()](https://developer.wordpress.org/reference/functions/wp_schedule_event/) . This function takes three required parameters, and one additional parameter that is an array that can be passed to the function executing the wp-cron task. We will focus on the first three parameters. The parameters are as follows:

1.  `$timestamp` – The UNIX timestamp of the first time this task should execute
2.  `$recurrence` – The name of the interval in which the task will recur in seconds
3.  `$hook` – The name of our custom hook to call

We will use the 5 second interval we created [here](https://developer.wordpress.org/plugins/cron/understanding-wp-cron-scheduling/) and the hook we created above, like so:

```php
wp_schedule_event( time(), 'five_seconds', 'bl_cron_hook' );
```

Remember, we need to first ensure the task is not already scheduled. So we wrap the scheduling code in a check like this:

```php
if ( ! wp_next_scheduled( 'bl_cron_hook' ) ) {
    wp_schedule_event( time(), 'five_seconds', 'bl_cron_hook' );
}
```

## Unscheduling tasks

When you no longer need a task scheduled you can unschedule tasks with [wp\_unschedule\_event()](https://developer.wordpress.org/reference/functions/wp_unschedule_event/) . This function takes the following two parameters:

1.  `$timestamp` – Timestamp of the next occurrence of the task
2.  `$hook` – Name of the custom hook to be called

This function will not only unschedule the task indicated by the timestamp, it will also unschedule all future occurrences of the task. Since you probably will not know the timestamp for the next task, there is another handy function, [wp\_next\_scheduled()](https://developer.wordpress.org/reference/functions/wp_next_scheduled/) that will find it for you. `wp_next_scheduled()` takes one parameter (that we care about):

1.  `$hook` – The name of the hook that is called to execute the task

Put it all together and the code looks like:

```php
$timestamp = wp_next_scheduled( 'bl_cron_hook' );
wp_unschedule_event( $timestamp, 'bl_cron_hook' );
```

It is very important to unschedule tasks when you no longer need them because WordPress will continue to attempt to execute the tasks, even though they are no longer in use (or even after your plugin has been deactivated or removed). An important place to remember to unschedule your tasks is upon plugin deactivation.

Unfortunately there are many plugins in the WordPress.org Plugin Directory that do not clean up after themselves. If you find one of these plugins please let the author know to update their code. WordPress provides a function called [register\_deactivation\_hook()](https://developer.wordpress.org/reference/functions/register_deactivation_hook/) that allows developers to run a function when their plugin is deactivated. It is very simple to setup and looks like:

```php
register_deactivation_hook( __FILE__, 'bl_deactivate' ); 

function bl_deactivate() {
    $timestamp = wp_next_scheduled( 'bl_cron_hook' );
    wp_unschedule_event( $timestamp, 'bl_cron_hook' );
}
```

Note: You can read more about activation and deactivation hooks [here](https://developer.wordpress.org/plugins/plugin-basics/activation-deactivation-hooks/).