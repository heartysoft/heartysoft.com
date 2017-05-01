
+++
title = "Assigning Port Numbers for the Dev Server"
description = "A lot of times, specially when using web services, we want the dev server to use a specific port. Su ..."
tags = [ "ASP.NET", "blog" ]
date = "2008-10-24 00:25:00"
slug = "Assigning-Port-Numbers-for-the-Dev-Server"
+++
<p>A lot of times, specially when using web services, we want the dev server to use a specific port. Suppose you have a web service and a website which consumes it. You put them in two separate projects. You&rsquo;d want the web service to always use the same port. One option is to set it up in IIS. But what if you don&rsquo;t want to use IIS? What if your putting together a simple test and wish to use the dev server? How do we do that?</p>
<p>Well, the process varies between websites and web application projects.</p>
<p><strong>Web Application Projects</strong></p>
<p>Just right click the project in solution explorer and select properties. In the web tab, you&rsquo;ll get the option to set the port.</p>
<p><a href="http://weblogs.asp.net/blogs/ashicmahtab/WebAppPort_475EF76F.jpg" target="_blank"><img style="BORDER-BOTTOM: 0px; BORDER-LEFT: 0px; DISPLAY: inline; BORDER-TOP: 0px; BORDER-RIGHT: 0px" title="WebAppPort" src="http://weblogs.asp.net/blogs/ashicmahtab/WebAppPort_thumb_262BAF3B.jpg" border="0" alt="WebAppPort" width="522" height="398" /></a></p>
<p><strong>Websites</strong></p>
<p>The procedure is slightly more complex for websites. First, in solution explorer, click the project. Hit F4. This brings up the properties pane. Note, this is NOT the property pages found by right-click&gt;&gt;Property Pages.</p>
<p><a href="http://weblogs.asp.net/blogs/ashicmahtab/WebsitePort_1_2B330E4C.jpg" target="_blank"><img style="BORDER-BOTTOM: 0px; BORDER-LEFT: 0px; DISPLAY: inline; BORDER-TOP: 0px; BORDER-RIGHT: 0px" title="WebsitePort_1" src="http://weblogs.asp.net/blogs/ashicmahtab/WebsitePort_1_thumb_40493C14.jpg" border="0" alt="WebsitePort_1" width="270" height="297" /></a></p>
<p>Set the &ldquo;Use dynamic ports&rdquo; option to False. At this point, you can&rsquo;t assign the Port number. Run the web server once, by making any request to the project (a simple ctrl+F5 should work). Doing this will allow you to change the Port number.</p>
<p><a href="http://weblogs.asp.net/blogs/ashicmahtab/WebsitePort_2_0F4C1C18.jpg" target="_blank"><img style="BORDER-BOTTOM: 0px; BORDER-LEFT: 0px; DISPLAY: inline; BORDER-TOP: 0px; BORDER-RIGHT: 0px" title="WebsitePort_2" src="http://weblogs.asp.net/blogs/ashicmahtab/WebsitePort_2_thumb_0FDCFA62.jpg" border="0" alt="WebsitePort_2" width="269" height="257" /></a></p>
<p>Note that Port number is no longer grayed out. You can now assign the port number to be used. As an alternative to making one request, you can also set the &ldquo;Use dynamic ports&rdquo; to false, close the solution and open it up again.</p>
        