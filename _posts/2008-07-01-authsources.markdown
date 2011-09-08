---
layout: post
title: Adventures In Search - Part 1 - What is Search?
categories: [webcenter]
---

Since [Fabien finally updated his blog][1] with a nice write up on the [published content redirect][2], and I told him I would try and beat him to the punch, I think I now owe a post or two. This one has been sitting unpublished for a few days, so here we go...

* * *

One of my favorite parts of portal work is the fact that most portal implementations touch a variety of technologies: different programming languages, a variety of internal systems and many different authorization and authentication mechanisms. One of the most common of these is LDAP, whether it be in the form of Active Directory or some other LDAP server.

Unfortunately, I have the same problem I'd like to think most techies have: if I don't work with something for 6 months or so, I tend to forget at least half of the important details. And since LDAP integration usually only happens every so often, mostly on new portal installs, I find I tend to forget the details only to have to re-learn them again.

Hopefully this post can serve as a reminder, and perhaps a primer for the uninitiated.

### User Synchronization

For every user that logs into the portal, the portal has a record of their account in its **PTUSERS** database table. No matter that they are in Active Directory, LDAP, what have you, the user still must be listed in this table in order to log into the portal (basically meaning they are a user object in the portal).

The **PTUSERS** table is updated periodically by jobs run on configured authentication sources which essentially go out to an LDAP directory (or custom directory), ask for any new user accounts or groups and set them up in the portal.

One thing to note about this mechanism is that there is a time lag between when a user is created in a directory and when they can log into the portal. It would be nice if the authentication source were queried if the user was not found, and you can introduce customizations to do this, but OOTB you have to wait on the sync job.

Every authentication source in the portal also has an associated prefix, as well as a set of users and groups. The prefix is like (and can even be) a Windows domain. It is used to distinguish duplicate user names on different authentication sources.

### A Quick Reference

Unfortunately, it can be highly confusing when you're trying to figure out what all the different user properties in the portal are, how they relate to LDAP/AD configuration, and what they actually mean. To that end, I have come up with the following table that (hopefully) explains each mapping in enough detail that you can see how it is built and what it is meant to do.

<table>
	<tr>
	<td>PTUSERS Column</td>
	<td>AD Auth Source Value</td>
	<td>LDAP Auth Source Value</td>
	<td>User Profile Value</td>
	<td>Description</td>
	</tr>
	
	<tr>
	<td>NAME</td>
	<td>Auth Source Prefix + User Name Attribute</td>
	<td>Auth Source Prefix + User Name Attribute</td>
	<td>Display Name</td>
	<td>A &quot;throw away&quot; descriptive name for the user. Can be changed with a PWS or manually by the user.</td>
	</tr>
	
	<tr>
	<td>MAPPINGAUTHNAME</td>
	<td>User Name Attribute</td>
	<td>User Name Attribute</td>
	<td>none</td>
	<td>A base mapping name for the user (without auth source prefix)</td>
	</tr>
	
	<tr>
	<td>LOGINNAME</td>
	<td>Auth Source Prefix + User Name Attribute</td>
	<td>Auth Source Prefix + User Name Attribute</td>
	<td>Login Name</td>
	<td>The name a user has to type to log into the portal on the login screen (including the value that must be in the auth source drop down)</td>
	</tr>
	
	<tr>
	<td>AUTHUNIQUENAME</td>
	<td>objectGUID in AD (not specifiable)</td>
	<td>User Unique Name Attribute (defaults to DN)</td>
	<td>Remote Unique Name</td>
	<td>A uniqueness constraint in the directory to tell the portal this is the same user, even if their login or name changes.</td>
	</tr>
	
	<tr>
	<td>AUTHUSERNAME</td>
	<td>User Authentication Attribute (usually userPrincipalName to guarantee cross domain uniqueness - not sAMAccountName which is only unique in the domain)</td>
	<td>User Authentication Name Attribute (if not specified, defaults to DN - Distinguished Name)</td>
	<td>Remote Authentication Name</td>
	<td>The property used to authenticate the user with the directory. May not be used for authentication if an SSO solution is in place.</td>
	</tr>
</table> 

A couple of notes:

1.  The help files on the authentication source configuration files are surprisingly helpful. 
2.  If you need to figure out your LDAP/AD structure, see user properties or run queries, I highly recommend this free tool: [Softerra LDAP Browser][3]. 

See you next time.

 [1]: http://fsanglier.blogspot.com/
 [2]: http://fsanglier.blogspot.com/2008/06/alui-publisher-part-2-increase_30.html
 [3]: http://www.softerra.com/download.htm  