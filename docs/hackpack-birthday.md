---
title: "HackPack: Birthday"
categories:
  - HackPack
tags:
  - HackPack
---

Visiting https://online-birthday-party.cha.hackpack.club/account.php and signing up for a new account takes us to a page where we can see what other users share our birthday.

If we catch the request, we can change our birthday to something like *'or'1'='1* in order to dump the database. Now, when we visit the https://online-birthday-party.cha.hackpack.club/profile.php? page, we see a list of users and their birthdays. 

This appears to be a Windows MySQL Database as --+ works to comment out the rest of a query. In order to dump all passwords, we can use the following string as our birthday:

1' union select username, password FROM users;--+

This selects the username and password data from the users table, ends the query with ;, then comments out the rest of the query with --+

The first result is:

***flag{c0mpl1cat3d_2nd_0rd3r_sql}***




