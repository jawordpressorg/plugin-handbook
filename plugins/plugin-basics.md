<!--
# Plugin Basics
-->
# プラグインの基本

<!--
## Getting Started
-->
## はじめに

<!--
At its simplest, a WordPress plugin is a PHP file with a WordPress plugin header comment. It’s highly recommended that you create a directory to hold your plugin so that all of your plugin’s files are neatly organized in one place.
-->
WordPress プラグインは、WordPress プラグインヘッダーコメントを持つ PHP ファイルです。プラグインを保存するディレクトリを作成し、プラグインのすべてのファイルを一箇所にきちんとまとめることを強くお勧めします。

<!--
To get started creating a new plugin, follow the steps below.
-->
新しいプラグインの作成を始めるには、以下の手順に従ってください。

<!--
1.  Navigate to the WordPress installation’s **wp-content** directory.

3.  Open the **plugins** directory.

5.  Create a new directory and name it after the plugin (e.g. `plugin-name`).

7.  Open the new plugin’s directory.

9.  Create a new PHP file (it’s also good to name this file after your plugin, e.g. `plugin-name.php`).
-->

1.WordPress の **wp-content** ディレクトリに移動します。

2.**plugins** ディレクトリを開く。

3.新しいディレクトリを作成し、プラグインの名前を付けます (例: plugin-name)。

4.新しいプラグインのディレクトリを開きます。

5.新しい PHP ファイルを作成します (このファイルにもプラグインの名前を付けるとよいでしょう、plugin-name.php など)。

<!--
Here’s what the process looks like on the Unix command line:
-->
Unixのコマンドラインでは次のようになります:

```bash
wordpress $ cd wp-content
wp-content $ cd plugins
plugins $ mkdir plugin-name
plugins $ cd plugin-name
plugin-name $ vi plugin-name.php
```
<!--
In the example above, `vi` is the name of the text editor. Use whichever editor that is comfortable for you.
-->
上の例では、vi はテキストエディターの名前です。あなたが使いやすいエディターを使用してください。

