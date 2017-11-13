---
layout: post
title:  "Postgresql: detect record changes"
date:   2017-11-13 12:35:00 +0200
categories: sql postgresql
---
In MySQL we can have a timestamp column which is automatically set to the current date/time whenever the record is inserted or updated:

````sql
create table mytable (
  id int primary key,
  value varchar(100),
  updated_timestamp timestamp not null default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP
)
````

In PostgreSQL we can have a timestamp column which is automatically set to the current date/time whenever the record is **inserted**:

````sql
create table mytable (
  id int primary key,
  value varchar(100),
  updated_timestamp timestamp not null default CURRENT_TIMESTAMP
)
````

but to have it updated whenever there's an update in the row we have to write a trigger. First we have to write a function that changes
the `updated_timestamp` column in a table:

````sql
CREATE OR REPLACE FUNCTION set_update_timestamp()
RETURNS TRIGGER AS $$
BEGIN
   NEW.updated_timestamp = now(); 
   RETURN NEW;
END;
$$ language 'plpgsql';
````

then we create the trigger:

````sql
CREATE TRIGGER mytable_update
BEFORE UPDATE ON mytable
FOR EACH ROW
  EXECUTE PROCEDURE set_update_timestamp();
````

there's still a little difference between MySQL and PostgreSQL, as MySQL will execute the update only whenever there's an actual update:

```sql
insert into mytable (id, value) values (1, 'Hello'); -- updated_timestamp set to current timestamp
update mytable set value='Hello' where id=1;         -- MySQL won't update current_timestamp, PostgreSQL will
update mytable set value='Hi!' where id=1;           -- both MySQL and PostgreSQL will update the record
````

but we can add more flexibility adding some conditions to the trigger:

````sql
CREATE TRIGGER mytable_update
BEFORE UPDATE ON mytable
FOR EACH ROW
  WHEN (old.value)::text IS DISTINCT FROM (new.value)::text)
  EXECUTE PROCEDURE set_update_timestamp()
````

More tables can share the same `set_update_timestamp()` function if they share the same `updated_timestamp` column.