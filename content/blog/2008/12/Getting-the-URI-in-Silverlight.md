
+++
title = "Getting the URI in Silverlight"
description = "We may need the uri of the silverlight app in various circumstances (like if we move the app around  ..."
tags = [ "ASP.NET", "blog" ]
date = "2008-12-08 14:52:00"
slug = "Getting-the-URI-in-Silverlight"
+++
<p>We may need the uri of the silverlight app in various circumstances (like if we move the app around in the solution, or are dealing with images and other resources located in a folder on the web site that's not part of the silverlight project). We can get the path easily with:</p>
<p><strong><em>App.Current.Host.Source</em></strong></p>
<p>So, if we are to get a uri to an image in the Images folder in the web site that hosts the silverlight app, we can go:</p>
<p><strong><em>var uri = new Uri(App.Current.Host.Source, "../Images/somepic.jpg");</em></strong></p>
<p>Hope that helps.</p>
        