<!--
Now that you’re editing your new plugin’s PHP file, you’ll need to add a plugin header comment. This is a specially formatted PHP block comment that contains metadata about the plugin, such as its name, author, version, license, etc. The plugin header comment must comply with the [header requirements](https://developer.wordpress.org/plugins/the-basics/header-requirements/), and at the very least, contain the name of the plugin.
-->
新しいプラグインの PHP ファイルを編集しているので、プラグインヘッダーコメントを追加する必要があります。これは特別な書式の PHP ブロックコメントで、プラグインの名前、作者、バージョン、ライセンスなどのメタデータを含みます。プラグインヘッダーコメントはヘッダー要件に従わなければならず、最低限プラグインの名前を含まなければなりません。

<!--
Only one file in the plugin’s folder should have the header comment — if the plugin has multiple PHP files, only one of those files should have the header comment.
-->
プラグインのフォルダー内の1つのファイルだけがヘッダーコメントを持つようにします - プラグインに複数の PHP ファイルがある場合は、そのうちの1つのファイルだけがヘッダーコメントを持つようにします。

<!--
After you save the file, you should be able to see your plugin listed in your WordPress site. Log in to your WordPress site, and click **Plugins** on the left navigation pane of your WordPress Admin. This page displays a listing of all the plugins your WordPress site has. Your new plugin should now be in that list!
-->
ファイルを保存すると、WordPress サイトにプラグインが表示されるはずです。WordPress サイトにログインし、WordPress 管理画面の左側のナビゲーションペインにある「プラグイン」をクリックします。このページには、WordPress サイトにあるすべてのプラグインのリストが表示されます。新しいプラグインはこのリストにあるはずです !

<!--
## Hooks: Actions and Filters
-->
## フックアクションとフィルター

<!--
WordPress hooks allow you to tap into WordPress at specific points to change how WordPress behaves without editing any core files.
-->
WordPress のフックを使うと、コアファイルを編集することなく WordPress の動作を変更することができます。

<!--
There are two types of hooks within WordPress: *actions* and *filters*. Actions allow you to add or change WordPress functionality, while filters allow you to alter content as it is loaded and displayed to the website user.
-->
WordPress のフックには、*アクション*と*フィルター*の2種類があります。アクションは、WordPress の機能を追加または変更することができ、フィルターはそれがロードされ、Web サイトのユーザーに表示されるようにコンテンツを変更することができます。

<!--
Hooks are not just for plugin developers; hooks are used extensively to provide default functionality by WordPress core itself. Other hooks are unused place holders that are simply available for you to tap into when you need to alter how WordPress works. This is what makes WordPress so flexible.
-->
フックはプラグイン開発者だけのものではなく、WordPress のコア自体がデフォルトの機能を提供するために広く使われています。他のフックは未使用のプレースホルダーで、WordPress の動作を変更する必要があるときに利用できます。これが WordPress の柔軟性を高めているのです。

<!--
### Basic Hooks
-->
### 基本的なフック

<!--
The 3 basic hooks you’ll need when creating a plugin are the [register\_activation\_hook()](https://developer.wordpress.org/reference/functions/register_activation_hook/) , the [register\_deactivation\_hook()](https://developer.wordpress.org/reference/functions/register_deactivation_hook/) , and the [register\_uninstall\_hook()](https://developer.wordpress.org/reference/functions/register_uninstall_hook/) .
-->
プラグインを作成するときに必要な3つの基本フックは、[register\_activation\_hook()](https://developer.wordpress.org/reference/functions/register_activation_hook/)、[register\_deactivation\_hook( )](https://developer.wordpress.org/reference/functions/register_deactivation_hook/)、および [register\_uninstall\_hook()](https://developer.wordpress.org/reference/functions/register_uninstall_hook/)。

<!--
The [activation hook](https://developer.wordpress.org/plugins/the-basics/activation-deactivation-hooks/) is run when you *activate* your plugin. You would use this to provide a function to set up your plugin — for example, creating some default settings in the `options` table.
-->
[アクティベーションフック](https://developer.wordpress.org/plugins/the-basics/activation-deactivation-hooks/) は、プラグインを *アクティベート* するときに実行されます。これを使用して、プラグインをセットアップする機能を提供します。たとえば、「options」テーブルにいくつかのデフォルト設定を作成します。

<!--
The [deactivation hook](https://developer.wordpress.org/plugins/the-basics/activation-deactivation-hooks/) is run when you *deactivate* your plugin. You would use this to provide a function that clears any temporary data stored by your plugin.
-->
[アクティベーションフック](https://developer.wordpress.org/plugins/the-basics/activation-deactivation-hooks/) は、プラグインを *アクティベート* するときに実行されます。これを使用して、プラグインをセットアップする機能を提供します。たとえば、「options」テーブルにいくつかのデフォルト設定を作成します。

<!--
These [uninstall methods](https://developer.wordpress.org/plugins/the-basics/uninstall-methods/) are used to clean up after your plugin is *deleted* using the WordPress Admin. You would use this to delete all data created by your plugin, such as any options that were added to the `options` table.
-->
[非アクティブ化フック](https://developer.wordpress.org/plugins/the-basics/activation-deactivation-hooks/) は、プラグインを *非アクティブ化*すると実行されます。これを使用して、プラグインによって保存された一時データをクリアする関数を提供します。

<!--
### Adding Hooks
-->
### フックの追加

<!--
You can add your own, custom hooks with [do\_action()](https://developer.wordpress.org/reference/functions/do_action/) , which will enable developers to extend your plugin by passing functions through your hooks.
-->
[do\_action()](https://developer.wordpress.org/reference/functions/do_action/) を使用して独自のカスタム フックを追加できます。これにより、開発者はフックを介して関数を渡すことによってプラグインを拡張できます。

<!--
### Removing Hooks
-->
### フックの取り外し

<!--
You can also use invoke [remove\_action()](https://developer.wordpress.org/reference/functions/remove_action/) to remove a function that was defined earlier. For example, if your plugin is an add-on to another plugin, you can use [remove\_action()](https://developer.wordpress.org/reference/functions/remove_action/) with the same function callback that was added by the previous plugin with [add\_action()](https://developer.wordpress.org/reference/functions/add_action/) . The priority of actions is important in these situations, as [remove\_action()](https://developer.wordpress.org/reference/functions/remove_action/) would need to run after the initial [add\_action()](https://developer.wordpress.org/reference/functions/add_action/) .
-->
invoke [remove\_action()](https://developer.wordpress.org/reference/functions/remove_action/) を使用して、以前に定義された関数を削除することもできます。たとえば、プラグインが別のプラグインのアドオンである場合、[remove\_action()](https://developer.wordpress.org/reference/functions/remove_action/) を同じ関数コールバックで使用できます。[add\_action()](https://developer.wordpress.org/reference/functions/add_action/) を使用して前のプラグインによって追加されました。[remove\_action()](https://developer.wordpress.org/reference/functions/remove_action/) は最初の [add\_action()] の後に実行する必要があるため、このような状況ではアクションの優先順位が重要です。(https://developer.wordpress.org/reference/functions/add_action/) 。

<!--
You should be careful when removing an action from a hook, as well as when altering priorities, because it can be difficult to see how these changes will affect other interactions with the same hook. We highly recommend testing frequently.
-->
フックからアクションを削除するときや優先順位を変更するときは注意が必要です。これらの変更が同じフックとの他の対話にどのような影響を与えるかを確認するのが難しい場合があるためです。頻繁にテストすることを強くお勧めします。

<!--
You can learn more about creating hooks and interacting with them in the [Hooks](https://developer.wordpress.org/plugin/hooks/) section of this handbook.
-->
フックの作成とその操作について詳しくは、このハンドブックの [フック](https://developer.wordpress.org/plugin/hooks/) セクションをご覧ください。

<!--
## WordPress APIs
-->
## WordPress API

<!--
Did you know that WordPress provides a number of [Application Programming Interfaces (APIs)](https://make.wordpress.org/core/handbook/core-apis/)? These APIs can greatly simplify the code you need to write in your plugins. You don’t want to reinvent the wheel, especially when so many people have done a lot of the work and testing for you.
-->
WordPress が多数の [アプリケーション プログラミング インターフェイス (API)](https://make.wordpress.org/core/handbook/core-apis/) を提供していることをご存知ですか ? これらの API を使用すると、プラグインで記述する必要があるコードを大幅に簡素化できます。特に多くの人があなたのために多くの作業やテストを行ってくれた場合には、車輪の再発明はしたくないでしょう。

<!--
The most common one is the [Options API](https://codex.wordpress.org/Options_API), which makes it easy to store data in the database for your plugin. If you’re thinking of using [cURL](https://en.wikipedia.org/wiki/CURL) in your plugin, the [HTTP API](https://codex.wordpress.org/HTTP_API) might be of interest to you.
-->
最も一般的なのは [Options API](https://codex.wordpress.org/Options_API) で、これを使用すると、プラグインのデータベースにデータを簡単に保存できます。プラグインで [cURL](https://en.wikipedia.org/wiki/CURL) を使用することを考えている場合、[HTTP API](https://codex.wordpress.org/HTTP_API) は役に立つでしょう。

<!--
Since we’re talking about plugins, you’ll want to study the [Plugin API](https://codex.wordpress.org/Plugin_API). It has a variety of functions that will assist you in developing plugins.
-->
ここではプラグインについて話しているので、[プラグイン API](https://codex.wordpress.org/Plugin_API) について勉強してください。プラグインの開発を支援するさまざまな機能が備わっています。

<!--
## How WordPress Loads Plugins
-->
## WordPress がプラグインをロードする方法

<!--
When WordPress loads the list of installed plugins on the Plugins page of the WordPress Admin, it searches through the `plugins` folder (and its sub-folders) to find PHP files with WordPress plugin header comments. If your entire plugin consists of just a single PHP file, like [Hello Dolly](https://wordpress.org/plugins/hello-dolly/ "Hello Dolly"), the file could be located directly inside the root of the `plugins` folder. But more commonly, plugin files will reside in their own folder, named after the plugin.
-->
WordPress が WordPress Admin の プラグインページにインストールされているプラグインのリストを読み込むと、「plugins」フォルダー (およびそのサブフォルダー) を検索して、WordPress プラグイン ヘッダー コメントを含む PHP ファイルを見つけます。[Hello Dolly](https://wordpress.org/plugins/hello-dolly/ "Hello Dolly") のように、プラグイン全体が1つの PHP ファイルのみで構成されている場合、ファイルは ` のルート内に直接配置できます。プラグインフォルダー。ただし、より一般的には、プラグイン ファイルは、プラグインにちなんで名付けられた独自のフォルダーに存在します。

<!--
## Sharing your Plugin
-->
## プラグインを共有する

<!--
Sometimes a plugin you create is just for your site. But many people like to share their plugins with the rest of the WordPress community. Before sharing your plugin, one thing you need to do is [choose a license](https://opensource.org/licenses/category). This lets the user of your plugin know how they are allowed to use your code. To maintain compatibility with WordPress core, it is recommended that you pick a license that works with GNU General Public License (GPLv2+).
-->
作成したプラグインがサイト専用である場合があります。しかし、多くの人は自分のプラグインを WordPress コミュニティの他のメンバーと共有したいと考えています。プラグインを共有する前に、[ライセンスを選択](https://opensource.org/licenses/category) する必要があります。これにより、プラグインのユーザーはコードの使用がどのように許可されているかを知ることができます。WordPress コアとの互換性を維持するには、GNU General Public License (GPLv2+) で動作するライセンスを選択することをお勧めします。

