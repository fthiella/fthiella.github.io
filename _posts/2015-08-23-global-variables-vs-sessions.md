---
layout: post
title:  "Global variables between components? No! Sessions!"
date:   2015-08-23 12:00:00 +0200
categories: mason perl learningmason
---

I didn't post much recently because I wanted to focus on coding: even if it's still to early,
even if a lot of functions I want to implement are still missing, and even if the code is not
too robust, I want to start using the application within my company and I want to publish a
limited but working version on GitHub, so anyone can download it and see it in action.

**Global variables again**

My basic idea was to declare a `authenticated_user` variable in the `Base.mp` component,
then `Base.mp` will set it to the currently authenticated user if there's a valid session,
or to *Guest* otherwise.

> (Reminder: I should only move out the login/logout actions from Base.mp to the proper components, but at the moment is fine).

Since every component will implicitly extend Base.mp:

````perl
<%flags>
  extends => "Base.mp"
</%flags>
````

I can be sure that every component will have the correct authenticated user set. Or not? Something I didn't expect happens here:

- If not set, $.authenticated_user will be set to GET or POST data, e.g. http://localhost:5000/?authenticated_user=12345 (ouch!)</li>
- I can set a value to authenticated_user on Base.mp, so it will <strong>always</strong> override GET or POST data. But does this code smell? I'm still undecided.</li>
- Components called with `<& component.mc &>` won't have the correct $.authenticated_user set, so I have to call it with
  `<& component.mc, authenticated_user=>$.authenticated_user &>`
- But what if I call the component directly with http://localhost:5000/component?authenticated_user=12345 (ouch again!). No problem as Base.mp will override the GET or POST data.

At the moment I can still go on with this approach, it's starting to get less elegant and I need to clarify something
(which component extends which other? what's the logic here?) but if ever, I will fix it later.

>Temporary fixes have a funny way of becoming permanent because you never seem to have the time/inclination/memory to go back and fix them.
>
>If you're going to fix something, fix it the right way the first time.

Lol, yes, but I'm still a little unsure on what other alternatives do I have:

````perl
$m->notes("authenticated_user", "12345")
````

?

It seems to have the same behavior on called components, but it turns out that I just can use.... sessions!

````perl
$m->session->{auth_id} = $auth_id;
$m->req->{env}->{'psgix.session.options'}->{expires} = "+1h";
````

Some things I have to implement: the timeout, and making a session expire immediately. But I didn't have to reinvent the wheel like I was doing before hehe, plack sessions have ad ID stored on the server, and that's enough at the moment!