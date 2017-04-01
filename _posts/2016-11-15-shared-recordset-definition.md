---
layout: post
title:  "Shared Recordset Definition"
date:   2016-11-15 12:00:00 +0200
categories: mason perl learningmason
---
I think that the modules I developed for Sentosa Autoforms are quite elegant and flexible (expecially the Sentosa::SQL module, and the DataDatables and JSON modules) so I decided to reuse them! However I don't like the fact that Sentosa Autoforms is all driven by database data.. while it's sometimes very convenient, I don't like the layer of complexity that it adds.

So I started to work on it, trying to figure out a simpler solution!

First of all, I needed to store all recordset information on a variable, shared to all components. The hashref `recordset` goes to the `Base.mp` pure perl component:

````perl
my $recordset = {
  # Allarms
  "alarms" => {
    description => 'Alarms',
    connection => { db => 'dbi:Pg:dbname=dbname;host=10.10.10.10', username => 'myusername', password => 'mypassword' },
    source => "alarms",
    pk => 'id',
    columns => [
        { col => 'paziente',                 caption => 'Paziente', type => 'text', 'link-id' => '4' },
        { col => 'data_nascita',             caption => 'Data Nasc', type => 'text' },
        { col => 'dataora',                  caption => 'Data/ora PS', type => 'text' },
        { col => 'osp_ps_priorita_ingresso', caption => 'Priorità', type => 'text' },
        { col => 'link_fascicolo',           caption => 'Fascicolo', type => 'hidden' }
      ],
    json => 'json/alarms.json'
  },
}
````

Pretty elegant! Also I can use all kinds of Perl stuff to make it more tidy or more compact (e.g. I can specify a connections hash_ref, and I can calculate the array_ref columns somehow.. also, Sentosa has to process every query every time we access a recordset, while with this systems I can pre-calculate all queries at runtime the first time we access it).

How can I share this recordset data to all components? I can specify a method at the end of the `Base.mp` component:

````perl
method getrecordset($rs) {
  return $recordset->{$rs};
}
````

and then the default handler inside the json directory can be just like this:

````perl
<%flags>
extends => '../Base.mp';
</%flags>
%class
  has '_id';
  has '_app';

  has 'iDisplayStart';
  has 'iDisplayLength';
  has 'iColumns';

  has 'sEcho';
</%class>
<%init>
  my ($recordset_name, $ext) = $m->path_info =~ /(.*)(\.json)$/;
  if (! $.getrecordset($recordset_name)) {
    $m->not_found();
  }
</%init>
<&
  ../query-data-widget.mi,
    obj => {
      source => $.getrecordset($recordset_name)->{source},
      pk     => $.getrecordset($recordset_name)->{pk},
      db     => $.getrecordset($recordset_name)->{connection}->{db},
      name   => 'allarmi_osp_ricovero',
      description => $.getrecordset($recordset_name)->{description},
      username => $.getrecordset($recordset_name)->{connection}->{username},
      password => $.getrecordset($recordset_name)->{connection}->{password}
    },
    columns => $.getrecordset($recordset_name)->{columns},

    iDisplayStart => $.iDisplayStart,
    iDisplayLength => $.iDisplayLength,
    iColumns => $.iColumns,
    sEcho => $.sEcho,
    searchArgs => $.args
&>
````

it will inherit only from the Base.mp (no html, only perl!) ... and yes, I know this is a little ugly and not too elegant, but at least I can use the query-data-widget (and the query-widget as well) without major updates. I will work on making it better :)

*Ah only one thing has to be updated!* instead of working on the `$.columns` object I figured out that I have to work on a local copy of the object:

````perl
    use Clone 'clone';
    my $local_columns = clone($.columns);
````
otherwise, everytime I modify the columns object to add a temporary filter, the next time the filter is still there - because it's a hash_ref!

To call the query widget I use this:

````perl
<&
  query-widget.mi,
    query => {
      id => 'osp-ps',
      json => $.getrecordset($recordset_name)->{json},
      description => $.getrecordset($recordset_name)->{description},
      pk => $.getrecordset($recordset_name)->{pk},
      columns => $.getrecordset($recordset_name)->{columns},
      params => { 'table_link'  => undef, 'hide_title'  => 1, 'hide_top'    => 1, 'hide_bottom' => 0 }
    }
&>
````

Well this environment is much more comfortable, I think I can add plenty of improvements soon, and I think I can write a good manual of how to use those components. See you soon!
