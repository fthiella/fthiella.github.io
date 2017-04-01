---
layout: post
title:  "User authentication"
date:   2015-08-11 12:00:00 +0200
categories: mason perl learningmason
---

Okay I know already that the code I'm going to post here is quite a big mess: it will work under
some circumstances but it's buggy and insecure. I will update this post as soon as I figure out
how to make this code better (expecially the `UserSession` part) :)

**Users tables**

Users go in af_users table, the password is stored as clear text and even if that's bad practice it's easy to fix and it's not the issue here ;) there are bigger problems.

````sql
CREATE TABLE `af_users` (
`id` integer primary key autoincrement,
`username` varchar(200) NOT NULL,
`password` varchar(200) NOT NULL
);
````

Then I'm creating a `Sentosa::Users` package that contains functions to manage users.
At the moment it's like this:

````perl
package Sentosa::Users;
use Poet::Moose;
extends 'Poet::Import';

use Poet qw($dbh);

sub auth_user {
 my ($u, $p) = @_;
 my $ar = $dbh->selectall_arrayref(
 "SELECT id, username, password
 FROM af_users
 WHERE username=? AND password=?",
 {Slice => {}},
 $u,
 $p
 );

 if ($ar->[0]->{username} eq $u) {
 return $ar->[0]->{id};
 }
 return undef;
}

1;
````

then I will add change password, groups managements, or other useful funcions.

**Sessions table**

Here's where I should do some improvements otherwise it's easy to hack cookies and get a working session without the need of getting authenticated.

Sessions goes in the `af_sessions` table:

````sql
CREATE TABLE `af_sessions` (
 `id` INTEGER PRIMARY KEY AUTOINCREMENT,
 `auth_user_id` INTEGER,
 `auth_ts` timestamp DEFAULT CURRENT_TIMESTAMP
);
````

and the `Sentosa::UserSessions` package is like this:

````perl
package Sentosa::UserSessions;
use Poet::Moose;
extends 'Poet::Import';

use Poet qw($dbh);

sub session_is_valid {
 my ( $sid ) = @_;

if (defined $sid) {
 my $ar = $dbh->selectall_arrayref("SELECT id, auth_user_id FROM af_sessions WHERE id=? AND auth_ts>=datetime('now', '-30 minute')", {Slice => {}}, $sid);

if ($ar->[0]->{id} eq $sid) {
 # TO DO: Big Warning - this is UNSAFE - need to perform some extra checks!
 return $ar->[0]->{auth_user_id};
 }
 }
 return undef;
}

sub add_session {
 my ($nid) = @_;
 my $sth = $dbh->prepare("INSERT INTO af_sessions (auth_user_id) VALUES (?)");
 $sth->execute($nid);
 return $dbh->last_insert_id("", "", "af_sessions", "id");
}

1;
````

**Authentication: Base.mp**

On `base.mp` I'm checking if username and passwords are provided. If they are I will call `auth_user`
to see if they are correct, and if they are I will create a new session.

Then I will check the provided session (if any) to see if it's still valid (here's some extra work
is required) and if it is not expired. Here's the final code, but I know already that I should move
some parts outside in a proper package:


````perl
use Sentosa::Users;
use Sentosa::UserSessions;

has 'username';
has 'password';
has 'authenticated_user';

has 'title';

method wrap() {
if ($.username || $.password) {
$.authenticated_user( Sentosa::Users::auth_user( $.username, $.password ) );
if (defined $.authenticated_user) {
$m->session->{auth_id} = Sentosa::UserSessions::add_session($.authenticated_user);
} else {
$.authenticated_user(undef);
$m->session->{auth_id} = undef;
}
}

$.authenticated_user(Sentosa::UserSessions::session_is_valid($m->session->{auth_id}));

inner();
}
````

Okay I'm not too proud of this code, I know it's not high quality, but at least I could move on and see my application working.
I'll come back to this code at a later time.
