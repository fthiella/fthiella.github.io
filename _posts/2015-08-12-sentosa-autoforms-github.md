---
layout: post
title:  "Sentosa Autoforms on GitHub"
date:   2015-08-12 12:00:00 +0200
categories: mason perl learningmason
---
I finally managed to open a repository on GitHub:


[https://github.com/fthiella/Sentosa/](https://github.com/fthiella/Sentosa/)


so I don't have to document here on this blog all the small changes I'm making to the code,
but I will still document the bigger changes, and there are going to be a lot!


Btw, I added some code for groups and apps management:

- Users belongs to one or more groups
- Apps belongs to one or more groups
- Apps are visible only to users that belongs to the groups associated to the app
- An administrator can see everything

**Sentosa AutoForms**

Sentosa AutoForms will be published as http://localhost:5000 during the development process (or anywhere else like http://local-intranet/sentosa)
and it will provide access to one or more *"web applications"*.


A *"web application"* is composed by forms, subforms, and datatables liked together, and has one or more groups associated to it.
Users belonging to those groups have access to the app (at a later time we can specify more detailed permission,
e.g. rear-write or read-only for every single object, or we can also specify filters based on the group or on the user,
but we'll see that later).


The sample app *"gardens"* will be accessible as http://localhost:5000/gardens (or http://local-intranet/sentosa/gardens).


A form of the app will be accessible as http://localhost:5000/gardens/flowers (it will be redirected somehow to http://localhost:5000/form?id=234).
A datatable of the app will be accessible as http://localhost:5000/gardens/ (it will be redirected somehow to http://localhost:5000/table?id=234
and the JSON driving the table will be http://localhost:5000/table_json?id=234).

**Bootstrap Admin Template**

Here on my machine I'm using a Bootstrap Admin Template that I bought from [Themeforest](http://themeforest.net/),
it's only a few dollars but it feels and looks great. Unfortunately, since it's not free, I cannot include it on my public project.


I'm really looking forward to implement some new code, about forms and about tables, I have something almost ready already!
But what I'm going to do now is to try some free Bootstrap Templates (maybe from startbootstrap.com?). Even if I'm focusing on coding in
*Perl/Mason*, working on a nice looking web app is more fun than working with a ugly interface.
