---
layout: post
title:  "Cut a fixed width text file in CMD"
date:   2018-08-07 12:48:00 +0200
categories: cmd
---
I recently had the need to cut a fixed width text file at a certain column, and I had to do it in the *Windows 10*
world, without any particular utility.

In linux it would be rather simple, thanks to the *cut* command:

````bash
$ cut -c -10 example.txt

````

On my windows machines there's always *Perl* available, and one alternative solution would be using one liner substitution:

````cmd
perl -pe s/(.{10}).*/$1/ example.txt
````

but on my colleagues machines there are no particular utilities installed, so I had to create the following script:

````cmd
@echo off
setlocal EnableDelayedExpansion
for /f "delims=" %%r in (example.txt) do (
	set s=%%r
	echo !s:~0,10!
)
````

it's still possible to use a single line command, without the need of a `.cmd` script, but firts we have to launch
the prompt with a special parameter `cmd /v` which indicates that [delayed expansion](https://ss64.com/nt/delayedexpansion.html) is enabled (it is disabled by default
to be compatible with MS-DOS 2.0 batch files!). After we launch `cmd /v` the command becomes:

````cmd
for /f "delims=" %r in (example.txt) do @(set s=%r && echo !s:~0,10!)
````

not really intuitive, is it? I think I am missing my teenager days when I used to program Assembly x86: that was complex,
but for good reasons. Cmd scripting is still complicated, but for no knowns reasons!
