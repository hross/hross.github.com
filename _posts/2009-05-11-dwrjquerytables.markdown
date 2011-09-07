---
layout: post
title: Direct Web Remoting, jQuery and Tables
categories: [jquery, dwr]
---

I recently came across a project where I had a need to display the results of a large SQL query in an HTML table using Java. Of course, I wanted to paginate it, style it, use AJAX to update it, and avoid the need for bulky toolkits or large frameworks. Oh yeah, I was also on an extremely tight deadline (*read:* proper coding and design principles were not used).

I looked into a couple of options:

1.  [Display Tag][1] " I used this for another project, but it uses session variables to paginate and doesn't lend itself well to AJAX updates. 
2.  [GWT][2] " One of the guys over at [Function1][3] used it recently and it looked pretty slick. Unfortunately, it seemed like a lot of overhead and a styling headache for simple “display a table” functionality. 
3.  [DWR][4] and [jQuery][5] " As it turns out, I found a great series of blog posts ([part 1][6], [part 2][7]) over at Spartan Java that pretty much laid out a solution to my problem.

As you can see from the posts on Spartan Java, creating the framework to display SQL results using DWR and jQuery was simple, fast, and fairly straightforward. Because the poster makes some assumptions about your knowledge of DWR and jQuery, I would suggest combining the above with the [getting started guide for DWR][8] and the [jQuery tutorials][9], if you are unfamiliar with either.

If you know basic DHTML and Java the learning curve should be no problem.

### DWR and the Portal

As usual, the code I was writing was eventually destined to show up in the portal. Because jQuery is just a simple javascript library, it works great in the portal without issue. Unfortunately, DWR, like many other AJAX frameworks, has its issues when it is gatewayed.

After combing through the dustier nooks of the documentation, googling profusely, and downloading the source code, I discovered the secret to making DWR work. Since some of you may be thinking “wow, DWR looks like something I want to use in my next portlet”, I thought I'd elaborate:

1.  Add an anchor tag to whatever portlet you will eventually display in the portal. This anchor tag should have an id (let's call it “gatewaybase”) and it should reference the base path of DWR in your application (this will almost always be /dwr/). So, for any portlet I want to use DWR in, I would always have the following (this is in a JSP, you might need to change to another base path): 
    script src="dwr/interface/ClassNameExample.js"&gt;script&gt;
    &gt;script&gt;
    a id="gatewaybase" target="/dwr/" href="/dwr/"&gt;a&gt;

2.  The trick to getting DWR working is intercepting its initialization javascript. As it turns out, DWR unofficially supports this, but does not document it. Assuming you're using jQuery, adding this javascript to either an external .js file, or directly in the page, should do the trick: 
    
        jQuery(document).ready(function() {
            if (typeof(PTPortalPage)!="undefined") {
                //TODO: this check won't work if JS in gateway
                dwr.engine._urlRewriteHandler = doInterceptUrl;
            } else if (document.getElementById("gatewaybase") != null) {
                dwr.engine._urlRewriteHandler = doInterceptUrl;
            }
        });
        
        function doInterceptUrl(data) {
            // this function intercepts http requests from DWR
            // and gateways them using an anchor on the main page
            //TODO: is there a better way? AJAX request for base?
            var rooturl = document.getElementById("gatewaybase").href;
            var nongateroot = document.getElementById("gatewaybase").target;
        
            data = data.replace(nongateroot, rooturl);
            return data; 
        };

What this will do is effectively intercept any javascript requests from your page and add a properly gatewayed URL (via that anchor tag you added in step 1) to the HTTP request.

Also note that you may need to modify your web.xml with the following init-param for DWR:

    servlet&gt;
      servlet-name&gt;dwr-invokerservlet-name&gt;
      display-name&gt;DWR Servletdisplay-name&gt;
      servlet-class&gt;org.directwebremoting.servlet.DwrServletservlet-class&gt;
      init-param&gt;
         param-name&gt;debugparam-name&gt;
         param-value&gt;trueparam-value&gt;
      init-param&gt;
      
      init-param&gt;
         param-name&gt;crossDomainSessionSecurityparam-name&gt;
         param-value&gt;falseparam-value&gt;
      init-param&gt;
    servlet&gt; I'm sure there are probably a million ways to build a better mousetrap when displaying tables with Java, but using the above technologies was quick, easy and rewarding.

 [1]: http://displaytag.sourceforge.net/1.2/
 [2]: http://code.google.com/webtoolkit/
 [3]: http://www.function1.com
 [4]: http://directwebremoting.org/
 [5]: http://jquery.com/
 [6]: http://www.spartanjava.com/2008/paginated-lists-made-really-easy-part-1-of-2-front-end/
 [7]: http://www.spartanjava.com/2008/paginated-lists-made-really-easy-part-2-of-2-back-end/
 [8]: http://directwebremoting.org/dwr/getstarted
 [9]: http://docs.jquery.com/How_jQuery_Works  