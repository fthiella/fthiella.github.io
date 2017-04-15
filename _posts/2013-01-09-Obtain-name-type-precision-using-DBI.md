---
layout: post
title:  "Obtain name type and precision of all columns in a table using DBI"
date:   2013-01-09 19:53:00 +0200
categories: perl
---

I had to obtain the structure (name, type, and precision for every column) of a table, connected via JDBC using DBI, and this is how I did it:

````perl
#!/usr/bin/perl

use strict;
use warnings;
use DBI;
 
my $host = '127.0.0.1';
my $url = 'jdbc:Cache://127.0.0.1/SOURCE';

my $dbo = DBI->connect("dbi:JDBC:hostname=$host;port=9001;url=$url", 'username', '***')
or die $DBI::errstr;
 
my $qry = $dbo->prepare("select * from custom.table");
 
$qry->execute();

print "Structure of $table \n\n";
 
my $num_fields = $qry->{NUM_OF_FIELDS};
 
for (my $i=0; $i< $num_fields; $i++) { 
  my $field = $qry->{NAME}->[$i];
  my $type = $qry->{TYPE}->[$i];
  my $precision = $qry->{PRECISION}->[$i];
  print "$field, $type, $precision\n";
}
 
$qry->finish();
$dbo->disconnect();
````
