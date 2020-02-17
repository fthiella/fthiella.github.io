---
layout: post
title:  "Share a variable between components"
date:   2015-08-10 12:00:00 +0200
categories: mason perl learningmason
---
On my previous posts we saw how to to create and use true global variables. Their usage is limited to `$dbh` at the moment. But I also need a "global" variable, shared between all components of the same request. Here's one of my first tries, and this is how the `Base.mc` will look like:

{% highlight perl %}
{% raw %}
has 'title';
has 'authenticated_user';

<%augment wrap>
 <html>
 <head>
 <link rel="stylesheet" href="/static/css/style.css">
% $.Defer {{
 <title><% $.title %></title>
% }}
 </head>
 <body>
 <% inner() %>
 </body>
 </html>
<%init>
  $.authenticated_user('Admin');
</%init>
</%augment>
{% endraw %}
{% endhighlight %}

and this is the `index.mc`:

{% highlight perl %}
{% raw %}
Welcome, <% $.authenticated_user %>
<%init>
$.title("Welcome to Sentosa");
</%init>
{% endraw %}
{% endhighlight %}

I think I will use `Base.mp` and here's what I'm going to do:

{% highlight perl %}
{% raw %}
has 'authenticated_user';
has 'title';

method wrap() {
 $.authenticated_user ("Superuser");
 inner();
}
{% endraw %}
{% endhighlight %}

Next, how do I actually authenticate my users? See it on my next post!