---
layout: post
title:  "Custom Sentosa System Database"
date:   2016-09-22 12:00:00 +0200
categories: perl markdown sql
---

Sentosa Autoforms by default stores all of its users, settings and objects in a local SQLite database.
The default connection is defined in `lib\Sentosa\Import.pm` module:

{% highlight perl %}
package Sentosa::Import;
use Poet::Moose;
extends 'Poet::Import';

use DBI;

method provide_var_dbh ($caller) {
  return DBI->connect(
    'dbi:SQLite:dbname=data/sentosa.db',
    '',
    '',
    { RaiseError => 1 },
  ) or die $DBI::errstr;
}

1;
{% endhighlight %}

this module provides a shared variable `$dbh` that contains an active DBI handler to the standard SQLite database,
you can however modify it in order to use any other database. This is what I use for PostgreSQL:

{% highlight perl %}
package Sentosa::Import;
use Poet::Moose;
extends 'Poet::Import';

use DBI;

my $mydbh = DBI->connect(
    'dbi:Pg:dbname=xxxx;host=xxxx',
    'username',
    'password',
    { RaiseError => 1 },
  ) or die $DBI::errstr;

method provide_var_dbh ($caller) {
  $mydbh->do("SET search_path TO sentosa");
  return $mydbh;
}

1;
{% endhighlight %}

Since I stored Sentosa tables into a PostgreSQL schema called `sentosa` I had to modify
a little the previous module in order to use the correct schema:

{% highlight sql %}
SET search_path TO sentosa
{% endhighlight %}
