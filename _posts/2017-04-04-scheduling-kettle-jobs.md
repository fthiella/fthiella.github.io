---
layout: post
title:  "Scheduling Kettle Jobs"
date:   2017-04-14 20:35:00 +0200
categories: pentaho kettle
---
Pentaho Data Integration is an open source tool that provides Extraction, Transformation, and Loading (ETL) capabilities.
While it's an essential DWH tool, I use it quite a lot also as an **integration** tool, where it performs really well.

![Pentaho Kettle]({{ site.baseurl }}/images/pentaho_kettle_screenshot.jpeg)

We use it to populate a table with the current load of our emergency rooms, then a Perl application will publish the
contents of the table in JSON format, and finally a Web Application and an Android Application read the content of the JSON
and shows the data to the users formatted in a nice way:

![First Aid]({{ site.baseurl }}/images/first-aid.jpeg)

We also use it to transfer historical data from legacy applications to the new ones. To distribute patients data between
different softwares of different vendors. To do quality checks on our tables, and generate automatically emails with alarms.
Or to fetch *XML* or *Excel* files we get from web services, and integrate the contents into our live systems. To prepare
monthly *TXT* or *Excel* or *XML* extractions.

All those tasks are managed by some *jobs* that calls multiple *transformations*, but I have to schedule those jobs somehow.
But how? This is what I am using, and I find it pretty *elegant*.

## Windows Scheduler

I wanted to use the standard Windows Scheduler, not the Pentaho scheduler. For Linux machines the procedure is very similar.
First of all I have created a `StartJob.cmd` batch file, that accepts only one parameter: the job to be executed.
Here's how it looks like:

````cmd
@echo off

set kitchen="D:\Pentaho\kettle\data-integration\kitchen.bat"

set repository="Repository ULSS4"
set repository_user=username
set repository_pass=password

set job_dir="/Updates"
set job_name=%1
set log="D:\Script\Spoon\log\"%2".log"

%kitchen% /rep:%repository% /user:%repository_user% /pass:%repository_pass% /dir:%job_dir% /job:%job_name% /level:Basic >> %log%
````

We can start a job very simply with the following command:

    C:\>StartJob.CMD "job_updates_15min"

which is more convenient than calling the `kitchen.bat` batch file directly (if you need more flexibility, just add a little more parameters).

I then scheduled a list of jobs to be executed at different intervals, e.g.

- job_updates_daily
- job_updates_1h
- job_updates_mondays
- job_updates_1th_month

this is kinda boring but it has to be configured only once. We won't touch the task scheduler anymore. Each of those jobs will contain
a list of sub-jobs that will be executed (e.g. `job_update_json`, `job_send_email`, ...). Whenever we want to schedule a job hourly
instead of daily, we'll have to remove the job from the daily job, and add it to the 1h job. No need to access the server where Kettle
is installed anymore.

## Log Table

We want to have a log of which tasks have been executed, when, how long they took, how many rows were updated, etc.

The table structure will be like created with this PostgreSQL query:

````sql
create table log_updates (
  id serial primary key,
  data_esecuzione varchar(20),
  flagtipotabella varchar(20),
  nometrasformazione varchar(40),
  dtiniziocaricamento timestamp,
  dtfinecaricamento timestamp,
  esitocaricamento int,
  read int,
  written int,
  updated int
  unique (data_esecuzione, flagtipotabella, nometrasformazione)
)
````

and it will look like:

id | data_esecuzione | flagtipotabella | nometrasformazione | dtiniziocaricamento | dtfinecaricamento   | esitocaricamento | read | written | updated
---|-----------------|-----------------|--------------------|---------------------|---------------------|------------------|------|---------|--------
1  | 2017-04-14      | DWH             | dim_doctors_update | 2017-04-14 15:00:01 | 2017-04-14 15:03:35 | 0                | 743  | 3       | 86       
2  | 2017-04-14      | Web Apps        | update_ps_json     | 2017-04-14 15:03:37 | 2017-04-14 15:05:54 | 2                |      |         |        
3  | 2017-04-14      | Web Apps        | update_patients    | 2017-04-14 15:05:59 |                     | 1                |      |         |        

here we can see that the task `DWH` / `dim_doctors_update` was executed successfully (esitocaricamento=0), the task `Web Apps` / `update_ps_json`
ended up with an error (esitocaricamento=2), while the task `Web Apps` / `update_patients` didn't finish yet (maybe it's still running, maybe it hang...).

Whenever we start a new job, we will generate a single row with esitocaricamento=1:

data_esecuzione | flagtipotabella | nometrasformazione | esitocaricamento | dtiniziocaricamento
----------------|-----------------|--------------------|------------------|--------------------
2017-04-14      | DWH             | dim_doctors_update | 1                | 2017-04-14 15:00:01

and we will add it to the logs table with an *Insert/Update* step using the lookup keys (`data_esecuzione`, `flagtipotabella`, `nometrasformazione`).

