
+++
title = "ASP.NET Routing with IIS 7 – Remember Your Modules"
description = "ASP.NET 4.0 introduces routing and it works very well. With IIS 7, there are some interesting config ..."
tags = [ ".NET", "ASP.NET", "IIS 7", "blog" ]
date = "2010-07-26 18:04:57"
slug = "aspnet-routing-iis7-remember-modules"
+++
<p>ASP.NET 4.0 introduces routing and it works very well. With IIS 7, there are some interesting configuration options that can help improve your site's performance. When done wrong, they can have unexpected side effects. And we're going to look at one of them today.</p>  <h3>Extensionless Urls and Wildcard Mappings</h3>  <p>Before IIS 7, the easy way to get extensionless urls when working with routing would be to have a wildcard mapping (yes, I know there are other ways with ISAPI handlers and stuff, but that's not too simple). A wildcard mapping would simply put every request for every file through the asp.net pipeline running all the managed modules and stuff. This does carry with it a certain performance hit – whether that hit applies to you or not, you'd have had to test for yourself. There's a good chance that you'd need admin privileges to do this, which is another problem if you're on shared hosting. Even without routing, there are a ton of different things that need to be set in IIS, with no simple way of overriding in your individual application (without IIS access).</p>  <h3></h3>  <h3>Enter IIS 7</h3>  <p>IIS 7 reads in a section of the web.config file called &lt;system.webserver&gt;. The &lt;system.webserver&gt; node goes directly inside the &lt;configuration&gt; node ([i.e. it's parallel &lt;system.web&gt;). This section is totally ignored by previous versions and can enable you to set settings that previously required IIS access or some not so nice workarounds to do (if at all possible). In this section, you can tell ASP.NET to run certain modules, not run others – or even to run all managed modules for every request. The latter feature is also the easiest to set up, and simply goes like this:</p>  <p><pre class='brush:xml'>    
    
&lt;system.webServer&gt;     
&#160; &lt;modules runAllManagedModulesForAllRequests=&quot;true&quot;&gt;     
&#160;&#160; &lt;/modules&gt;     
&lt;/system.webServer&gt;     
    
</pre></p>  <p>That would do exactly what you'd expect – it'll run all managed modules for all requests. That's kind of like the wildcard mappings. Not the question is, do we need to run every single managed module for every request? Probably not. So, this can be a chance for improvement. We only want routing to run for all requests – and that should give us extensionless urls. So let's do that now:</p>  <h3>An Improvement</h3>  <p>We'll now tell IIS 7 to not run all managed modules for all requests.</p>  <p><pre class='brush:xml'>    
    
&lt;system.webServer&gt;     
&#160; &lt;modules&gt;     
&#160;&#160; &lt;/modules&gt;     
&lt;/system.webServer&gt;     
    
</pre></p>  <p>That was simple, but that now means that the routing module isn't running. Let's make that run for all requests. We can do so by doing this:</p>  <p><pre class='brush:xml'>    
    
&lt;system.webServer&gt;     
&#160; &lt;modules&gt;     
&#160;&#160;&#160; &lt;remove name=&quot;UrlRoutingModule-4.0&quot; /&gt;     
&#160;&#160;&#160; &lt;add name=&quot;UrlRoutingModule-4.0&quot; type=&quot;System.Web.Routing.UrlRoutingModule&quot; preCondition=&quot;&quot; /&gt;     
&#160; &lt;/modules&gt;     
&lt;/system.webServer&gt;     
    
</pre></p>  <p>With this, our routing should work as expected, while not loading all the other managed modules for every request. So alls well, right?</p>  <h3></h3>  <h3>Hang on a Mo'</h3>  <p>While what we just did is better in the sense that it will only do routing for all requests (which we need for extensionless urls), we have potentially just introduced a bug. We're only running routing for every request, and all the other managed modules will only be run for known ASP.NET resources (for example, .aspx files). Read that again – other than routing, all the other managed modules are only be run for requests to known ASP.NET resources like .aspx files. Say, we have a page at http://mysite.com/folder/index.aspx. If we target that url, all shall be fine – IIS will see that the requested file has an aspx extension and it will run the required managed modules. Now, let's say we use routing to route requests for http://mysite.com/foo to that same page. If we request that url, IIS will look at it and will think it's not for ASP.NET. Since we configured the routing module to run, that will run and the request will get processed correctly by /folder/index.aspx. Keep in mind though, that since we aren't running all managed modules for all requests and don't have a wildcard mapping, IIS will not cause the other managed modules to fire. What does this mean? It means that commonly used functionality might suddenly disappear. If our /folder/index.aspx page uses Session, then we'll get an exception. (The error message for that particular one will keep you guessing – it tells you that you need to set enableSession to true!) In other words, if the page uses Session and if it's requested via it's actual path (i.e. ~/folder/index.aspx with the .aspx extension), then it'll run fine – but if it's requested with ~/foo (i.e. the extensionless route), then trying to access Session will fail. </p>  <h3>So How do We Fix it?</h3>  <p>One obvious way would be to enable all managed modules for all requests. But if we wish to follow our previous approach of only loading the required modules and not any more, then we can do this:</p>  <p><pre class='brush:xml'>    
    
&lt;system.webServer&gt;     
&#160; &lt;modules &gt;     
&#160;&#160;&#160; &lt;remove name=&quot;UrlRoutingModule-4.0&quot; /&gt;     
&#160;&#160;&#160; &lt;add name=&quot;UrlRoutingModule-4.0&quot; type=&quot;System.Web.Routing.UrlRoutingModule&quot; preCondition=&quot;&quot; /&gt;     
&#160;&#160;&#160; &lt;remove name=&quot;Session&quot;/&gt;     
&#160;&#160;&#160; &lt;add name=&quot;Session&quot; type=&quot;System.Web.SessionState.SessionStateModule&quot; preCondition=&quot;&quot;/&gt;     
&#160; &lt;/modules&gt;     
&lt;/system.webServer&gt;     
    
</pre></p>  <p>In other words, we're first removing the Session module and then adding it in again with a blank precondition. This will ensure that the Session module also gets run for all requests and not just for known ASP.NET resources. As such, Session will work regardless of whether you're using a real path or a routed path. Keep in mind though, that if the concerned module is loaded in IIS, then we must ensure we use the exact same name in the &lt;remove&gt; tag(s). There is a bit of magic stringy nature to this, but luckily, you can lookup the module name easily in IIS. In IIS 7 manager, just select the server from the left and double click on &quot;Modules&quot; from the centre pane. You should see a list of all loaded modules with the first column showing the names. That name is what you should give the &lt;remove&gt; tag in the web.config file. Remember, you'll need to do this for all the modules you intend to have running. These include - but are not limited to – Session, Forms Authentication, Windows Authentication etc.</p>  <p>So next time you're doing &quot;optimized&quot; routing and your routed pages don't seem to be working right, check to see if the modules are in fact getting loaded.</p>
        