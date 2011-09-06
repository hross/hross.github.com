---
layout: post
title: Quick and Dirty Drupal Profiling
categories: [drupal]
---

One major question I had when I first started debugging module code was "how can I see how my code is performing?". Turns out its pretty easy to get this info in a variety of ways.

### XHProf

The talk of the PHP profiling town seems to be [Facebook's latest entry XHProf](https://github.com/facebook/xhprof). For what I'm looking for it's a bit heavy, and it only seems to work on Linux, which is a problem, since I'm developing and deploying on a mac and a wintel box.

### Xdebug

Ah yes, my old friend [XDebug](http://www.xdebug.org). As it turns out, you can set up function profiling in XDebug with a couple of php.ini options. I won't spend too much time on setup, since Zend did an exhaustive job of explaining it [here](http://devzone.zend.com/article/2899). Or you can probably just take a look at the great [documentation here](http://www.xdebug.org/docs/profiler).

### Webgrind

Okay, so you created yourself a bunch of cachegrind files and you're ready to see what's happening. You now have a couple of options, depending on your platform ([WinCacheGrind](http://sourceforge.net/projects/wincachegrind/) on Windows, KCacheGrind on [Linux](http://kcachegrind.sourceforge.net/html/Home.html) or [MacCallGrind](http://www.maccallgrind.com/) on a Mac). However... the Mac version isn't free and setting up Linux compatibility stuff to view a call stack isn't my thing. Turns out there is a truly righteous web based grind file viewer called [webgrind](http://code.google.com/p/webgrind/). Cross platform and super easy install. Sweet. Only problem you may have is that large cachegrid files will take forever to load.

### Devel Module

So what about Drupal specific profiling information? Turns out there is a great module for that called [Devel](http://drupal.org/project/devel). It will give you SQL statement profiling and memory usage, and a variety of other features.

Finally, on the OS side you have your system specific tools like Process Explorer, Activity Monitor, top, etc.. Happy Profiling!