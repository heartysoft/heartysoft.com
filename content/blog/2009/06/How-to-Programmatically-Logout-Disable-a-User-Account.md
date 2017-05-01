
+++
title = "How to Programmatically Logout / Disable a User Account"
description = "There may be the need to programmatically logout the user in an asp.net application. If you&rsquo;re ..."
tags = [ "ASP.NET", "blog" ]
date = "2009-06-13 22:29:00"
slug = "How-to-Programmatically-Logout-Disable-a-User-Account"
+++
<p>There may be the need to programmatically logout the user in an asp.net application. If you&rsquo;re using Forms authentication, this is very simple to do:</p>
<p><pre class='brush:c#'>

FormsAuthentication.SignOut();

</pre></p>
<p>You may also need to block the currently logged in / specific user from the system. This may be needed to avoid th possibility of someone doing brute force attacks to get to site data (once they&rsquo;re logged in). This is also quite simple:</p>
<p><pre class='brush:c#'>

MembershipUser user = Membership.GetUser(); //to block currently logged in user

MembershipUser user = Membership.GetUser("username"); //To block a specific user:

user.IsApproved = false; 
Membership.UpdateUser(user);

</pre></p>
<p>To unblock the user, all you&rsquo;d need to do is:</p>
<p><pre class='brush:c#'>

MembershipUser user = Membership.GetUser("username");
user.IsApproved = true; 
Membership.UpdateUser(user);&nbsp;

</pre></p>
<p>Hope that helps.</p>
<div class="wlWriterHeaderFooter" style="margin:0px; padding:0px 0px 0px 0px;">
<div class="shoutIt"><a rev="vote-for" href="http://dotnetshoutout.com/Submit?url=http%3a%2f%2fwww.heartysoft.com%2fpost%2f2009%2f06%2f13%2fHow-to-Programmatically-Logout-Disable-a-User-Account.aspx&amp;title=How+to+Programmatically+Logout+%2f+Disable+a+User+Account"><img style="border:0px" src="http://dotnetshoutout.com/image.axd?url=http://www.heartysoft.com/post/2009/06/13/How-to-Programmatically-Logout-Disable-a-User-Account.aspx" alt="Shout it" /></a></div>
</div>
        