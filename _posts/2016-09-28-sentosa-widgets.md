---
layout: post
title:  "Sentosa Widgets"
date:   2016-09-22 12:00:00 +0200
categories: mason perl learningmason
---
Query and Forms are provided by the `query.mi` and the `form.mi` components.

Initially all the code for each object was inside a single component, but later on I decided to separate it into two parts.

The first half just loads the object definition (query, or form) from the Sentosa database using the `Objects.pm` library:

````perl
my ($obj, $columns) = Sentosa::Objects::get_recordSource({
    app => $._app,
    obj => $._id,
    type => 'query',
    userid => $m->session->{auth_id}
});
if (!$obj) { $m->not_found(); }; # form not found
````

and then calls the `query-widget.mi` component which renders the actual data (the JSON output of the query in this case):

````perl
 {
      app => $._app,
      id => $._id,
      description => $obj->{description},
      pk => $obj->{pk},
      columns => $columns,
      params => {
        'table_link'  => $.table_link,
        'hide_title'  => $.hide_title,
        'hide_top'    => $.hide_top,
        'hide_bottom' => $.hide_bottom
      }
    }
&>
````

the reason is that I like a lot the components that render the data, but I don't like
to read Sentosa components from a DB. I'm still undecided if the problem is just the
database structure that's not too intuitive, or if it's just quicker to define objects
in the filesystem (but maybe I need to hack Mason somehow...).

Anyway this also helps Sentosa be more flexible (I'm using it at work for some new projects!).

What I've noticed is that I have never documented the `$columns` format yet.
That's definitely going to be the subject of my next post!