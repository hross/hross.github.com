---
layout: post
title: Monitoring Performance with Amazon CloudWatch
categories: [ec2]
---

It is rare that I am on the bleeding edge of technology. Normally, I don't think its worth the time and effort necessary to learn something brand new unless it has been at least somewhat widely adopted and accepted by the community at large.

Oddly enough, my blog post about [running a game server on EC2][1] turned out to be perfectly timed, as Amazon [launched its new CloudWatch, Elastic Scaling and Load Balancing services on Sunday][2]. And since, as I discussed earlier, I have been looking at ways to monitor the usage of my EC2 game server, I somehow find myself on the bleeding edge of the cloud.

### Why CloudWatch?

As I discussed in my [previous post][1], setting up monitoring on an EC2 instance wasn't that hard to do. However, it did come with some drawbacks:

*   **Maintenance** " Although it can be fun to install new software and learn its in's and out's, the actual task of upgrading that software, maintaining it, patching it, watching it for security risks, etc, etc is a major pain in the rear end. [CloudWatch][3] solves this problem by providing a simple service for retrieving performance data, no maintenance or special setup required. 
*   **Granularity** " As I discovered with [munin][4], there are limitations to the frequency with which you can store performance data, not to mention the storage requirements for vast quantities of it. Again, this is hidden from us in the case of [CloudWatch][3]. 
*   **Performance** " Last but certainly not least, monitoring something usually incurs a performance hit. In my previous article I was sampling data on the same host I was tracking statistics from. The very act of collecting performance data could cause that data to be skewed. Since [CloudWatch][3] abstracts this away from individual instances, this is no longer a problem. 

### Getting Started With CloudWatch

There are quite a few resources available to get you started with CloudWatch. I recommend taking a look at the [javascript scratch pad][5] and the other various [developer libraries already available][6] (more on this later).

If you really want to get down to the nitty gritty, you should start with the CloudWatch command line interface (CLI). Here are some simple steps to get you started:

