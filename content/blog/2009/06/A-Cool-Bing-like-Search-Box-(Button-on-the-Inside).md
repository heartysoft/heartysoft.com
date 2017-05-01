
+++
title = "A Cool Bing-like Search Box (Button on the Inside)"
description = "In the recent &ldquo;web 2.0&rdquo; sites, we see cool search boxes, with the search button appearin ..."
tags = [ "ASP.NET", "HTML", "blog" ]
date = "2009-06-15 23:57:00"
slug = "A-Cool-Bing-like-Search-Box-(Button-on-the-Inside)"
+++
<p>In the recent &ldquo;web 2.0&rdquo; sites, we see cool search boxes, with the search button appearing &ldquo;inside&rdquo; the text box. <em>How&rsquo;d they do that? </em>Microsoft&rsquo;s bing search also has this feature. A snapshot of the bing search box looks like this:</p>
<p><a href="http://www.heartysoft.com/image.axd?picture=search.png"><img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="search" src="http://www.heartysoft.com/image.axd?picture=search_thumb.png" border="0" alt="search" width="244" height="33" /></a></p>
<p>It&rsquo;s actually quite simple to do, with a bit of css. First, let&rsquo;s look at the markup:</p>
<p><code>[code:html]</code></p>
<p><code>&lt;div class='search-box'&gt;</code></p>
<p><code>&nbsp;&nbsp;&nbsp; &lt;asp:TextBox runat="server" ID='txt1'&gt;&lt;/asp:TextBox&gt;</code></p>
<p><code>&nbsp;&nbsp;&nbsp; &lt;asp:Button runat="server" ID='btn1' Text='Search' /&gt;</code></p>
<p><code>&lt;/div&gt;</code></p>
<p>&nbsp;</p>
<p><code></pre></code></p>
<p>And let&rsquo;s now see the associated css:</p>
<p><code>[code:html]</code></p>
<p><code>&lt;style type="text/css"&gt;</code></p>
<p><code>&nbsp;&nbsp;&nbsp; .search-box input</code></p>
<p><code>&nbsp;&nbsp;&nbsp; {</code></p>
<p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; border:none;</code></p>
<p><code>&nbsp;&nbsp;&nbsp; }</code></p>
<p><code>&nbsp;&nbsp;&nbsp; .search-box</code></p>
<p><code>&nbsp;&nbsp;&nbsp; {</code></p>
<p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; display:inline;</code></p>
<p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; border:1px solid #000000;</code></p>
<p><code>&nbsp;&nbsp;&nbsp; }</code></p>
<p><code>&lt;/style&gt;</code></p>
<p>&nbsp;</p>
<p></pre></p>
<p>And you can see the result:</p>
<p><a href="http://www.heartysoft.com/image.axd?picture=my-search.png"><img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="my-search" src="http://www.heartysoft.com/image.axd?picture=my-search_thumb.png" border="0" alt="my-search" width="233" height="40" /></a></p>
<p>Sure&hellip;it&rsquo;s not exactly the same, but you can play around with it, change the css here and there, use an ImageButton or whatever to make it look exactly like you want. The idea is basically the same.</p>
<p>Hope that helps.</p>
<div class="wlWriterHeaderFooter" style="margin:0px; padding:0px 0px 0px 0px;">
<div class="shoutIt"><a rev="vote-for" href="http://dotnetshoutout.com/Submit?url=http%3a%2f%2fwww.heartysoft.com%2fpost%2f2009%2f06%2f15%2fA-Cool-Bing-like-Search-Box-(Button-on-the-Inside).aspx&amp;title=A+Cool+Bing-like+Search+Box+(Button+on+the+Inside)"><img style="border:0px" src="http://dotnetshoutout.com/image.axd?url=http://www.heartysoft.com/post/2009/06/15/A-Cool-Bing-like-Search-Box-(Button-on-the-Inside).aspx" alt="Shout it" /></a></div>
</div>
        