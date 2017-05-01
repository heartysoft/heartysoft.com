
+++
title = "WCF .svc Gives HTTP 404 on Host, Works Perfectly on Local"
description = "I was recently working on a project that used WCF web services extensively. Everything was ready for ..."
tags = [ ".NET", "AJAX", "ASP.NET", "blog" ]
date = "2008-10-01 00:19:00"
slug = "WCF-svc-Gives-HTTP-404-on-Host-Works-Perfectly-on-Local"
+++
<p>I was recently working on a project that used WCF web services extensively. Everything was ready for deployment, I deployed to the remote host, and voila &ndash; nothing works (nothing WCF related). After a LOT of frustration, I managed to make it work. I found two areas which were causing the problem:</p>
<p>1. The .svc extension was <strong>NOT</strong> setup right at our host. You can set this in the web.config webhandlers section for IIS 7, but not for IIS 6.0. I had IIS 7, but decided to bug my host to set up the extension on the server ;)</p>
<p>2. I had multiple host headers; i.e. my site worked for both <a href="http://mysite.com">http://mysite.com</a> and <a href="http://www.mysite.com">http://www.mysite.com</a>. To resolve this issue, you can either disable all but one if your host permits. Otherwise, you need to use a factory. Here&rsquo;s an example of such a factory:</p>
<p><pre class='brush:c#'>

using System; 
using System.Collections.Generic; 
using System.Linq; 
using System.Web; 
using System.ServiceModel; 
using System.ServiceModel.Activation;

namespace Shipping.WebService 
{ 
&nbsp;&nbsp;&nbsp; public class MultipleIISBindingSupportServiceHostFactory : ServiceHostFactory 
&nbsp;&nbsp;&nbsp; {

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; protected override ServiceHost CreateServiceHost(Type serviceType, Uri[] baseAddresses) 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Uri[] requiredAddress = GetAppropriateBase(baseAddresses);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return base.CreateServiceHost(serviceType, requiredAddress);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Uri[] GetAppropriateBase(Uri[] baseAddresses) 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; List&lt;Uri&gt; retAddress = new List&lt;Uri&gt;(); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; retAddress.Add(baseAddresses[0]);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return retAddress.ToArray();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; }

}

</pre></p>
<p>Granted, it can be way more efficient and you can do nifty tricks with the url to support both http:// and <a href="http://www">http://www</a>, but I can happy to have just one for my webservice.</p>
<p>Next, you need to set the factory in the markup of the ,svc file. You can do it like this:</p>
<p>&lt;%@ ServiceHost Language="C#" Debug="false" <strong>Factory="Shipping.WebService.MultipleIISBindingSupportServiceHostFactory"</strong> Service="Shipping.WebService.ClientService" CodeBehind="ClientService.svc.cs" %&gt;</p>
<p>Hope that helps.</p>
        