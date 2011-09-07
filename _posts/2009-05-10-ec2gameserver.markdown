---
layout: post
title: Running a Game Server on Amazon EC2
categories: [ec2]
---

Yes, it's true that I haven't posted in quite a while. My bad. Hopefully you enjoy this little tidbit, even though it's my first non-ALUI post on this blog…

### Game Night

Recently, after a long workday some co-workers and friends of mine started discussing a “game night”. All of us have jobs, lives outside of work, and are no longer college students, but all of us remember the glory days of [Counterstrike][1], [Quake][2] and the like.

Of course, none of us has anything more than a decently performing laptop, and all of us have an aversion to spending money. And so it was that we happened upon a game called [F.E.A.R. Combat][3]. Perhaps its a bit long in the tooth, and perhaps it is behind the times, but it sure is fun, and it sure is **FREE**.

The point of that longwinded story is that every other Wednesday has become game night, or more specifically, **F.E.A.R.** night. And since we are all computer geeks, and we all work in the web technology world in some way or another, [someone][4] brought up the idea of running a **F.E.A.R.** instance on Amazon's EC2.

Recently, I had a bit of time on my hands and an urge to try it out, and thus this post was born…

### Starting and Connecting to an EC2 Instance

First, I signed up for [Amazon EC2][5] (actually, I had already signed up when I wrote [this blog post][6]). The invaluable [Getting Started Guide][7] contained all the basics I needed to start instances, make images, sign up, etc..

Next, I made sure to download the [ElasticFox plugin for Firefox][8]. This makes managing and running EC2 instances much easier. If you want to get started quickly, here is a great [Getting Started Guide][9] for the plugin.

After installing and setting up ElasticFox, I was ready to start up a base image. I chose to run an Ubuntu image, since package management and documentation is readily available. [This site][10] has a few base AMI's which I used to get started. I simply searched for the AMI ID I wanted and followed the Elasticfox instructions on starting an instance.

One thing I had to keep in mind was that I wanted to allow the proper TCP/UDP access so that people could connect to my server. In this case, I allowed the following ports:

<table>
	<tr><td>Application</td><td>Protocol</td><td>Port</td></tr>
	
	<tr><td>SSH</td><td>TCP</td><td>22</td></tr>
	<tr><td>HTTP</td><td>TCP</td><td>80</td></tr>
	<tr><td>F.E.A.R.</td><td>TCP</td><td>27888</td></tr>
	<tr><td>TeamSpeak</td><td>UDP</td><td>8767</td></tr>
</table>