Then, whenever the task end succesfully, we will generate a row with the same lookup keys `('2017-04-14', 'DWH', 'dim_doctors_update')` but
`esitocaricamento=0` and the current_timestamp as the end datetime, and eventually the number of rows read, written and updated. We are still using
an *Insert/Update* task that will change the status from 1 to 0, will leave the starting datetime untouched, and will add the end datetime and the number
of rows read, written and updated.

Whenever the task end unsuccesfully, we are doing the same but we will update the status from 1 to 0.

If the tast is still running, the status will be 1. Even if the task won't finish the status will still be 1, so we know that there might be a problem somewhere.

## The Scheduled Jobs

Every scheduled job (daily, 1h, 15min, etc.) is structured this way:

- first we define the starting point for job execution
- then we execute a transformation that sets in a variable the current date (for logging reasons)
- finally, we insert in sequence all the jobs we actually want to execute (e.g. `job_update_json`, `job_send_email`, ...)

this is how it looks like:

![Daily Job]({{ site.baseurl }}/images/job_updates_daily.jpeg)

make sure that connections are **black** (unconditional), so even if a job fails the following jobs
will be executed anyway (if some jobs are related one to each other and you want a **conditional** connection, we will use a sub-job).

## Setting the current date in a variable

This task is for logging pourposes. We are going to run a query that returns the current date:

    select current_date as currentdate from dual

and save in the `data_caricamento` variable. Here's how it will look like:

![Set Load Date]({{ site.baseurl }}/images/set-current-date.jpeg)

## How a standard Job will look like

A standard job will performs the following tasks:

- first we define a starting point
- then we will call the transformation `general_log_start` that will put one row in a log table, so we know that the job has started
- then we call one or more transformations (or sub-jobs) that will perform the taks we need, eg. import some data, update a table, etc.
- when everything is okay (green connections) we will call the `general_log_end` transformation called `general_log_end OK`
- when anything goes wrong (red connections) we will call the `general_log_end` transformation called `general_log_end KO`

and will look like this:

![Standard Job]({{ site.baseurl }}/images/standard-job.jpeg)

for logging reasons, we will define two parameters on each of our jobs:

- flagtipotabella eg. `DWH`, `Web App`, `Other`
- nometrasformazione eg. `job_update_patients`, `job_send_email`

## General Log Start

This task will insert a new row on the logs table with the following informations:

- `data_caricamento` which is the current date time of the shceduled task
- `flagtipotabella`, taken from the parameters of the outer job
- `nometrasformazione`, taken from the parameters of the current job
- the current date time, which will be the start date/time
- `esitocaricamento=1`, the task has been started

![General Log Start]({{ site.baseurl }}/images/general_log_start.jpeg)

Here's how the *generate rows* step will look like:

![Generate Rows]({{ site.baseurl }}/images/general_log_start_generate_rows.jpeg)

Here's a JavaScript step that will get the parameters from the outer job:

````javascript
var dtcaricamento = getVariable("data_caricamento","");
var nometrasformazione = getVariable("nome_trasformazione","no");
var flagtipotabella = getVariable("flag_tipotabella","no");
var dtiniziocaricamento = new Date();
var dtfinecaricamento;
var esitocaricamento = 1
````

and here's how the *Insert/Update* step will look like:

![Insert/Update]({{ site.baseurl }}/images/general_log_start_insert_update.jpeg)

## General Log End

The general log end is very similar to the general log start. There's only one general log end,
but we will pass the parameter `esitocaricamento=0` whenever the execution is OK, and `esitocaricamento=2`
whenever there's an error (right click -> Edit job entry)

![Insert/Update]({{ site.baseurl }}/images/esito_caricamento_ok.jpeg)

only the JavaScript code is different:

````javascript
var dtcaricamento = getVariable("data_caricamento","");
var prog = getVariable("prog","");
var nometrasformazione = getVariable("nome_trasformazione","");
var flagtipotabella = getVariable("flag_tipotabella","");
var esitocaricamento = getVariable("esito_caricamento","");


var read = getVariable("LINES_READ","") | "";
var written = getVariable("LINES_WRITTEN","") | "";
var updated = getVariable("LINES_UPDATED","") | "";

setVariable("read", null, "p");
setVariable("written", null, "p");
setVariable("updated", null, "p");

var dtfinecaricamento;

if (esitocaricamento == '0') {
  dtfinecaricamento = new Date();
} else {
  dtfinecaricamento = null;
}
````

## Transformation

The simplest transformation will read data from one table, perform some tasks on this data,
ad output some rows with a Table output or an Insert/Update task.

The interesting part here is to use an *Output Step Metrics* to get the number or rows *read*, *written*, *updated*
and make it available to the outer job where the *General Log End* transformation will save those values in the logs table:


![Insert/Update]({{ site.baseurl }}/images/actual_transformation.jpeg)
