---
layout: post
title:  "DB.pm Library"
date:   2015-12-12 12:00:00 +0200
categories: mason perl learningmason
---
Hello, I'm finally back! Yes not much posts lately, but lots of coding - it's a lot considering
that I'm working full time and that this is only a side project on my spare time.

After some previous attempts, I decided that I had to write a library to manage queries, to apply
filters and searches, to move to the first/last/previous/next record, and to update and insert data.

I finally came out with `DB.pm` library, and even if it still needs to be improved I'm very proud
of how small, elegant and clean it is!

I might rename it to Sentosa::SQL as I might reuse it on other projects as well.

## Syntax

You need to define a $columns array reference as the following:

{% highlight perl %}
my $columns = [
  { col => 'id', 'pk' => 1},
  { col => 'name'},
  { col => 'surname'}
];
{% endhighlight %}

where id is the primary key, and then you have to call `selectQuery` and get a hash reference like this:

{% highlight sql %}
my $q = Sentosa::Db::selectQuery(
  'mytable',
  $columns,
  'ASC',
  undef,
  0, #offset
  10, #number of records
  'SQLite'
);
{% endhighlight %}

this will return three queries and three array references like below:

{% highlight perl %}
  print $q->{query}, "\n";
  print "(".join(',', @{$q->{query_data}}).")\n";

  print $q->{query_search}, "\n";
  print "(".join(',', @{$q->{query_search_data}}).")\n";

  print $q->{query_limit}, "\n";
  print "(".join(',', @{$q->{query_limit_data}}).")\n";
{% endhighlight %}

yes I could just call the same function thrice with different parameters (that would make the library even more elegant),
but at the moment I have good reasons to call it once, but I might change my mind in a near future.

- query: is the query, with a first level filter
- query_search: is the query with a first filter and second level search filter
- query_limit: is the query with a first level filter and a second level search filter, and also a limit on the number of rows (useful for pagination)

## Filters

There are two levels of filters, a main filter that is applied to all of the queries,
and an additional search filter that is applied only to `query_search` and `query_limit`.

This could be useful because a main filter can be applied to a table,
but the user can search for some records within the already filtered table.

For example, many users could have records in the same table:


ID | User | Song
---|------|---------------------------------
1  | 1    | AC-DC - Ride On.mp3
2  | 1    | Metallica - Enter Sandman.mp3
3  | 2    | The Sweet - Action.mp3
4  | 2    | Sixx::AM - Life Is Beautiful.mp3
5  | 2    | Metallica - Metal Militia.mp3

We can filter the previous table for each user, then the user can apply an additional filter:

{% highlight perl %}
my $columns = [
  { col => 'id', 'pk' => 1},
  { col => 'user', 'filter' => 1},
  { col => 'song', 'search' => 'Metallica', 'searchcriteria' => 'SUB'}
];
{% endhighlight %}