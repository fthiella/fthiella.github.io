---
layout: post
title:  "Postgresql: Detect Status Changes in a Table"
date:   2017-10-27 12:23:00 +0200
categories: sql postgresql
---
This is another challenging problem I had, and it took me some hours of work to find out how to solve it.
I was very tempted to use a script in a procedural language like Perl, which would make the problem easy and
the solution straightforward, but there had to be a pure SQL way and here's how I did it!

## The problem

Anytime a patient has to do some kind of (non urgent) surgery, we create a new event with all patient info on the main table 
`Event`, then we log all updates to this event on the `Changes` table.

- at first the patient is in it's **initial** state: he/she has to provide documents, papers, previous results, insurance, allergies, etc.
- then the **preparation** phase can begin: he/she has to do some tests, talk to some doctors, wait for the results;
- when everything the preparation phase is completed, he/she's **ready** and he has to wait for instructions;
- then the hospital will **give an appointment**, which can be updated (different room, different time) or changed (different day);
- finally he/she's got the surgery and the event is considered **completed**:

id | event_id |           status | change_date | other
---|----------|------------------|-------------|------
1  | 1        |          Initial |  2017-10-07 |   ...
2  | 1        |          Initial |  2017-10-08 |   ...
3  | 1        |      Preparation |  2017-10-09 |   ...
4  | 1        |      Preparation |  2017-10-10 |   ...
5  | 1        |            Ready |  2017-10-11 |   ...
6  | 1        | Appoinment-Given |  2017-10-12 |   ...
7  | 1        | Appoinment-Given |  2017-10-13 |   ...
8  | 1        | Appoinment-Given |  2017-10-14 |   ...
9  | 1        |        Completed |  2017-10-15 |   ...

There are a lot of useful informations than can be calculated here: how long does it take for a Ready event to be Completed? How long does it take
the initial or the preparation phase? And what about the whole process?

The table is not normalized: it is optimized for entering data, not for querying it, that's why the query isn't simple.

## Detect status changes

To detect status changes we can make use of the LAG window function:

    LAG(status, 1, status) over (PARTITION BY event_id ORDER BY id)

that will return, for every row, the status value of the previous (1) row.  If there's no such row in the window partitioned by the `event_id`,
it will return the current status. Then we can compare the previous value with the current value, and return 1 if a change is detected, and 0 otherwise:

````sql
with c1_detect_changes as (
select
  event_id,
  id,
  status,
  case when LAG(status, 1, status) OVER(PARTITION BY event_id ORDER BY id) = status then 0 else 1 end as n
from
  changes
order by
  event_id, id
)
select * from c1_detect_changes
````

and the result is:

event_id | id |           status | n
---------|----|------------------|--
       1 |  1 |          Initial | 0
       1 |  2 |          Initial | 0
       1 |  3 |      Preparation | 1
       1 |  4 |      Preparation | 0
       1 |  5 |            Ready | 1
       1 |  6 | Appoinment-Given | 1
       1 |  7 | Appoinment-Given | 0
       1 |  8 | Appoinment-Given | 0
       1 |  9 |             Done | 1

(it really doesn't matter if the first row is set to 0 -no status change- or 1 -status change detected-, but what's really important is that all other status changes are detected correctly with a 1).

## Create a group for all consecutive rows with the same status

We can calculate a running sum:

    sum(n) over (partition by event_id order by id) g

our query becomes:

````sql
with c2_running_sum as (
select
  event_id,
  id,
  status,
  sum(n) over (partition by event_id order by id) g
from c1_detect_changes
)
select * from c2_running_sum;
````

can you see it? Every row that shares the same consecutive status is now part of the same group:

event_id | id |           status | g
---------|----|------------------|--
       1 |  1 |          Initial | 0
       1 |  2 |          Initial | 0
       1 |  3 |      Preparation | 1
       1 |  4 |      Preparation | 1
       1 |  5 |            Ready | 2
       1 |  6 | Appoinment-Given | 3
       1 |  7 | Appoinment-Given | 3
       1 |  8 | Appoinment-Given | 3
       1 |  9 |             Done | 4

## Get the first status change for each group

Now we can get the first (minimum) id for every group (g):

````sql
whti c3_get_first_id_per_group as (
select event_id, g, status, min(id) as min_id
from c2_running_sum
group by event_id, g, status
)
select * from c3_get_first_id_per_group;
````

and here's the result:

 event_id | min_id |           status
----------|--------|-----------------
        1 |      9 |             Done
        1 |      6 | Appoinment-Given
        1 |      3 |      Preparation
        1 |      5 |            Ready
        1 |      1 |          Initial

## Back to Initial

We also have an additional requirement: sometimes it might happen that an event has to be sent back to the **Initial** status,
so I want to ignore all things that happened previously, what is done is done, and only consider the last Initial status:

````sql
with c4_get_last_initial as (
select
  event_id,
  max(min_id) max_initial
from
  c3_get_first_id_per_group
where
  status='Initial'
group by
  event_id
),
c5_latest_status_changes as (
select c3.event_id, min_id from c3_get_first_id_per_group as c3 inner join c4_get_last_initial as c4 on c3.event_id=c4.event_id where c3.min_id >= c4.max_initial
)
````

c4 will find the latest initial status, c5 will return all events after the last initial status.

## Getting all the rows

The latest query C5 will return the event_id and the min_id of the rows to be considered:

````sql
select changes.*
from changes
where (event_id, id) in (select * from c5_latest_status_changes)
````

and we can pivot the results with FILTER (if we have at least PostgreSQL 9.4)

````sql
select
  event_id,
  max(change_date) filter (where status='Initial') as initial,
  max(change_date) filter (where status='Preparation') as preparation,
  max(change_date) filter (where status='Ready') as ready,
  max(change_date) filter (where status='Appointment-Given') as app_given,
  max(change_date) filter (where status='Done') as done
from
  changes where (event_id, id) in (select * from c5)
group by
  event_id
````

A fiddle to play with some data is [here](http://sqlfiddle.com/#!17/7113d/1).