
+++
title = "UrlRewriter.net and Themes"
description = "As I mentioned in my last post, I've finally given urlrewriter.net a try - just to get over my php f ..."
tags = [ "ASP.NET", "blog" ]
date = "2008-07-28 05:14:00"
slug = "UrlRewriternet-and-Themes"
+++
<p>As I mentioned in my <a href="http://weblogs.asp.net/ashicmahtab/archive/2008/07/28/kaspersky-and-banner-s.aspx">last post</a>, I've finally given <a href="http://urlrewriter.net/">urlrewriter.net</a> a try - just to get over my php faithful friends who always sneered at us for those .aspx extensions.</p>
<p>I watched this video: <a href="http://www.asp.net/learn/videos/video-154.aspx">How Do I: Implement URL Rewriting?</a> by <em>Scott Golightly.</em> While it was good for the basic stuff (and informative), you'd have to code everything by hand. Also,near the end, he shows an implementation of a custom Form class that can handle postbacks with url rewriting. This has the major drawback of having to change the form tags on every page, and also having to register the control on every page....i.e. it's messy. The actual implementation also seems to totally remove the action on the form entirely. Of course, as asp.net forms submit to themselves (non-MVC), I don't know how big a problem this is.</p>
<p>Anyhow, I went on over to <a href="http://weblogs.asp.net/scottgu/archive/2007/02/26/tip-trick-url-rewriting-with-asp-net.aspx">the Gu's tutorial</a> - something I have been wanting to do for ever so long. I did everything according to method three. Everything worked as expected...except for images. If I created some redirect rules that traversed folders (i.e. <a href="http://www.site.com/folder/page">www.site.com/folder/page</a> rewrites to <a href="http://www.site.com/home.aspx?f=folder&amp;p=page">www.site.com/home.aspx?f=folder&amp;p=page</a>), sometimes the themes and images were loading, sometimes they weren't (depending on the rewrite rule). As ScottGu mentioned on his blog, making images have absolute paths helps; but themes seem to cause a problem with more complex rules.</p>
<p>So, I thought for a bit and added this little rewrite rule to the mix:</p>
<p>&lt;rewrite url=".*/App_Themes/(.+)" to="~/App_Themes/$1" /&gt;</p>
<p>What this does is that it takes any request to anything with App_Themes in the url and maps it to the root App_Themes folder. So, no matter what the level of nesting, and no matter what rewrites are used, anything I the App_Themes folder will work perfectly. This works perfectly with themes. And as an added bonus, relative urls like "App_Themes/theme1/image.png" (as opposed to absolute urls like "~/App_Themes/theme1/image.png") will also work for images, css files, etc. And if you don't like the App_Themes folder, you can always add a rewrite for another folder using the same technique as above.</p>
<p>Needless to say, for this to work, you'll need to map every request to asp.net (just as you need with extensionless urls). This is damn easy in IIS7, and tougher on IIS6, as ScottGu points out.</p>
<p>&nbsp;</p>
<p>So, there you have it, a simple hack that makes themes and resources work in perfect harmony with url rewriting. Can't wait to show my php religious friends. And just to add insult to injury, I made an asp.net page specifically for them:</p>
<p><a href="http://heartysoft/LookAtThisAspxPageYouMoron.php">http://Heartysoft/LookAtThisAspxPageYouMoron.php</a></p>
<p>Can't wait till they wake up and log onto the LAN ;)</p>
        