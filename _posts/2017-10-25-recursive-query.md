---
layout: post
title:  "Appointments Table and Recursive Query in PostgreSQL"
date:   2017-10-25 14:35:00 +0200
categories: sql postgresql
---
My company uses a booking table like this simplified one:

id | next_id | status | various_info       | booking_date | appointment_date | transfer_date | cancel_date
---|---------|--------|--------------------|--------------|------------------|---------------|------------
1  |         | B      | Paul from New York | 2017-10-01   | 2017-10-05       |               |
2  | 3       | T      | Lisa from London   | 2017-10-02   | 2017-10-05       | 2017-10-04    |
3  |         | B      | Lisa from London   | 2017-10-04   | 2017-11-03       |               |
4  |         | C      | Tom from Glasgow   | 2017-10-07   | 2017-11-04       |               | 2017-10-25

as you can see, anytime a user books an appointment, a row is added to this table where we store various user info, the booking date which is the current_date, and the date of the appointment. When the booking is active, the status of the row is `B = Booked`.

Users can ask to transfer an existing booking to a new date, so we just create a new row with the status `B = Booked` and we set the old appointment to `T = Transferred` setting also the `transfer_date` to the current date and the `next_id`field to the newly created appointment.

This makes things easy whenever we want to find all active bookings:

{% highlight sql %}
select various_info, appointment_date
from   appointments
where  status = 'B'
;
{% endhighlight %}

but to find when a booking was booked for the first time we need a recursive query. We start with this:

{% highlight sql %}
WITH RECURSIVE recursive_bookings AS (
  /* non recursive/root part: get all active bookings */
select
  b.id,
  b.id AS last_id,
  1 AS level
from
  bookings b
where
  b.status = 'B'

union all
  /* recursive part: go back to the previous transferred bookings */
select
  a.id,
  r.last_id,
  r.level+1
from
  recursive_bookings r join bookings a on a.next_id = r.id and a.status='T'
)
select * from recursive_bookings
;
{% endhighlight %}

this query on the dataset above will return the following rows:

id | last_id | level
---|---------|------
1  | 1       | 1
3  | 3       | 1
2  | 3       | 2

as you can see, for the booking with last_id=3 we have multiple rows:
- id=3 and level 1 which is the last and active one
- id=2 and level 2 which is the first time the user booked the appointment
and there might be many others in case the same booking is transferred multiple times.

If we want to get the first time an appointment was booked we have to only get the row per each last_id

{% highlight sql %}
select s.id as first_id, s.last_id
from (
  select id, last_id, level, max(level) OVER (PARTITION BY last_id) AS maxlevel FROM recursive_bookings
) s
where
  (s.level = s.maxlevel)
;
{% endhighlight %}

then we can play around this query and return other columns we might be interested in.

I have also one additional column which stores the reason why the appointment was transferred, and it's stored on the row which is transferred. The reason can be either C = the company had to transfer the appointment, because the slot
was no longer available or U = the user wanted to transfer the appointment. If I want to ignore the transfers caused by the user, I'll have to add this condition to the join:

{% highlight sql %}
recursive_bookings r join bookings a on a.next_id = r.id and a.status='T' and a.reason='C'
{% endhighlight %}

this will ignore all transfers caused by the user (and all transfers caused by the company but before user intervention, which might be desiderable or might not, but this depends on what the requirements are).
