---
layout: post
title: Quick and Dirty Drupal Debugging
categories: [drupal]
---

Over the past few months I've had the opportunity to branch out and  hack on some Drupal modules. In doing so, I noticed that (a) I had no  idea what I was doing, and (b) there wasn't a simple, easy description  of how to debug Drupal code. Hopefully this will serve as a quick and dirty primer.

### Watchdog

This is the quick and dirty way to "write to console" in Drupal. Simply putting the following PHP statement in a module:

	watchdog("foobar", "hello world");
	
Will print to the Drupal log (Admin | Recent Log Messages).

### XDebug
I highly recommend you immediately download and install [XDebug](http://www.xdebug.org/) for your PHP installation. Many *AMP stacks include it by default ([MAMP](http://www.mamp.info/en/index.html) and [XAMPP](http://www.apachefriends.org/en/xampp.html) among them). 

Along with a bunch of other stuff, it will "pretty print"  PHP error messages in HTML, so the next time you take your Drupal  instance down by forgetting a semicolon, you'll see a nicely formatted call stack, memory profile, etc.. Installing it is ridiculously easy,  since [they give you very specific instructions based on your configuration](http://www.xdebug.org/find-binary.php).

### var_dump and debug_backtrace

Here is where XDebug comes in really handy, since it auto-formats these functions when they are used. In a nutshell, [var_dump](http://php.net/manual/en/function.var-dump.php) will literally dump the contents of any variable directly to the current HTML page it executes on. If you add it to some module code, it  will crap its output directly to the screen.

Even more useful is the [debug\_backtrace](http://php.net/manual/en/function.debug-backtrace.php) function, which gives you an array that contains the entire stack trace at the time it was called, _and_ the values of the variables that were passed to all of the functions in  that stack trace (you need XDebug to really appreciate this function).

### set_error_handler

Another nifty trick is replacing the default PHP error handler with your own (using the function [set_error_handler](http://php.net/manual/en/function.set-error-handler.php)).  Typically I replace it with a custom handler that uses debug\_backtrace  to give me a call stack and/or use var\_dump to see what's going on. Here is an example:

	set_error_handler('debugErrorHandler');
	
	// problem code here          
	
	restore_error_handler();
	
	// error handler function
	//TODO: remove this after we figure out the problem
	function debugErrorHandler($errno, $errstr, $errfile, $errline) {
		var_dump(debug_backtrace());
		// call php error handler
		return false;
	}

### Warning: mysql_real_escape_string() expects parameter 1 to be string, array given in includes/database.mysql.inc on line 321

This error seems to end up all over the place and it makes a great "how to debug" story. A lot of times someone will write some ugly code,  forget about it, then push it to your development environment or provide  it via a module.

What do you do if you start seeing this (or a similar type of error)  appearing in your watchdog logs? How can you figure out which module is  causing the problem?

Simply put, you follow the [instructions here](http://drupal.org/node/766256), except you replace the function with this one:

	function db_escape_string($text) {
		global $active_db;
		if(is_array($text)) {
			var_dump(debug_backtrace())
		}
	
		return mysql_real_escape_string($text, $active_db);
	}

Alright, well that's about it. Happy debugging!