---
layout: post
title: Charity Campaign (a node.js web app)
categories: [node.js, mongodb]
---

Over the past couple of months I've been putting together a [node.js](http://nodejs.org/) application for fun. The goal was to get some "real world" development experience with [node.js](http://nodejs.org/) and [mongodb](http://www.mongodb.org/).. figure out what the challenges are (is event oriented programming hard? how will I live life without RDBS JOINs?) and build something people might actually use.

To wit: [Charity Campaign](https://github.com/hross/Charity-Campaign)... an application for tracking charity drive donations. The basic concept is to have teams of users submitting various items, assign bonuses to those users based on what they submit, give some basic security/management and display stats.

The idea isn't new, [other people I know have written Google App Engine applications for it](http://techopener.com/), but its a lot easier to focus on a niche you know than make up a new one for the purposes of writing software.

Without further ado, here are some of the pieces/concepts I used to build the application:

### node.js packages

* [express](http://expressjs.com/) for most of the http framework and middleware support
* [connect-form](https://github.com/visionmedia/connect-form) for file uploads
* [ldapjs](http://ldapjs.org/) for ldap connectivity
* [mongodb](https://github.com/christkv/node-mongodb-native) for mongo db connectivity
* [async](https://github.com/caolan/async) for keeping my sanity when writing event driven code that had dependencies
* [express](http://expressjs.com/) for most of the http framework and middleware support

### concepts/info

The [vision media samples](https://github.com/visionmedia/express/tree/master/examples) were uber helpful in learning how to use express. [The mvc sample](https://github.com/visionmedia/express/tree/master/examples/mvc) is the basis for my routing framework.

[Stackoverflow](http://stackoverflow.com/) was useful for finding [configuration management information](http://stackoverflow.com/questions/5869216/how-to-store-node-js-deployment-settings-configuration-files).

Found a good [csv parsing](http://blog.james-carr.org/2010/07/07/parsing-csv-files-with-nodejs/) starting point (ended up hacking this code to bits, but the basics and the regex are there).

Twitter recently released [boostrap](http://twitter.github.com/bootstrap/) and it seemed cool so I converted the user interface to use it (previously I was using some free html template or other).

### false starts

[Geddy](http://geddyjs.org/) looked like a good MVC starting place, but development appears to be halted so I didn't get far.

[Mongoose](http://blog.learnboost.com/blog/mongoose/) looked cool, but seemed a bit heavy and constrictive. It probably would have helped with data validation and error handling, though.

I initially started with [couch-db](http://couchdb.apache.org/) and [cradle](https://github.com/cloudhead/cradle), but I didn't care about the replication or json consumption features and [generating unique integers for slugs started to become a PIA](http://stackoverflow.com/questions/5073343/approaches-to-generate-auto-incrementing-numeric-ids-in-couchdb).

The [mongodb tutorials](http://www.mongodb.org/display/DOCS/Tutorial) were much more accessible and [accommodating](http://www.mongodb.org/display/DOCS/Object+IDs).

### summary

Hopefully this application actually gets used at some point. Even if it doesn't, building and committing it to github has been a satisfying experience in and of itself.

