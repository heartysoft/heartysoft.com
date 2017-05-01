
+++
title = "Disable Caching in an HttpHandler"
description = "I was generating some custom reports in Word 2007 format today. The reports were being served by an  ..."
tags = [ ".NET", "ASP.NET", "blog" ]
date = "2008-09-18 10:56:00"
slug = "Disable-Caching-in-an-HttpHandler"
+++
<p>I was generating some custom reports in Word 2007 format today. The reports were being served by an HttpHandler and various params are passed to it (mostly by query string). One report needed a list of ids to be passed and the query string wasn't an option there, so I put that in Session. [My other post today shows how]. The trouble was that the urls were identical and someone clever (the browser or the server) was caching the report. So, changing the parameter that was made up of ids resulted in no change of the report. Now, output caching is pretty simple to eliminate on pages, and for asmx web services for that matter, but I found that doing so for a handler is slightly tricky. Here's what I did:</p>
<p><pre class='brush:c#'> context.Response.Clear(); context.Response.Cache.SetCacheability(HttpCacheability.Public); context.Response.Cache.SetExpires(DateTime.MinValue); </pre></p>
<p>&nbsp;in the ProcessRequest method.</p>
<p>I would've thought context.Response.Cache.SetCacheability(HttpCacheability.None) would do it, but that kept giving me errors when downloading the docx file. Rather, enabling cacheing and forcing a timeout seems to do it.</p>
<p>&nbsp;</p>
<p>Hope that helps.</p>
        