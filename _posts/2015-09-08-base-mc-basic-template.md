---
layout: post
title:  "Base.mc: Basic template"
date:   2015-09-08 12:00:00 +0200
categories: mason perl learningmason
---

On my own machine I'm using a HTML Admin template I bought from [themeforest.net](http://themeforest.net): it's just a few dollars,
and I can concentrate on my Mason code, forgetting for a while about HTML, CSS, and JavaScript libraries. Of course I had to choose
a template that supports DataTables, we are going to use that a lot later.


Since the template I am using is not free and cannot be included, I choose this one:

http://startbootstrap.com/template-overviews/sb-admin-2/

which is free, quite complete and it looks good. The paid one is still better, but not that much better, and
for the purpose of this project the free template is great!


Both the free template I'm publicly using and the internal one that I've bought are based on bootstrap,
so they are interchangeable quite easily.

**Base.mc**

Base.mc is the superclass of most components of the project, the following flag is implicitly set:

````perl
<%flags>
  extends => 'Base.mc'
</%flags>
````

and it will provide the standard template for most of the components on the application.
This means that if my `index.mc` contains:

````html
<h1>Welcome to Sentosa Autoforms!</h1>
Hello, this is my new project,
work is still in progress
but please come back soon!
````

it will be nicely wrapped by the template provided in `Base.mc`, with the html, head, body, includes, and everything!

**The code**

This is my `Base.mc` - I've just removed something to make this post smaller.

I still have some doubts, here I'm using `title` and `_app` parameters, but what if some component is POSTing a `title` or an `_app` value? Anyways:

````perl
<%class>
has 'title';
has '_app';
</%class>

<%augment wrap><!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
% $.Defer {{
    <title><% $.title %></title>
% }}
    <& head_includes.mi &>
</head>
<body>
    <& body_includes.mi &>
    <div id="wrapper">
        <div id="page-wrapper">
            <% inner() %>
        </div>
    </div>
</body>
</html>
</%augment>
````

All CSS and JavaScript code that has to be included in the head section has been moved to the `head_includes.mi` component, all code that has to be included in the body section has been moved to `body_includes.mi`.

The inner component will be wrapped inside the template.

I had to remove the newline after augment wrap, otherwise it will appear as the first line on the generated HTML page:

````perl
<%augment wrap><!DOCTYPE html>
````

The title will be defined in an inner component, so we have to use defer:

````perl
% $.Defer {{
    <title><% $.title %></title>
% }}
````

I translated some components from HTML to Mason, for example I created a `dropdown_alerts.mc` component that calls the internal `dropdown_alerts.mi`:

````
<& dropdown_alerts.mi, items=>
    [
      {type=>'fa-comment', notification=>'New Comment', when=>'4 minutes ago'},
      {type=>'fa-twitter', notification=>'3 New Followers', when=>'7 minutes ago'},
      {type=>'fa-envelope', notification=>'Message Sent', when=>'12 minutes ago'},
      {type=>'fa-tasks', notification=>'New Task', when=>'5 minutes ago'},
      {type=>'fa-upload', notification=>'Server Rebooted', when=>'15 minutes ago'},
    ] &>
````

Isn't this nice? Yes! I only converted few components, because I'll have to use AJAX sooner or later.
Now I just wanted to have a template and I can move on working on my project!

Is it better to use dashes instead of underscores on component names?
I think... yes, this is the trend! I'll work on it.
