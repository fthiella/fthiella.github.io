---
layout: post
title:  "Adminer for Oracle"
date:   2017-07-31 13:12:00 +0200
categories: sql
---
I'm becoming a fan of [Adminer](http://www.adminer.org), a database management tool written in PHP, with lots of features and consisting
only of a single PHP file, and very easy to deploy to an Apache web server. I have started to develop a plugin
[adminer-plugin-dump-markdown](https://github.com/fthiella/adminer-plugin-dump-markdown) so I can quickly convert queries and tables to
Markdown format, ready to be included in my e-mails or in my blog posts.

I wanted to connect to an Oracle database, but I got the following message error:

>No extension
>
>None of the supported PHP extensions (OCI8, PDO_OCI) are available.

I quicly checked my server and the `oci8.so` library wasn't available:

````bash
find / -name oci8.so 2>&1 | grep -v "Permission denied"
````

so here's how I proceeded:

1. download oracle instant client RPMs (Basic + Devel) from the Oracle website, and installation using `yum install instant-client-etc.ect.rpm`;
2. make sure that `pecl` tool is installed (if not `yum install php-pear`). Pecl can manage PHP extensions from a public repository;
3. install the oci8 extension with `pecl install oci8` (for PHP7, for older PHP need to specify oci8 version);
4. add the `extension=oci8.so` row to the `php.ini` configuration file on the `[PHP]` section.

Once the webserver is restarted, I can access an Oracle instance. I can use a service string like this:

````

(DESCRIPTION = (ADDRESS_LIST = (ADDRESS = (PROTOCOL = TCP)(HOST = host)(PORT = 1521))) (CONNECT_DATA = (SERVICE_NAME = service)))

````

without having to set the connection on the `tnsnames.ora` file. Adminer works great also on Oracle databases!