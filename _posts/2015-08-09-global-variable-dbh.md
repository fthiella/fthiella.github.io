---
layout: post
title:  "Global variable $dbh"
date:   2015-08-09 12:00:00 +0200
categories: mason perl learningmason
---
The name of my project is "Sentosa AutoForms", and it is going to be driven by a SQL database.
I'm using SQLite but it can be changed at a later time to MySQL or Postgresql.

Let's create my new project:

````bash
poet new Sentosa

cd Sentosa
````

and let's create our SQLite database. The schema goes on the <code>db/schema.sql</code> file:

````sql
create table if not exists af_info (
      id integer primary key autoincrement,
      attribute string not null,
      value string not null
    );

insert into af_info (attribute, value) values
('name', 'Sentosa AutoForms'),
('version', '0.01');
````

and the actual database data goes to `data/sentosa.db`:

````bash
sqlite3 -batch data/sentosa.db < db/schema.sql
````

This is how I would access this DB with a standard Perl application (using the DBI module):

````perl
#!/usr/bin/perl

use strict;
use warnings;
use DBI;

my $dbh = DBI->connect(          
    'dbi:SQLite:dbname=../data/sentosa.db',
    '',                          
    '',                          
    { RaiseError => 1 },         
) or die $DBI::errstr;

my $sth = $dbh->prepare('SELECT value FROM af_info WHERE attribute="name"');
$sth->execute();

my $name = $sth->fetch();

print @$name;
print "\n";

$sth->finish();
$dbh->disconnect();
````

and this is how I am connecting to it in my web application:

````perl
package Sentosa::Import;
use Poet::Moose;
extends 'Poet::Import';

use DBI;
 
method provide_var_dbh ($caller) {
  return DBI->connect(          
    'dbi:SQLite:dbname=data/sentosa.db',
    '',                          
    '',                          
    { RaiseError => 1 },         
  ) or die $DBI::errstr;
}
1;
````

and this is how I'm using it on my component `index.mc`:

````perl
<%class>
use Poet qw($dbh);
</%class>
<% $.title %>
<%init>
my $sth = $dbh->prepare('SELECT value FROM af_info WHERE attribute="name"');
$sth->execute();

my $name = $sth->fetch();

$.title("Welcome to @$name");
</%init>
````
