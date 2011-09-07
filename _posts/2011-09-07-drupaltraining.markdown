---
layout: post
title: Setting up a Drupal Training Instance
categories: [drupal]
---

In a couple of days I'm providing Drupal training at [PPC](http://ppc.com/). The goal of the training is to provide an overview and "get your feet wet" with Drupal.

I found a couple of cool things while looking for some decent training resources. First, the [handbooks on drupal.org](http://drupal.org/handbooks) are top notch (and free). Second, there is an excellent (free) ebook over at [learnbythedrop](http://learnbythedrop.com/) called [Building Your Blog With Drupal](http://learnbythedrop.com/buildingyourblog). It covers a soup to nuts install and configuration of a blogging platform (and is pretty up to date as of this writing). Perfect for a beginning training session.

To facilitate the training I set up an [nginx](http://nginx.net/) install with [php](http://www.php.net/) on my local windows machine (the training is in windows to make things less of a culture shock). You can find a great tutorial on doing that [here](http://eksith.wordpress.com/2008/12/08/nginx-php-on-windows/) and an nginx config tailored for Drupal [here](http://wiki.nginx.org/Drupal). Finally, I set up [mysql](http://www.mysql.com/) with [phpmyadmin](http://www.phpmyadmin.net/home_page/index.php) and I was ready to rock.

The best part of this is that I needed to set up 25+ training machines with the same configuration. Because it's all batch file oriented, I simply had to copy the entire directory tree, [take a dump of the database](http://www.clockwatchers.com/mysql_dump.html) and copy the whole thing to an [EC2](http://aws.amazon.com/ec2/) windows instance. Finally, with a little [nssm](https://iain.cx/src/nssm/) goodness I was able to make the whole thing automatic.

Presto. Drupal training.