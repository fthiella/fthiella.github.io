---
layout: post
title:  "Show text files width"
date:   2018-08-10 14:24:00 +0200
categories: perl
---
I am often working with fixed width text files. Sometimes text files aren't produced properly, so I have to make sure that all rows share the same width.

This is one liner perl thay I use quite often, on a Windows shell:

{% highlight cmd %}
c:\> perl -lne "$h{length($_)}=1; END{print join \"\n\", sort keys %h}" source.txt
{% endhighlight %}

this is how it looks like in Linux, with a simpler way of quoting:

{% highlight bash %}
$ perl -lne '$h{length($_)}=1; END{print join "\n", sort keys %h}' source.txt
{% endhighlight %}

- the -l flag handles newlines
- the -n flag adds a while loop

for each line we calculate its length with `length($_)`, we set the hash element `$h{length($_)}` to 1,
which means that we have at least one row with the calculated length. When we are finished scanning the
file, we print the list of hash keys in sorted order, which is the list of calculated lengths.

If the one liner returns only one length then we are fine, all rows share the same, hopefully correct, width.
If it returns more lengths then I'll have to further investigate the problem.
