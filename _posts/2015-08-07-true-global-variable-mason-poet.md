---
layout: post
title:  "True Global Variable for Mason in Poet"
date:   2015-08-07 12:00:00 +0200
categories: mason perl learningmason
---
A few days ago, while I was roaming around Singapore, I offered a bounty (twice)
on an already existing question on StackOverflow [Global Variable mason2 in Poet](http://stackoverflow.com/questions/28281187/global-variable-mason2-in-poet)
 where the poster was asking how to use global variables in Mason. And I got a very nice and useful answer :)

Here I am describing how to get a *true global and persistent variable*.
The content of a true global and persistent variable is preserved between requests.

Has your web server 10 different threads that answer HTTP requests?
Then you'll end up having 10 different variables.
Has your web server 1000 threads? Well... you got the idea!

But what's the proper use of such global variables? Any thread can potentially serve any request,
so we cannot predict which server is going to answer which request.
Private informations doesn't have to go here.

But most web applications are driven by some datatabase, and connecting to a database has cost in terms of performance.
Since the database won't (usually) change between requests why do we have to establish the same database connection over and over again?

Actually, we don't have to: we can define a global variable `$dbh` whose value is a valid
database handler, which will be preserved between requests. This database handler value
will born with the thread and will die with the thread!

Oh but we still have to be careful since the handler could have been valid at the time
of the creation of the thread, but then something bad could have happened (e.g. a timeout could have occoured,
or some evil DBA might had killed your connection, just for the sake of it). Somehow we'll have to manage this.

Okay now the fun part. Please read [Poet::Import](https://metacpan.org/pod/Poet::Import)
and the [awarded answer](http://stackoverflow.com/a/31159329/833073) which I'm going to use as a reference,
and then let's do some coding!

**Global Variable Example**

````bash
# generate app Myapp
poet new Myapp
cd Myapp
````

add a class `Myapp::Import`, here's where we have to define our global variables:

````bash
vi lib/Myapp/Import.pm
````

and then let's add some code:

````perl
package Myapp::Import;
use Poet::Moose;
extends 'Poet::Import';

# create some variable
has 'mytemp' => (is => 'ro', default => 'my temp value');

method provide_var_mytemp ($caller) {
    return $self->mytemp;
}
1;
````

then we can test our global variable in our components, let's print its value on `comps/index.mc`:

````perl
<%class>
use Poet qw($mytemp);
</%class>
I got this <% $mytemp %> variable.
````

Let's see how to store a database handler, on my next post!
