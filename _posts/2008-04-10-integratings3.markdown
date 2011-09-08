---
layout: post
title: Integrating Amazon S3 with the Portal
categories: [s3]
---

Once again, my blog posting has been sparse for the past few weeks. But, as the old adage goes: good things come to those who wait. As you can see from the title of this post, good things come in the form of integrating your portal with Amazon's S3 web service framework. Hopefully you think that's cool. Otherwise you may as well stop reading right now.

Okay. For those of you still with me... down the rabbit hole we go...

### What is S3?

[Amazon S3][1] is a recently released pay-as-you -go "**S**imple **S**torage **S**ervice". Hence the alliterative S3 moniker. Simply put, rather than worrying about your storage needs by buying more disk, you open an account with Amazon and they provide you with an unlimited REST/SOAP based interface to store as much content as you want. You pay for uploads/downloads and storage space (prices are on the main page, or you can check out [this calculator][2]). Relatively speaking, the pay as you go model works great for a rapidly expanding site or for those who don't want to deal with the maintenance headaches of keeping up their own storage space.

In terms of the portal, integration with S3 means a place we could store an infinite amount of document data. Ideally, this would be for any of the core ALUI portal services: Knowledge Directory, Collaboration, Publisher, etc.

But how could we go about accomplishing this...?

### Integration Through the Document Repository

Ah yes, my old friend the Document Repository. If you'll recall from previous posts, the repository is just that: a central place for storing document data in the portal. So what if we could somehow create our own repository, or modify the existing one, to upload our documents to Amazon S3 instead of the file system?

As it turns out, I'm guilty of a little unintentional foreshadowing. If you read my comments on repository configuration in Part 1 of Deconstructing the Document Repository, you'll see a mention of the possibility of implementing other types of providers.

And guess what? that's exactly what I ended up doing.

### Ah, but it's never quite that simple...

Unfortunately, in the course of writing this article I discovered a bug in the document repository. Apparently nobody's ever written another provider for it. How do I know that? Well, there's an explicit cast to the FileSystemProvider in one of the basic classes that enable DR operation (yes, I realize that sentence is entirely technical mumbo jumbo). In order to implement your own provider you have to patch the class to get it to work. 

Hopefully, I can convince the guys in engineering to fix this minor bug, but until then I've included a patched version below, along with some install instructions:

