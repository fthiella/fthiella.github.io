---
layout: post
title:  "Tiles Page"
date:   2016-11-18 12:00:00 +0200
categories: mason perl learningmason
---

![Cruscotto]({{ site.baseurl }}/images/cruscotto-cot.jpg)

For my new application, I had to develop a summary page with all alarms in a single location:
a tiles page is perfect for this purpose!


Of course, data has to be fetched using AJAX, however I didn't have a tiles module ready
so this was the right moment to start developing one!


A tiles page is implemented like an unordered list:

````html
<ul class="tiles">
  <li class="blue high long">
    <a href="pazienti_cot">
      <span class='count'></span>
      <span class='name'>Pazienti Monitorati</span>
    </a>
  </li>
  <li>
    ...
  </li>
</ul>
````

I just added a `data-ajax-source` data column on the `ui` tag
(here's where we will fetch the counts) and a `data-value`
that specifies the row where to get the counts. The AJAX data source
will be in the same format as a datatables data source (because I'm lazy and
I can reuse the same module hehe):

````javascript
{
    "aaData":[
      ["PAZIENTI","1021"],
      ["TERR-RICOVERO","1"],
      ["TERR-ASSENZAPIANI","997"],
      ["OSP-DDP","5"],
      ["TERR-AGGDIARIOCA","30"],
      ["OSP-RICOVERO","24"],
      ["OSP-PS","4"],
      ["TERR-ACCESSOCA","5"],
      ["LETTI","10"]
    ],
    "iTotalRecords":"9",
    "iTotalDisplayRecords":"9",
    "sEcho":null
}
````
The tiles widget will be called as this:

````
  tiles-widget.mi,
    tiles => {
      json => "json/riepilogo.json",
      columns => [
        { caption => 'Pazienti Monitorati', value => 'PAZIENTI',          class => 'blue high long', href => 'pazienti_cot' },
        { caption => 'Pronto Soccotso',     value => 'OSP-PS',            class => 'red long',       href => '#' },
        { caption => 'Ricoveri',            value => 'OSP-RICOVERO',      class => 'brown long',     href => '#' },
        { caption => 'Aggiornamenti DDP',   value => 'OSP-DDP',           class => 'lime long',      href => '#' },
        { caption => 'Accesso Cont. Ass.',  value => 'TERR-ACCESSOCA',    class => 'orange long',    href => '#' },
        { caption => 'Agg. Diario',         value => 'TERR-AGGDIARIOCA',  class => 'teal long',      href => '#' },
        { caption => 'SVAMA',               value => 'TERR-SVAMA',        class => 'satgreen long',  href => '#' },
        { caption => 'Assenza Piani',       value => 'TERR-ASSENZAPIANI', class => 'satblue long',   href => '#' },
        { caption => 'Letti Disponibili',   value => 'LETTI',             class => 'lightred long',  href => 'letti_cot' },
      ]
    }
````
that will create a tiles html structure like the one above:

````
            <ul class="tiles" data-ajax-source="{json} %>">
% foreach my $box (@{$.tiles->{columns}}) {
               <li class="{class} %>" data-value="{value} %>">
                <a href="{href} %>">
                  <span class='count'></span>
                  <span class='name'>{caption} %></span>
                </a>
              </li>
% }
            </ul>
````

then the magic starts in the `sentosa.js` module:

````javascript
function init_tile(tile) {
    $.getJSON( tile.attr("data-ajax-source"), function( data ) {
        // create an hash from the json aaData
        var a = data["aaData"];
        var h = {};
        for (var i=0; i< a.length; i++) {
            h[ a[i][0] ] = a[i][1];
        };
        // loop through all elements, and set data from the hash
        // (yes, I could directly read aaData and put it inside the html... but I also want to put - where data is not avaliable, so I have to use an hash first)
        $("li", tile).each(function () {
            $('span.count', this).text(h[$(this).attr("data-value")] || '-');
        });
    });
}
````

okay, this "init" will load the JSON, parse it in a map data structure, then loop through
each `li` element and put the value accordingly to the `data-value` tag (or put the `-` symbol
if not available). I will then call it every some seconds:

````javascript
setInterval (function autoRefresh() {
    /* refresh all tiles */
    $('ul[data-ajax-source]').each(function () {
        init_tile($(this));
    });
}, 30000);
````

This link looks interesting if I want to build a more professional and reusable plugin:
[Basic Plugin Creation](https://learn.jquery.com/plugins/basic-plugin-creation/)

¡Hasta pronto!
