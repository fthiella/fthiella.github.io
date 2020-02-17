---
layout: post
title:  "Systemctl start sentosa.service"
date:   2016-02-26 12:00:00 +0200
categories: mason perl learningmason
---

I just started to publish my Sentosa web portal and users of my company are now starting to use it. Great!


I wanted to add a service on my CentOS server and I wanted to manage it with Systemd.


First I created a `/etc/systemd/system/sentosa.service` file that contains:

{% highlight bash %}
[Unit]
Description=Sentosa Autoforms
After=network.target

[Service]
ExecStart=/usr/local/bin/plackup -E production --port 5001 --access-log /var/www/apps/sentosa/logs/access.log /var/www/apps/sentosa/bin/app.psgi
{% endhighlight %}

the app.psgi is the standard file from GitHub, I just disabled the debug mode.


Now I can start, restart, stop my service with:

{% highlight bash %}
systemctl start sentosa.service
systemctl restart sentosa.service
systemctl stop sentosa.service
{% endhighlight %}

And I can monitor the tail of the logs with:

{% highlight bash %}
journalctl -u sentosa.service -f
{% endhighlight %}
