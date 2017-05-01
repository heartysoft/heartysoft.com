
+++
title = "How to Read the HTML of a Web Page Programmatically"
description = "We might need to read the contents of some page (local or remote) by code. This is quite simple in . ..."
tags = [ ".NET", "ASP.NET", "blog" ]
date = "2008-11-21 04:38:00"
slug = "How-to-Read-the-HTML-of-a-Web-Page-Programmatically"
+++
<p>We might need to read the contents of some page (local or remote) by code. This is quite simple in .net.</p>
<p><em>using System.Net; <br />using System.IO;</em></p>
<p><em>WebRequest req = WebRequest.Create("</em><a href="http://www.asp.net"><em>http://www.asp.net");</em></a> <br /><em>WebResponse res = req.GetResponse(); <br />StreamReader sr = new StreamReader(res.GetResponseStream()); <br />string html = sr.ReadToEnd();</em></p>
<p>The string <em>html</em> will then hold the html contents of <a href="http://www.asp.net">www.asp.net</a>. We can also use relative uris in the same website:</p>
<p><em>WebRequest req = WebRequest.Create(new Uri("somepage.aspx", UriKind.Relative));</em></p>
<p>Hope that helps.</p>
        