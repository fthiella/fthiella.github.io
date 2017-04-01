---
layout: post
title:  "Installing and testing Poet and Mason, on Windows!"
date:   2015-08-02 12:00:00 +0200
categories: mason perl learningmason
---

Good news about Mason2 is that it can be easily installed using ○cpanm○:

○○○○bash
cpanm -S --notest Poet
○○○○

And good news is that nowadays is pretty easy to install it on the Windows platform as well.
While my production servers are always Linux, I just switched back from Mac to Windows for my
desktop machine (pretty weird, isn't it?) and I'm happy to know that I can use Mason natively
rather than starting up a virtual machine.

Just install a recent version of ActivePerl, launch the ○cpan○ command and it will automatically
download the ○gcc○ compiler and the ○dmake○ utilty (though you can use your own installation,
if you know what you're doing). Then you can launch the ○"install Poet"○ command at the ○cpan○ prompt, and the installation process should proceed smoothly (or somehow smoothly. Hint: sometimes ○"force install Poet"○ helps).

I'm also using Sublime Text editor, which has become my favourite editor, and with the Mason syntax it does a pretty good job.

Don't forget to install ○cpanminus○ (with the provided ppm ... Perl Package Manager) it will be very handy at a later time. Now we can test our environment:

○○○○bash
C:\Users\fede\code>poet new test
test/.poet_root
test/bin/app.psgi
test/bin/get.pl
test/bin/run.pl
...
...

Now run 'test/bin/run.pl' to start your server.
○○○○bash

and we can run our server, in development mode:

○○○○bash
C:\Users\fede\code>test\bin\run.pl
Running plackup, -E, development, --port, 5000, -R, etc. etc.
HTTP::Server::PSGI: Accepting connections at http://0:5000/
○○○○

Okay, we can now connect to http://localhost:5000 and see the wecome page.
Later on these pages how to run our application in a production server.