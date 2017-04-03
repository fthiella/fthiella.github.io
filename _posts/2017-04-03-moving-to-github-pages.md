---
layout: post
title:  "Moving to GitHub Pages"
date:   2017-04-03 22:05:00 +0200
categories: github
---
I have never been a big fan of the WYSIWYG paradigm: it looks fast and convenient at first,
but it becomes unmanageable at a later time. This applies to almost everything, like Word
processors, Web Pages editors, Grapgical programs, but also to blogging platforms. Have you
ever tried to insert some programming code in your articles, and edit the code at a later time?
Chances are that the code will become messed up, sooner or later.

That's why I like writing in [Markdown syntax](https://guides.github.com/pdfs/markdown-cheatsheet-online.pdf):
I can just concentrate on the contents without worrying how the document will look like, if it
looks nice in *Sublime Text* it will look great once converted to PDF using [StackEdit](http://stackedit.io)
or once converted to HTML using [Markdown-Here](http://markdown-here.com/).

I have also been generating all of my websites locally on my machine, with some combination
of *bash* + *perl* + *makefile* + *ImageMagic* + *php* + anything scrpitable + templates,
without the need of setting up a database (unless the web application *really* needs to access
dynamic data) making the whole process easyer to manage, more secure, and making the hosting
platform cheaper.

The idea behind the blogging platform [Jekyll](https://jekyllrb.com/) is exactly this.
You can write your posts in simple text files, using the Markdown syntax, and you can then
"compile" your website that you can just upload to your hosting provider. Or if you use GitHub pages
as a hosting provider, you can even skip the compile process because it will happen automatically
once you upload your files.

I followed this guide: [Build A Blog With Jekyll And GitHub Pages](https://www.smashingmagazine.com/2014/08/build-blog-jekyll-github-pages/)
and I am starting to use GitHub pages as my blogging platform.

The default style is okay, but it can be improved. For example Markdown tables are correctly generated, but there's no CSS
style associated to tables, so I just added to the `style.scss` file:

````css
table {
  padding: 0;
}
table tr {
  border-top: 1px solid #cccccc;
  background-color: white;
  margin: 0;
  padding: 0;
}

table tr:nth-child(2n) {
  background-color: #f8f8f8;
}

table tr th {
  font-weight: bold;
  border: 1px solid #cccccc;
  text-align: left;
  margin: 0;
  padding: 6px 13px;
}
table tr td {
  border: 1px solid #cccccc;
  text-align: left;
  margin: 0;
  padding: 6px 13px;
}
table tr th :first-child, table tr td :first-child {
  margin-top: 0;
}
table tr th :last-child, table tr td :last-child {
  margin-bottom: 0;
}
````

Having all articles sitting in my Hard Disk is great, I can use any text tool on them, like *grep* or I my own *SQL Markdown Builder*.

BTW, this is a trick I've been using for years, a blogging platform can be used not only as a personal blog, but also as a way to
display systems, applications, and database logs: some blogging platform like blogger supports publishing posts by email so you can
just configure your crontab jobs to send an email to a special address, and whenever blogger receives such email it will publish a new post
(to a private blog, of course). Appliances can be configured the same way. Since Jekyll blog posts are just text files on a filesystems,
those text files can be generated automatically.