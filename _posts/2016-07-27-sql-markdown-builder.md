---
layout: post
title:  "SQL Markdown Builder"
date:   2016-07-27 12:00:00 +0200
categories: perl markdown sql
---
I like text editors, especially Sublime Text. And I like to work at the command line (on Linux, on Mac... but even on Windows 10 it has become very nice).


So it's pretty normal that everything I write is in Markdown syntax!


Since I work every day with SQL, and every day I have to prepare quick reports, extract some data, share some tables, share some rows... I needed a tool to run a SQL query against a database that returns a table in Markdown format!

This way I can easily copy & paste to a new mail, and format it nicely and professionally with [markdown-here](http://markdown-here.com/), quickly and without becoming crazy.

So I quickly wrote this tool and I called [SQL Markdown Builder](https://github.com/fthiella/Sql-mk-builder) because it can easily be integrated with Sublime Text, even if it is not a native plugin.

Yes I know this is a little off topic from Mason and Sentosa, but this tool is written in Perl and... I included it in Sentosa anyway! So let's start!

## Getting Started

Make sure you have installed `perl`, `File::Slurp`, `Getopt::Long`, `DBI`, and the `DBD` libraries for your database:

{% highlight bash %}
cpanm File::Slurp
cpanm Getopt::Long
cpanm DBI
cpanm DBD::SQLite
cpanm DBD::Pg
cpanm DBD::mysql
...
{% endhighlight %}

## Run a query

Once everything is installed, you can write a single query, or more queries separated by a `;` in a `.SQL` text file:

{% highlight sql %}
drop table if exists gardens;

create table gardens (
	id integer primary key,
	name varchar(100),
	city varchar(100)
);

insert into gardens (name, city) values
('Gardens By The Bay', 'Singapore'), ('Hyde Park', 'London'),
('Central Park', 'New York'), ('Villa Borghese', 'Rome'),
('Princes Street Gardens','Edinburgh');

select * from gardens;
{% endhighlight %}

and you can run your query file at the command line:

{% highlight bash %}
perl sqlbuild.pl -c dbi:SQLite:dbname=test.sqlite3 -s query.sql
{% endhighlight %}

the connection string is in the DBI format, this example is for SQLite so we don't need to specify a username or a password.

**SQL Markdown Builder** will execute every single query in sequence against the specified database. If the query is an INSERT or an UPDATE query, it will return '0 rows', otherwise it will return the results in Markdown format (nicely aligned):

````
0 rows
0 rows
0 rows

id | name                   | city
---|------------------------|----------
1  | Gardens By The Bay     | Singapore
2  | Hyde Park              | London
3  | Central Park           | New York
4  | Villa Borghese         | Rome
5  | Princes Street Gardens | Edinburgh
````

If some columns become too big, you can specify the maximum size of a column with `-mw` parameter (or use `-h` to see all parameters).

Instead of using the command line, you can also specify all connection strings, usernames, passwords inside the .SQL file itself:

{% highlight sql %}
/*
  conn="dbi:SQLite:dbname=test.sqlite3"
  username=""
  password=""
*/
{% endhighlight %}

This is very handy but not too secure (other users might peek inside your files, and also updating a password might become complicated).

## Integration with Sublime Text

This is for Windows, but Linux and OSX will be very similar.

Just get the provided file Sql-mk-build.sublime-build, update the working_dir:

````
{
	"cmd": ["perl", "sqlbuild.pl", "-s", "$file" ],
	"working_dir": "c:\\GitHub\\Sql-mk-builder\\"
    "selector": "*.sql"
}
````

and move it to the build directoy:

{% highlight cmd %}
C:\Users\YOURUSERNAMEHERE\AppData\Roaming\Sublime Text 3\Packages\User
{% endhighlight %}

then you can edit your .SQL files with Sublime Text, and see the results using CTRL+B.

Happy SQL & Markdown!