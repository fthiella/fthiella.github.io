---
layout: post
title:  "MySQL and Full Outer Join"
date:   2017-04-01 21:00:00 +0200
categories: sql mysql
---
In SQL, a **full outer join** is a join that returns all records from both tables,
wheter there's a match or not:

![Full Outer Join]({{ site.baseurl }}/images/full-outer-join.gif)

unfortunately MySQL doesn't support this kind of join, and it has to be emulated somehow. But how?

In SQL it happens often that the same result can be achieved in different ways, and which way is
the most appropriate is just a matter of taste (or of perormances).

But this time the question is a little controversial, even on [StackOverflow](http://stackoverflow.com/questions/4796872/full-outer-join-in-mysql)
not everyone agrees and the solution marked as correct isn't actually the correct one.

Suppose we have the following tables:

**Customers**:

company_id | name
-----------|------------
1          | Abc Company
2          | Noise Inc.
3          | Mr. Smith

**Partners**:

company_id | name
-----------|-----------
2          | Noise Inc.
4          | The Pages

a full outer join would be written as:

{% highlight sql %}
select
  c.name, p.name
from
  customers c full outer join partners p
  on c.company_id = p.company_id
{% endhighlight %}

and the expected result is:

name        | name
------------|-----------
Abc Company |
Noise Inc.  | Noise Inc.
Mr. Smith   |
            | The Pages

to get the same result we have to combine a left outer join query:

{% highlight sql %}
select c.name, p.name
from
  customers c left join partners p
  on c.company_id = p.company_id
{% endhighlight %}

with a right outer join query:

{% highlight sql %}
select c.name, p.name
from
  customers c right join partners p
  on c.company_id = p.company_id
{% endhighlight %}

(the right join is quite uncommon because it's more difficult to read,
and is equivalent to a left join with the order of the tables switched).

We could combine both queries with an `UNION ALL` clause, but this would
return some duplicates (all rows where the join succeeds will be returned
twice).

We could then use an `UNION` clause which will remove duplicates, but it
will fail if one of the table has no primary key or unique constraints, or
if the selected columns are not unique.

We could also use a different approach:

{% highlight sql %}
select c.name, t.name
from
  (select company_id from customers UNION
   select company_id from partners) n
  left join customers c on n.company_id = c.company_id
  left join partners p on n.company_id = p.company_id
{% endhighlight %}

which is often a good solution, but would fail if we allow the company_id to be NULL
in one or both tables (a full outer join will return those rows, while the previous one won't).

The most general solution is this:

{% highlight sql %}
select c.name, p.name
from
  customers c left join partners p
  on c.company_id = p.company_id

union all -- don't remove duplicates

select c.name, p.name
from
  customers c right join partners p
  on c.company_id = p.company_id
where
  c.company_id is null
{% endhighlight %}

duplicates, if already present on the source tables, won't be removed.
And the anti-join pattern on the second query assures that we are not introducing
new duplicated rows.