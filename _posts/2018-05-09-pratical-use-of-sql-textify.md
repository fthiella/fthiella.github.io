---
layout: post
title:  "Pratical use of Sql::Textify"
date:   2018-05-09 21:29:00 +0200
categories: sql perl textify
---
I know that perl is considered out of fashion nowadays, but for some tasks it's still a good and handy choice. Here's a practical use
of my module Sql::Textify. Every morning I need to check the status of some tasks, like the number of times my web services have been
called the day before, with some performance analysis and the number of errors. At the same time I want to know if my pentaho tasks are
all finished. And I want to know if all my backups are updated.

Luckily I managed to write all the info I need into some sql tables, updated automatically. So basically I just need to login to my
Adminer.php instance and run some queries or some views. But how if I need to share those info to my colleagues? I can quickly export
a dataseto to a Markdown text file, ready to be beautified with Markdown Here plugin.

But I wanted to make things more practical and faster. My initial idea was to make a Mason2 plugin that calls Sql::Textify and this
might still be a good idea to handle complex contents, but since my content is often simple and Mason2 is out of fashion anyways, I
wrote a simple script that handles everyting.

Here's an example. First we need a very simple template html page:

````html
<html>
<header>
<style>
..insert a good style..
</style>
<title>[[ $title ]]</title>
</header>
<body>
[[ $body ]]
</body>
</html>
````

then the perl script is like this:

````perl
use strict;
use warnings;
use SQL::Textify;
use File::Slurp;

my $t = Sql::Textify->new(
    conn => "dbi:SQLite:dbname=samples.db",
    username => "username",
    password => "password",
    format => 'html'
);

# read the template file
my $html = read_file( 'main.html' );

# set the title
my $title = "Report Indicizzazione";

# set the content of the main component
my $body = <<'BODY';
<h1>Daily report</h1>

<h2>Public Web Service</h2>

<% $t->textify("select * from view_web_service_status where eventdate>=currentdate"); %>

<h2>Pentaho Integrations Log</h2>

<% "select * from view_pentaho_log" | $t->textify %>
BODY

# first syntax, evaluates code between <% and %>
$body =~ s /\<\%\s+(.*?);\s+\%\>/$1/eeg;

# second syntax, apply $t->textify to the query (works as a filter)
$body =~ s /\<\%\s+(\".*?\")\s*\|(.*?)\s+\%\>/"$2\($1\)"/eeg;

# convert all [[ $variable ]] to the actual value
$html =~ s/\[\[ (\$\w*) \]\]/$1/eeg;

print $html;
````
This is not a perfect solution, but I just needed a quick tool to export my data and I wanted it to look good.
The syntax is inspired somehow to the Mason2 syntax.
A real Mason2/(or anything else) component of course is much more flexible but at the same time is little slower to write and more difficult to mantain.