1.  Download the [EC2 API Tools][7] first (you'll need them to set up monitoring). Check out the [Getting Started Guide][8] for instructions on extracting the tools and setting up the proper environment variables. 
2.  Download the [CloudWatch API Tools][9]. Check out the included readme for details on environment variable setup. 
3.  Start up an EC2 instance like you normally would (see my [previous post][1]). 
4.  Enable monitoring on your running instance using the EC2 API Tools command: *ec2-monitor-instances .* 
5.  Take a look at the [CloudWatch Getting Started Guide][10] for details on the available monitoring parameters, etc. 
6.  Run the CloudWatch command *mon-get-stats* to get some statistics from your running instance (*mon-get-stats "help* should give you some examples). 

Here are a few things to keep in mind when running the command line utility:

*   I normally output data to a CSV file so I can create fancy graphs in Excel. Here is an example command (Windows) that delimits stats by comma and outputs to a CSV file: 
        mon-get-stats CPUUtilization --start-time 2009-05-19T21:00:00
         --end-time 2009-05-19T22:00:00 --period 60 --statistics Average 
        --namespace AWS/EC2 --delimiter "," 
        --dimensions "InstanceId=i-2bb5cc42" &gt; stats.csv

*   Timestamps " As per the forums, input timestamps are in [ISO-8601][11] format with the default timezone UTC (Eastern Standard Time + 4 hours). Output timestamps are in UTC and cannot be changed (so start thinking in Greenwich Mean Time). 
*   Virtually as soon as monitoring is enabled, statistics are retrieved from your instances. Data is available up to a per-minute frequency and is stored for two weeks. 

### Writing a Simple Java Monitoring Utility

As much fun as I was having trying to parse and decipher various command line inputs, I was somewhat disappointed in the output. For one thing, there was the time formatting problem. For another, only one set of statistics (CPU utilization, network I/O, etc) were available at one time.

I am not one to do more work than I need to, so instead of setting off to invent an uber-utility for aggregating data, I simply downloaded the [Java library for CloudWatch][12] and hacked up some of the sample code until I had a very basic utility for downloading and aggregating the data I wanted. I present it below in case someone finds it useful:

    import java.io.File;
    import java.io.FileOutputStream;
    import java.io.FileWriter;
    import java.io.IOException;
    import java.util.ArrayList;
    import java.util.HashMap;
    import java.util.Iterator;
    import java.util.Properties;
    
    import com.amazonaws.cloudwatch.AmazonCloudWatch;
    import com.amazonaws.cloudwatch.AmazonCloudWatchClient;
    import com.amazonaws.cloudwatch.AmazonCloudWatchException;
    import com.amazonaws.cloudwatch.model.Datapoint;
    import com.amazonaws.cloudwatch.model.GetMetricStatisticsRequest;
    import com.amazonaws.cloudwatch.model.GetMetricStatisticsResponse;
    import com.amazonaws.cloudwatch.model.GetMetricStatisticsResult;
    
    public class GrabStats {
    
        public static void main(String[] args) {
            
            String fileName = "C:\stats.csv";
    
            String startTime = "2009-05-19T20:00:00";
            String endTime = "2009-05-20T00:00:00";
            
            String[] statList = { "CPUUtilization","NetworkIn","NetworkOut" }; //(%, bytes, bytes)
            
            HashMap&gt; map = new HashMap&gt;();
            
            // grab stats for each stat value
            for (int i = 0; i  stats = getStatistics(startTime, endTime, statList[i]);
                map.put(statList[i], stats);
            }
            
            // write to disk
            try {
                FileWriter fw = new FileWriter(fileName);
                
                // write the header
                fw.write("Date");
                for (int i = 0; i ",");
                    fw.write(statList[i]);
                }
                fw.write("n");
                
                // get a date iterator from our first statistic
                Iterator dateIterator = map.get(statList[0]).keySet().iterator();
    
                while(dateIterator.hasNext()) {
                    String date = dateIterator.next();
                    fw.write(date);
                    
                    // get values for each stat at this date
                    for (int i = 0; i value = map.get(statList[i]).get(date);
                        fw.write(",");
                        fw.write(value.toString());
                    }
                    
                    fw.write("n");
                }
                
                fw.close();
            } catch (IOException ex) {
                // error storing data
                System.out.print("Error writing file: " + fileName);
            }
    
        }
    
        // define the cloudwatch service (should be a singleton)
        private static final String _accessKeyId = "";
        private static final String _secretAccessKey = "";
        private static AmazonCloudWatch _service = new AmazonCloudWatchClient(
                _accessKeyId, _secretAccessKey);
    
        public static HashMap getStatistics(String startTime,
                String endTime, String statName) {
            HashMap map = new HashMap();
    
            // build the request with some defaults
            GetMetricStatisticsRequest request = new GetMetricStatisticsRequest();
            ArrayList stats = new ArrayList();
            stats.add("Average");
            request.setStartTime(startTime);
            request.setEndTime(endTime);
            request.setPeriod(60); // statistics every minute
            request.setMeasureName(statName);
            request.setNamespace("AWS/EC2");
            request.setStatistics(stats);
    
            try {
    
                GetMetricStatisticsResponse response = _service
                        .getMetricStatistics(request);
    
                if (response.isSetGetMetricStatisticsResult()) {
                    GetMetricStatisticsResult getMetricStatisticsResult = response
                            .getGetMetricStatisticsResult();
                    java.util.List datapointsList = getMetricStatisticsResult
                            .getDatapoints();
                    for (Datapoint datapoints : datapointsList) {
                        map.put(datapoints.getTimestamp(), datapoints.getAverage());
                    }
                }
    
            } catch (AmazonCloudWatchException ex) {
    
                System.out.println("Caught Exception: " + ex.getMessage());
                System.out.println("Response Status Code: " + ex.getStatusCode());
                System.out.println("Error Code: " + ex.getErrorCode());
                System.out.println("Error Type: " + ex.getErrorType());
                System.out.println("Request ID: " + ex.getRequestId());
                System.out.print("XML: " + ex.getXML());
            }
    
            return map;
        }
    
    }

### Conclusion

The CloudWatch tools and utilities are nothing less than I'd expect from Amazon. Everything worked as expected, the documentation was well put together and there were no real surprises with the API. Overall, I am very satisfied with the finished product of my meager efforts.

There are, of course, a few shortcomings:

1.  It would be nice to have more statistics available (memory usage being the main one I'm thinking of). Having the ability to define and collect your own statistics via an API would be even better. Since the API already has a flexible way of defining statistic and type, I have to assume this is coming. 
2.  Output visualization is certainly lacking. It would be great to see someone hack a Google Chart generator into the javascript scratch pad (given my lack of copious amounts of free time, this person won't be me). 
3.  Adding some statistic collection and enablement to ElasticFox would certainly make things easier to set up and administer. 

I have to assume these drawbacks will be addressed in future updates, as they have been in the past. I am willing to accept them as the price to pay for being on the bleeding edge of the cloud.

 [1]: http://hross.net/blog/2009/05/running-a-game-server-on-amazo.html
 [2]: http://aws.amazon.com/about-aws/whats-new/2009/05/17/monitoring-auto-scaling-elastic-load-balancing/
 [3]: http://aws.amazon.com/cloudwatch/
 [4]: http://munin.projects.linpro.no/
 [5]: http://developer.amazonwebservices.com/connect/entry.jspa?externalID=2521&amp;categoryID=187
 [6]: http://developer.amazonwebservices.com/connect/kbcategory.jspa?categoryID=187
 [7]: http://developer.amazonwebservices.com/connect/entry.jspa?externalID=351&amp;categoryID=88
 [8]: http://docs.amazonwebservices.com/AWSEC2/2008-12-01/GettingStartedGuide/
 [9]: http://developer.amazonwebservices.com/connect/entry.jspa?externalID=2534&amp;categoryID=88
 [10]: http://docs.amazonwebservices.com/AmazonCloudWatch/latest/DeveloperGuide/index.html
 [11]: http://en.wikipedia.org/wiki/ISO_8601
 [12]: http://developer.amazonwebservices.com/connect/entry.jspa?externalID=2517&amp;categoryID=187  