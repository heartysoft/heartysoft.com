
+++
title = "How to use Session values in an HttpHandler"
description = "When writing a custom HttpHandler, by default you have no access to the Session object. Doing someth ..."
tags = [ ".NET", "ASP.NET", "blog" ]
date = "2008-09-18 08:30:00"
slug = "How-to-use-Session-values-in-an-HttpHandler"
+++
<p>When writing a custom HttpHandler, by default you have no access to the Session object. Doing something like HttpContext.Current.Session also returns null. The workaround is quite simple:</p>
<p>Reference the System.Web.SessionState namespace:</p>
<p>using System.Web.SessionState;</p>
<p>&nbsp;...and decorate the handler with either the IRequiresSessionState attribute:</p>
<p><strong>public class MyHandler:IHttpHandler, IRequiresSessionState</strong><span style="color: #e0e0e0; font-size: x-small;"><span style="color: #e0e0e0; font-size: x-small;">&nbsp;</span></span></p>
<p><span style="color: #e0e0e0; font-size: x-small;"><span style="color: #e0e0e0; font-size: x-small;"><span style="color: #000000;">or the IReadOnlySessionState attribute:</span></span></span></p>
<p><span style="color: #e0e0e0; font-size: x-small;"><span style="color: #e0e0e0; font-size: x-small;"><span style="color: #000000;">
<p><strong>public class MyHandler:IHttpHandler, IReadOnlySessionState</strong></p>
<p>with the latter giving read only access to the seesion object.</p>
<p>&nbsp;</p>
<p>Hope that helps.</p>
<p>&nbsp;</p>
<p>EDIT: As pointed out...IReadOnlySessionState and IRequiresSessionState are not attributes, but empty interfaces. This is pretty apparent from the fact that we're not decorating the handler, rather the handler implements the interface [and since it's an empty interface, we don't need to implement anything to do so]. Late night blogging can lure the fingers to strange routes on the keyboard, it seems 8)</p>
</span></span></span></p>
        