---
layout: post
title: Debugging Drupal Cron
categories: [drupal]
---

Recently, we had some issues with Drupal cron and indexing of Solr results, so I figured I'd share a couple of quick tips on debugging Drupal cron:


* Get [Supercron](http://drupal.org/project/supercron). That will let you individually run each cron task so you can figure out which one is failing.
* Usually the problem is with search\_cron. In order to dig deeper into this, you can hack core with watchdog statements and run a few database queries to identify problematic nodes. [See this thread here for a quick how to](http://drupal.org/node/361171). Here is an example of a hacked up search_cron function for debugging purposes:

		{% highlight php %}
		function search_cron() {
			// We register a shutdown function to ensure that search_total is always up
			// to date.
			register_shutdown_function('search_update_totals');
			
			// Update word index
			foreach (module_list() as $module) {
				watchdog('search_debug', 'update index for ' . $module);
				module_invoke($module, 'update_index');
			}
		}
		{% endhighlight %}

* Finally, if things get really hairy, you can build your search cron task into a module and run it by itself on demand. Build your own search cron task and exclude it. [Details here](http:/drupal.org/node/635480).