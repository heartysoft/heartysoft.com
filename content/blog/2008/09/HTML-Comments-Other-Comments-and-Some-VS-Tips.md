
+++
title = "HTML Comments, Other Comments and Some VS Tips"
description = "I just found out something really weird about HTML comments. Apparently, according to standar ..."
tags = [ "ASP.NET", "blog" ]
date = "2008-09-02 02:50:00"
slug = "HTML-Comments-Other-Comments-and-Some-VS-Tips"
+++
<p>&nbsp;</p>
<p>I just found out something really weird about HTML comments. Apparently, according to standards, you can't do this:</p>
<p>&lt;!--&nbsp; comment ------------------------ --&gt;</p>
<p>That is, you can't put two or more &lsquo;-&lsquo;s inside a comment. It works as expected in IE7, but it breaks down horribly in FireFox. FireFox simply renders the comment to the user, which is pretty useless as that's not what comment are for!!. The standards "recommend" developers to not put two or more consecutive &lsquo;-&lsquo;s inside an html comment. Now, that's weird. If there are specific delimeters like &lt;!-- and &nbsp;--&gt;, then I should be able to use any part of those delims (but not the whole) inside a comment, shouldn't I?</p>
<p>Since, we're in the vicinity of comments, let's look at some for reference:</p>
<p>HTML: &lt;!-comment --&gt;</p>
<p>C#: //single line comment</p>
<p>/* multiline comment */</p>
<p>VB.NET: &lsquo;single line comment</p>
<p>SQL: /* multiline comment */</p>
<p>-- single line comment</p>
<p>Asp.Net markup (aspx): &lt;%-- comment --%&gt;</p>
<p>&nbsp;</p>
<p>Now here's a neat trick regarding commenting and uncommenting code in Visual Studio and Visual Web Developer. In any code editor, highlighting a bit of code (be it VB.Net, C# or aspx markup), if you press ctrl+(k, c) [That is, press k and then c while holding down control), it'll automagically comment out that part using the comments that apply to the selected code (so, // in C# and &lsquo; in VB.Net etc). To uncomment, highlight and hit ctrl+(k,u). One thing to keep note is that the uncomment will work if you select any part of the beginning of a line for VB.Net and C#. For aspx markup though, you must select from &lt;% to the closing %&gt; for the ctrl+(k,u) uncommenting to work.</p>
<p>Also, asp.net &lt;%-- --%&gt; comments work server side. So, nothing in them are visible to the compiler. HTML comments work client side. So, if you put anything inside &lt;!-- --&gt;, it'll be visible to the compiler. Hence, if you're trying to comment out some part of markup that fails to compile, you must use the server side &lt;%-- --%&gt; comments.</p>
<p>&nbsp;</p>
<p>One last trick for today is the auto formatting of code. In a code editor in Visual Studio or Visual Web Developer, you can select some code and hit ctrl+(k,f). This'll auto format that part of the code. This is immensely helpful for formatting aspx markup really quickly. Keep in ind though, that your code must be valid and error-free. If the ctrl+(k,f) trick doesn't work, it's a sign that there's a problem with code/markup.</p>
<p>Hope that helps.</p>
        
