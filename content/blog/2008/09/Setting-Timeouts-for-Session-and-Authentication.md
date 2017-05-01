
+++
title = "Setting Timeouts for Session and Authentication"
description = "Setting the timeouts for Session and Authentication can be a bit tricky. If not set properly, your u ..."
tags = [ "ASP.NET", "blog" ]
date = "2008-09-29 04:59:00"
slug = "Setting-Timeouts-for-Session-and-Authentication"
+++
<p>Setting the timeouts for Session and Authentication can be a bit tricky. If not set properly, your user may be logged in when the Session expires. If you&rsquo;re app depends on Session specific to the logged in user, you&rsquo;ll have problems. By default, the Authentication timeout is 30 minutes while for that of Session is 20 minutes. Here&rsquo;s how you can change that on a site wise basis:</p>
<h4>Session</h4>
<p>Under &lt;system.web&gt;, add this node:</p>
<p>&lt;sessionState timeout="30"&gt;&lt;/sessionState&gt;</p>
<h4>Authentication</h4>
<p>Under &lt;system.web&gt;, have this node:</p>
<p>&lt;authentication mode="Forms"&gt; <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;forms timeout="20"&gt;&lt;/forms&gt; <br />&lt;/authentication&gt;</p>
<p>For both, the timeout is in minutes. Also, note that there are a host of other settings you can change for Authentication, which you can set within the &lt;forms&gt; tag. Intellisense does work.</p>
<p>A lot of people think setting the Session timeout will ensure that the user gets logged out after the Session expires. This is probably because of the fact that there&rsquo;s no timeout in the &lt;authentication&gt; node. The reason that there isn&rsquo;t one is because there are two types of authentication accessible in the &lt;authentication&gt; section: 1) Forms and 2) Passport. There can be different settings for each, and hence the timeout needs to be set in the &lt;forms&gt; tag under the &lt;authentication&gt; tags.</p>
<p>Hope that helps.</p>
        