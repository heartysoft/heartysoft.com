
+++
title = "Retrieving the Password for a User in Asp.net"
description = "If you&rsquo;re using the Membership API in asp.net and need to retrieve a user&rsquo;s password, yo ..."
tags = [ "ASP.NET", "blog" ]
date = "2009-06-14 00:30:00"
slug = "Retrieving-the-Password-for-a-User-in-Aspnet"
+++
<p>If you&rsquo;re using the Membership API in asp.net and need to retrieve a user&rsquo;s password, you can do so by doing this:</p>
<p><pre class='brush:c#'>MembershipUser user = Membership.GetUser("username");
string password = user.GetPassword();
string saferPassword = user.GetPassword("password answer");</pre></p>
<p>The latter is safer as it requires you to pass in the user&rsquo;s security answer as an added check. This will give you the unencrypted password [The default membership system stores hashed passwords in the database].</p>
<p>To support this feature, you&rsquo;ll need to have password retrieval enabled in the web.config. You can do this in the &lt;membership&gt; node under &lt;system.web&gt;. It&rsquo;ll look something like this:</p>
<p><pre class='brush:xml'><span style="color: #0000ff;">&lt;membership defaultProvider=<strong>"myProvider"</strong>&gt; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;providers&gt; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;add connectionStringName="LocalSqlServer" <strong><span style="color: #ff0000;">enablePasswordRetrieval="true" 
</span></strong>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; enablePasswordReset="true" requiresQuestionAndAnswer="true" 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; applicationName="/" requiresUniqueEmail="false" <strong><span style="color: #ff0000;">passwordFormat="Encrypted"</span></strong> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; maxInvalidPasswordAttempts="5" minRequiredPasswordLength="6" 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; minRequiredNonalphanumericCharacters="0" passwordAttemptWindow="10" 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; passwordStrengthRegularExpression="" <strong>name="myProvider"</strong> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; type="System.Web.Security.SqlMembershipProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" /&gt;</span>

<span style="color: #0000ff;">&nbsp;&nbsp;&nbsp;&nbsp; &lt;/providers&gt;</span>

<span style="color: #0000ff;">&lt;/membership&gt;</span><span style="color: #0000ff;"></pre></span></p>
<p>Hope that helps.</p>
<p><strong>As Richard points out, hashed passwords cannot be retrieved. The hash is one way while having the password format set to encrypted enables retrieval of passwords. I&rsquo;ve updated the web.config code to ensure that passwords can be retrieved.</strong></p>
<div class="wlWriterHeaderFooter" style="margin:0px; padding:0px 0px 0px 0px;">
<div class="shoutIt"><a rev="vote-for" href="http://dotnetshoutout.com/Submit?url=http%3a%2f%2fwww.heartysoft.com%2fpost%2f2009%2f06%2f13%2fRetrieving-the-Password-for-a-User-in-Aspnet.aspx&amp;title=Retrieving+the+Password+for+a+User+in+Asp.net"><img style="border:0px" src="http://dotnetshoutout.com/image.axd?url=http://www.heartysoft.com/post/2009/06/13/Retrieving-the-Password-for-a-User-in-Aspnet.aspx" alt="Shout it" /></a></div>
</div>
        