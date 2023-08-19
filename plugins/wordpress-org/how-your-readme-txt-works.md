# Plugin Readmes

This page will explain some aspects of the plugin directory, and explain of the more obvious aspects which a lot of people miss.

To make your entry in the plugin browser most useful, each plugin should have a readme file named `readme.txt` that adheres to the [WordPress plugin readme file standard](https://wordpress.org/plugins/readme.txt). This file controls the output on the front-facing part of the directory. Writing a description in the readme determines exactly what will be displayed on `wordpress.org/plugins/Your-Plugin`

You can use the [plugin readme generator](https://generatewp.com/plugin-readme/) and put your completed result through the [official readme validator](https://wordpress.org/plugins/developers/readme-validator/) to check it. If you need more visual assistance you can use the tool [wpreadme.com](https://wpreadme.com/)

Since WordPress 5.8 plugin [readme files are not parsed for requirements](https://core.trac.wordpress.org/ticket/48520). This means that headers `Requires PHP` and `Requires at least` are going to be parsed from plugin’s main PHP file.

## Section Details

All plugins contain a main PHP file, and almost all plugins have a `readme.txt` file as well. The `readme.txt` file is intended to be written using a subset of markdown.

### Readme Header Information

The Plugin readme header consists of this information:

```adoc
=== Plugin Name ===
Contributors: (this should be a list of wordpress.org userid's)
Donate link: https://example.com/
Tags: tag1, tag2
Requires at least: 4.7
Tested up to: 5.4
Stable tag: 4.3
Requires PHP: 7.0
License: GPLv2 or later
License URI: https://www.gnu.org/licenses/gpl-2.0.html
Here is a short description of the plugin.  This should be no more than 150 characters.  No markup here.
```