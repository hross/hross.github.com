---
layout: post
title: EC2/Cloudwatch Gaming Results
categories: [ec2]
---
As I mentioned in [my previous post][1], I wanted to capture some real world info on hosting a game server in the cloud. The results were a rousing success. We had 5 or 6 people connected at various times, played some Deathmatch and Capture the Flag, and everyone had a ping of 40 or less the entire time. I didn't notice any latency whatsoever and there were absolutely no packet loss or lag complaints throughout.

### Cost

I haven't broken down the numbers yet, but all told I started up an EC2 instance and hosted a game for 2 hours. I also attached an elastic IP for ease of use. That cost me less than $0.50. I'd say that's a pretty good deal.

### Usage

Below are the usage stats for network I/O and CPU usage. I gathered these using [my simple Java application][2] and created these no-frills charts in Microsoft Excel (all told, this took about 5 minutes to put together):

![network io][3] 
Figure 1 - Network I/O over a 2 hour F.E.A.R. game

![cpu usage][4]
Figure 2 - CPU Usage over a 2 hour F.E.A.R. game

### Conclusion

This is a short and imperfect analysis, but overall I'd say the “small” EC2 instance could easily have handled a 16 person game, both from a load and network traffic standpoint, and it would have cost me a dollar or so to host for 2 hours. That seems like great bang for your buck if you're looking to crank up a quick game and then move on to something else.

 [1]: http://hross.net/blog/2009/05/running-a-game-server-on-amazo.html
 [2]: http://hross.net/blog/2009/05/monitoring-performance-with-am.html
 [3]: /images/ec2_network_io.png
 [4]: /images/ec2_cpu_usage.png