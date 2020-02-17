---
layout: post
title:  "Markdown filter for Mason2"
date:   2017-08-25 13:59:00 +0200
categories: perl mason
---
Mason comes with some built-in filters that can be used to process portions of content in a component.
The standard way to invoke a filter is in a block:

{% highlight perl %}
    % $.Trim \{\{
        This string will be trimmed
    % \}\}
    # end Trim
{% endhighlight %}

but filters can appear also inside a `<% %>` tag:


{% highlight perl %}
<% $content | NoBlankLines,Trim %>
{% endhighlight %}

More details can be found at the official [Mason::Manual::Filters](https://metacpan.org/pod/distribution/Mason/lib/Mason/Manual/Filters.pod) page.
Here's how to create a custom filter:

{% highlight perl %}
package MyApp::Filters;
use Mason::PluginRole;
    
method Upper () {
    return sub { uc($_[0]) }
}
    
method Lower () {
    return sub { lc($_[0]) }
}
    
1;
{% endhighlight %}

I wanted to use a custom filter that converts Markdown text to HTML, so I installed the [Text::Multimarkdown](https://metacpan.org/module/Text::MultiMarkdown)
module and I created a filter like this:


{% highlight perl %}
package MyApp::Filters;
use Mason::PluginRole;
use Text::MultiMarkdown qw(markdown);

method Markdown () {
    my $m = Text::Markdown->new;
    return sub { markdown($_[0]) }
}

1;
{% endhighlight %}
which I put in `lib/MyApp` directory. Then I tried it in a `index.mc` page:


{% highlight perl %}
<%class>
  with 'Markdown::Filters';
</%class>

<%
"
## Markdown

This page is written using **markdown** syntax:

- it's using the `Text::MultiMarkdown` library
- it's a lot of fun
- `Mason2` is great!
" | Markdown
%>
{% endhighlight %}

another way using the block invocation syntax:


{% highlight perl %}
<%class>
  with 'Markdown::Filters';
</%class>

<h1>Let's try the block invocation syntax</h1>

% $.Markdown \{\{
## Do tables work?

id | description
---|------------
01 | begin at the beginning
02 | go on till you come to the end
03 | then stop

yes `MultiMarkdown` supports tables as well.
% \}\}
{% endhighlight %}

one more way is to use a Base component, telling to process all inner components with Markdown:


{% highlight perl %}
<%class>
  with 'Markdown::Filters';
</%class>

<%augment wrap>
  <html>
    <head>
      <link rel="stylesheet" href="/static/css/style.css">
      <title>Mason and Markdown</title>
    </head>
    <body>
% $.Markdown \{\{
      <% inner() %>
% \}\}
    </body>
  </html>
</%augment>
{% endhighlight %}

then your page can contain just Markdown syntax, here's an `index.mc` example page (an empy line at the beginning seems to be needed here):


{% highlight perl %}


# Only Markdown

This page will only contain Markdown syntax, but it will be converted to HTML:

% foreach my $i (qw(one two three)) {
  - <% $i %>
% }

yes you can still use perl code on it!
{% endhighlight %}
