
+++
title = "Error Installing SQL 2008 Business Intelligence Studio and Related Package Load Error in VS2008"
description = "It&rsquo;s been a while since I last blogged. I&rsquo;ve become a forum moderator, an All-Star and a ..."
tags = [ "ASP.NET", "blog" ]
date = "2009-02-21 06:35:00"
slug = "Error-Installing-SQL-2008-Business-Intelligence-Studio-and-Related-Package-Load-Error-in-VS2008"
+++
<p>It&rsquo;s been a while since I last blogged. I&rsquo;ve become a forum moderator, an All-Star and also completed my final exams for my bachelor&rsquo;s degree since then. Now that I have a little more time, I hope to contribute more often.</p>
<p>I was installing SQL Server 2008 Dev Edition with all features, and ran into some problems. Poking around the internet, I found that the problems arise if VS2008 is installed anywhere other than the default location that the installer provides. Experimenting a bit, I found that these steps help resolve the issues:</p>
<p><strong>Installing SQL 2008 Dev Edition with VS 2008 installed in a non-default location:</strong></p>
<p>First try to install SQL 2008 Dev Edition. Everything should succeed except for the Business Intelligence Studio (BIS). After the installer completes stating that BIS install failed, find the folder which contains devenv.exe (the VS executable). For me this was: D:\Program Files (x86)\Microsoft Visual Studio 9.0\Common7\IDE. In that folder, find and copy devenv.exe.config (you may need to have extensions displayed in folder options). Next, browse to one of the following:</p>
<p><strong>For x64 machines:</strong> C:\Program Files (x86)\Microsoft Visual Studio 9.0\Common7\IDE</p>
<p><strong>For x86 machines:</strong> C:\Program Files\Microsoft Visual Studio 9.0\Common7\IDE</p>
<p>[If your Windows installation is in another drive, replace &lsquo;C&rsquo; with that drive&rsquo;s letter.]</p>
<p>Paste the devenv.exe.config file in that folder. After that, try running the SQL 2008 installer again and BIS should install.</p>
<p><strong>Holy cow, what on Earth are all these package load failures in VS2008? Damn you, you broke my VS!!</strong></p>
<p>At this point, going into design view in VS should raise all hell with package load failures. The reason is pretty much the same as why the BIS install fails. The BIS packages don&rsquo;t work right with non-default locations for VS2008. To resolve this, go to one of the following folders:</p>
<p><strong>For x64 machines:</strong> C:\Program Files (x86)\Microsoft Visual Studio 9.0\Common7\IDE</p>
<p><strong>For x86 machines:</strong> C:\Program Files\Microsoft Visual Studio 9.0\Common7\IDE</p>
<p>[If your Windows installation is in another drive, replace &lsquo;C&rsquo; with that drive&rsquo;s letter.]</p>
<p>Copy everything (copy, don&rsquo;t cut. You need them there as well). Now go to the folder that contains devenv.exe. For me this was: D:\Program Files (x86)\Microsoft Visual Studio 9.0\Common7\IDE. Paste everything you just copied. [Overwriting should be safe.] VS should work as expected from now on.</p>
<p>There are &ldquo;better&rdquo; ways to work around the problem. But they involve messing around with the temp SQL installation config files at install time, and I found that this is the easiest and quickest solution.</p>
<p>Hope that helps.</p>
        