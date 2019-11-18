---
layout: post
title:  "Dump Markdown plugin for Adminer"
date:   2018-01-11 23:03:00 +0200
categories: sql php adminer
---
Few months ago I started working on a Markdown dump plugin for [Adminer](http://www.adminer.org), now I can finally write a post about it.

I am using Markdown almost every day, I use it for writing blog posts (like this one), I am using it for writing documents, wikis,
report, notes, and also for composing e-mails - thanks to the powerful plugin [Markdown Here](https://markdown-here.com/) - so I needed
a tool to quickly export SQL tables and queries from Adminer.php to Markdown tables.

Another tool I've been working on is my perl module [SQL::Textify](https://metacpan.org/pod/Sql::Textify), it is essentially an
improved version of my old *SQL Markdown Builder* tool (now considered obsolete), and I find it great to run SQL query from the
command line, and get the result in Markdown, HTML, JSON format.

## Install the plugin

I am using a directory `/var/www/tools` where I put all of my tools which I want to be accessible through a web interface.
This directory will be published at `https://localhost/tools`, make sure that php files put here will be handled correctly.

There I my `adminer.php` file which will load the original `adminer.php` along with my own plugin plus other plugins I've downloaded:

### adminer.php

{% highlight php %}
<?php
function adminer_object() {
    // required to run any plugin
    include_once "./plugins/plugin.php";

    // autoloader
    foreach (glob("plugins/*.php") as $filename) {
        include_once "./$filename";
    }

    $plugins = array(
        // specify enabled plugins here
        new AdminerDumpMarkdown,
    );

    /* It is possible to combine customization and plugins:
    class AdminerCustomization extends AdminerPlugin {
    }
    return new AdminerCustomization($plugins);
    */

    return new AdminerPlugin($plugins);
}

// include original Adminer or Adminer Editor
include "./adminer-4.3.1-en.php";
?>
{% endhighlight %}

on the same directory I have put the original adminer `adminer-4.3.1-en.php`, and 
in the directory `plugins` I have put the `plugin.php` file, which is required to run any plugin,
and my own plugin `dump-markdown.php`:

- `plugin.php` can be downloaded from [the adminer plugins page](https://www.adminer.org/en/plugins/)
- `dump-markdown.php` can be downloaded from [my github](https://raw.githubusercontent.com/fthiella/adminer-plugin-dump-markdown/master/dump-markdown.php)

you can make sure that only the `adminer.php` will be accessible to the outside world while any other file will be blocked.

# dumpFormat() function

This function will return the Markdown output options:

{% highlight php %}
return array('markdown' => 'Markdown');
{% endhighlight %}

this output will be added to the existing list.

# dumpTable() function

This function will add slashes to the table name before some special characters:

- \r
- \n
- \"
- \\

and will return it as a header h2 (preceded by two \#\#). The `return true;` will make sure that
the default output of Adminer won't be processed.

The header of the table won't be printed here, as at this point we still don't know the width of
every single column.

# dumpData() function

Here is where Markdown table will be printed to the output. This function will sample the first 100 rows to calculate
the width of every column, then it will start to output all sampled rows and then every single additional row.

Here we also use `return true;` so the default output of Adminer won't be processed.

# To Do

There are few things I want to implement:

- add slashes before every special character of every single field. There's no a "standard" way to quote Markdown strings and there's not
a standard list of special character, I am trying to get best results with stackedit.io, Markdown Here, other php and perl libraries;
- add some parameters in the query, to control its output e.g. maximum width, record or table format, etc. same as SQL::Textify
- add groups and levels in the output query. This will be implemented in SQL::Textify also.