1.  Download [this file here][4]. 
2.  Realize it's a .class file and think "what the heck are you having me do to my DR, Ross?" 
3.  Follow these instructions anyway. 
4.  Unpack your *$PT_HOME/ptdr/6.x/webapp/dr.war* file with your favorite zip editor (you'll be doing this later, anyway). 
5.  Open *WEB-INF/lib/dr.jar* (I have previously recommended [WinRAR][5] for these types of things). 
6.  Replace the exact same file under *complumtreedrtransportglueserver.* 
7.  Repack everything, restart and test the document repository as a sanity check. You should be good to go. 

*(P.S. -- I don't have original DR source, so this is decompiled and recompiled code. Did I mention this blog should come with a disclaimer?)*

### Signing Up For S3

The next step in this process is to get yourself an Amazon S3 account. Since it costs money, I'm not going to just go and provide you with mine (nice try though). Simply [sign up here][6] and note your access key and secret key in the email they send you (you'll be using them later).

Now you have two choices... you can continue with me to the ***How it Works*** section, or you can skip right ahead to the ***How to Install*** section.

### How it Works

A glutton for punishment, eh? Alright. Here we go...

#### Managing S3

First things first. There's a great [open source Java toolkit][7] called [JetS3t][7]. It abstracts all those fun SOAP/REST calls you might otherwise be making and gives you straight up Java objects to play with. I highly recommend using it if you're planning on any S3/Java development (there are others for .NET, etc). Here's some links for you to play with if you want to know more:

Out of the box, JetS3t just works: it uses *REST* and *HTTPS *calls only, so security is fairly good. You can reconfigure it by looking at the advanced configuration guide, should you so choose. You can also take a look at the additional jar's it requires ([apache commons][8]) and go from there. 

Should you want to manage your S3 account (and included files) without the benefit of actual programming, I highly recommend you [grab the S3Fox Firefox Plugin][9].

#### Creating a Repository

Before I go any further, let me provide you [this zip file][10] of the appropriate jar's for this project. The only code I've written is contained in *s3provider.jar*. The source I'm about to explain is also included therein.

The source itself is fairly simple. The DR allows us to implement a handful of interfaces (one for a document, one for a repository and one for a factory class that creates the repository). After that, all we have to do is manage where our documents go, how they are named, and implement the requisite functions of each class.

Really, there's not much to the source. I simply generate a new document when asked by the DR, open output/input streams to new or existing documents using their ID's, and generate new GUIDs for new documents after they are uploaded. I used a couple of the pre-existing temp file management classes to make my job even easier. All in all, the hardest part was understanding how the interfaces were supposed to work without any documentation.

The GUID's I used for unique document naming on upload were generated using the [Java Uuid Generator][11], which is an open source native Java implementation that worked quite well for my purposes.

Perhaps you were expecting more complexity? I was fairly impressed with the DR's flexible implementation. Actually, I initially started this project by rolling my own "document repository", but it seemed excessively complicated and I ended up sniffing around the DR source to see if I could do something easier. Turns out I could.

### How to Install

Got bored of the ***How it Works Section***, didn't you?

This will take a bit of effort, and faith, on your part, but I assure you it'll be worth it in the long run (***oh yeah, you might want to back up these files before you start messing with them***): 

1.  First we need to add the appropriate jars to the war file: 
    1.  Unpack your *$PT_HOME/ptdr/6.x/webapp/dr.war* file with your favorite zip editor. 
    2.  Extract [this zip file][10] to your hard drive and add the contained jars to the war file's *WEB-INF/lib* directory. See the ***How it Works*** section for details on what these jars do. 
    3.  Re-pack the war file. 
2.  Open your *$PT_HOME/ptdr/6.1/settings/config/dr-server.xml* file and get ready to start fiddling. 
3.  Now, set up the provider under your desired application node to amazon: **amazon**. The cool thing here is that you could register one or all of your document repository services to work with S3. Simply change the entries for ptcollab, ptupload, etc from the file system provider to the S3 provider. 
4.  Next, you need to configure the provider. Add a provider to the providers section of the configuration file, like so: 

	<provider>
		<name>amazon</name>
		<enabled>true</enabled>
		<factory>com.plumtree.dr.provider.amazon.S3Factory</factory>
		
		<applications>
			<application>
				<name>ptupload</name>
				<aws>
					<encrypted>false</encrypted>
					<awsAccessKey>XXXXXXXXXXXXXXX</awsAccessKey>
				   <awsSecretAccessKey>XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX</awsSecretAccessKey>
				</aws>
			</application>
		</applications>
	</provider>

Note that you will have to add an application section ***for each application you changed in step 2***. However, configuration is simple enough. You will really only need to change three things: 

*   **encrypted** - Are your AWS access key and secret key encrypted? Obviously in a production environment they should be, but for testing purposes I normally leave them in plain text. 
*   **awsAccessKey** - your Amazon S3 public access key 
*   **awsSecretAccessKey** - your Amazon S3 secret access key 

You can encrypt your keys by [using this utility][12] I have provided (extract zip, run bat file or convert to shell script). Incidentally, this can also be used to change your DR passwords (but don't forget to change them on both sides of the wire).

### So Here We Are

At this point you should have a working repository that integrates with an unlimited, web-based file system. If you're a dork like me, you think this is great and are glad you just spent a couple hours of your free time figuring out how to set it up. Otherwise, you probably didn't make it through the entire article.

Amazon S3 seems to have some amazing potential. The entire field of distributed services is an exciting area that could eventually change the way we think about corporate IT, and even the way we do business (wow, I just sounded like a marketing bobble-head for a second there, didn't I?). Hopefully this post will give you some interesting insight into ways you can leverage your portal implementation with some of these new technologies, or at the very least, has provided you with some information on S3.

 [1]: http://www.amazon.com/gp/browse.html?node=16427261
 [2]: http://calculator.s3.amazonaws.com/calc5.html
 [4]: /assets/StartUploadOperation.class
 [5]: http://www.rarlab.com/
 [6]: https://aws-portal.amazon.com/gp/aws/developer/subscription/index.html/104-5852112-7419144?ie=UTF8&amp;serviceID=8&amp;offeringId=6&amp;servicePlanID=6&amp;AWS%5Fredirect=true&amp;awscbctx=Amazon%20Simple%20Storage%20Service&amp;serviceName=Amazon%20Simple%20Storage%20Service&amp;awsrid=&amp;AWS%5Fnode=16427261
 [7]: http://jets3t.s3.amazonaws.com/index.html
 [8]: http://commons.apache.org/
 [9]: https://addons.mozilla.org/en-US/firefox/addon/3247
 [10]: /assets/s3provider.zip
 [11]: http://jug.safehaus.org/
 [12]: /assets/drencrypt.zip  