The other thing I did, in order to keep things simple for future connections, was associate a static IP with my running instance (these are called [Elastic IP's][11] in EC2 parlance). The procedure is mind-numbingly simple in ElasticFox, so I'll refer you to the [Getting Started Guide][9] if you need more information on how to do it.

At first, I had some issues actually connecting to my image using SSH (Elasticfox will auto launch an SSH client). The problem ended up being that I was using [Putty][12] for SSH and it does not recognize the private key format used by EC2. Doh. Fortunately, you can convert your keys using Puttygen. Amazon was nice enough to dedicate an appendix in their [Getting Started Guide][7] for [this exact problem][13].

Problem solved.

My next steps were the steps you'd take to install and configure any server so that it could host F.E.A.R. Combat, a TeamSpeak server (an in-game voice communication server), and a [Munin][14] monitoring instance (so I could get some stats to see how well EC2 performed in a real world scenario).

### Preparing for the Installation

After everything was running, I wanted to make sure I had the prerequisites to run F.E.A.R. and install any optional components. As it turned out, my Ubuntu instance was fairly locked down. In order to download/install what I needed, I had to [update my sources list to included the multiverse and universe repositories][15].

Once this was done, I updated the list of installable applications via:

    apt-get update

And installed some C++ compatibility libraries for the dedicated server via:

    apt-get install libstdc++5

At this point I was all set to install the base components of my server.

### Installing and Configuring F.E.A.R. Combat

My first step was to download the [F.E.A.R. dedicated linux server here][16].

Since I already had the prerequisites installed (see above), all I had to do was extract the archive to disk, modify the included *start.sh* to my liking (I used a custom configuration via the "optionsfile argument, used nohup to prevent it from shutting down accidentally, etc), and start the server.

### Installing and Configuring TeamSpeak

[TeamSpeak][17] is an in-game voice communication server. Since my game night buddies are mostly remote, I figured it would be nice to provide some voice communication for trash talk and strategy.

I logged in as root and ran:

    apt-get install teamspeak-server

A teamspeak user was added, the server started and I was ready to rock and roll. As for configuring the server… it seemed to work okay, so I didn't bother =). However, you can find some [instructions for configuration here][18].

### Installing and Configuring Munin

[Munin][14] is a monitoring tool that allows you to capture CPU, memory, process data, and all kinds of other stats in 5 minute increments. It can be used for monitoring many systems with many kinds of statistics, but that is outside of the scope of this post. For now let's just say I wanted a simple way to capture statistics for my AMI.

The installation also turned out to be very simple. It involved using apt-get to install apache and Munin. Rather than regale you with the details, I'll just point you to [this simple tutorial][19].

**Note:** I did have some issues getting Munin to work at first, but once I made sure my local node was listening on the loopback adapter only, it seemed to work. See *Section 1.3 (Configuring the Node)* of the [tutorial][19] for details.

### Creating and Registering an AMI

At this point I had everything I needed to run a game server. I tested client connections to my Teamspeak host, Apache server hosting Munin, and the F.E.A.R. server itself and everything worked great.

The only problem was that if I ever shut down the running instance, all of my work would be gone and I would have to re-install everything the next time I wanted to host a game. Thus, I needed to create an AMI from my base image.

The procedure for this was relatively simple, and well documented in the Getting Started Guide [here][20]. However, there are a few things you might want to know before you dive in:

*   You'll need at least a basic working knowledge of [Amazon S3][21], since you'll need it to store your finished AMI. I suggest grabbing [S3Fox][22] and using it to create an Amazon S3 bucket. This process is fairly simple, but still a minor annoyance. 
*   The base image I used did not have the [EC2 API tools][23] installed on it, which meant that I could not register my EC2 instance without installing them. I did this by running: 

    apt-get install ec2-api-tools

After that, all I needed to do was set my **JAVA_HOME** environment variable and follow the rest of the [Getting Started Guide][20].

### Final Thoughts

#### Security

As you probably noticed, the configuration on my AMI is hardly secure. I ran things as root, didn't bother changing passwords or restricting IP's, etc, etc.. I offer no excuses, save my own laziness.

However, the nice thing about an AMI is that it is only going to be used on game night for a few hours. I'm hardly worried about being hacked. Any time there is a problem, all I have to do is terminate the instance and boot up another AMI. Since nothing is persistent, and there are no credentials on the box, this is great. 

Imagine if I had set up a dedicated server for this. I'd have to worry about all kinds of hardening due to the longevity of the configuration. Yuck.

#### Going Further

Of course, as always, there are some things I could have done that would have taken this post further:

*   Capturing “real” usage stats and anecdotal performance data (is this a feasible, reliable, and cost effective solution?). This will probably follow in a future blog post (after the next “game night”). 
*   Writing a wrapper for the AMI so that it can be started and stopped on-demand via the web. Someone could definitely write a dedicated hosting web site if they could figure out all the possible licensing restrictions. 

Otherwise… that's about it. After reading this you should be in a position to create your own AMI's using EC2. The overall experience for me was rather pleasant, though there were some things I think Amazon could have done to simplify the process.

 [1]: http://en.wikipedia.org/wiki/Counter-Strike
 [2]: http://en.wikipedia.org/wiki/Quake
 [3]: http://projectorigin.warnerbros.com/fearcombat/main
 [4]: http://techopener.com/blog/
 [5]: http://aws.amazon.com/ec2/
 [6]: http://hross.net/blog/2008/04/integrating-amazon-s3-with-the.html
 [7]: http://docs.amazonwebservices.com/AWSEC2/2008-05-05/GettingStartedGuide/index.html
 [8]: http://developer.amazonwebservices.com/connect/entry.jspa?externalID=609
 [9]: http://developer.amazonwebservices.com/connect/entry.jspa?externalID=1797
 [10]: http://alestic.com/
 [11]: http://developer.amazonwebservices.com/connect/entry.jspa?externalID=1346
 [12]: http://www.chiark.greenend.org.uk/~sgtatham/putty/
 [13]: http://docs.amazonwebservices.com/AWSEC2/2008-05-05/GettingStartedGuide/index.html?putty.html
 [14]: http://munin.projects.linpro.no/
 [15]: https://help.ubuntu.com/community/Repositories/CommandLine
 [16]: http://fear.filefront.com/file/FEAR_v108_Dedicated_Linux_Server;71429
 [17]: http://www.teamspeak.com/
 [18]: http://forum.teamspeak.com/showthread.php?t=15438
 [19]: http://www.debuntu.org/book/export/html/134
 [20]: http://docs.amazonwebservices.com/AWSEC2/2008-05-05/GettingStartedGuide/index.html?creating-an-image.html
 [21]: http://aws.amazon.com/s3/
 [22]: http://www.s3fox.net/Default.aspx
 [23]: http://developer.amazonwebservices.com/connect/entry.jspa?externalID=351  