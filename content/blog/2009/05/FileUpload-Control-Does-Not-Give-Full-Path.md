
+++
title = "FileUpload Control Doesn’t Give Full Path….HELP!!!!"
description = "I&rsquo;ve answered this question about five times in the last few days at the forums. It&rsquo;s ge ..."
tags = [ ".NET", "ASP.NET", "blog" ]
date = "2009-05-19 20:44:00"
slug = "FileUpload-Control-Does-Not-Give-Full-Path"
+++
<p>I&rsquo;ve answered this question about five times in the last few days at the forums. It&rsquo;s getting tedious so I thought I&rsquo;d make a blog post about it.</p>
<p><strong>Issue:</strong></p>
<p>The FileUpload control has a FileName property. In FireFox (FF) and some other standards compliant browsers, the filename only holds the name of the file. It does NOT hold the full path. In some versions of Internet Explorer (perhaps all versions, but at least up until v7), the FileName property holds a full path. Hence, people see discrepancies among the browsers in terms of the FileName property. A lot of you guys are asking how to get the full path from the file upload control.</p>
<p><strong>What does the FileName property do?</strong></p>
<p>The FileName property holds the name of the uploaded file. In FF, it holds only the file name. In IE7 and earlier (possibly IE8 too &ndash; I&rsquo;ll refer to all of them as IE from here on) it holds <strong>THE FULL PATH TO THE FILE ON THE </strong><span style="font-size: large;"><strong><em>CLIENT&rsquo;S MACHINE. </em></strong></span><span style="font-size: x-small;">Read that again, and again, and one more time. It&rsquo;s the full path of the uploaded file on the client&rsquo;s machine. <strong>NOT THE PATH OF THE UPLOADED FILE ON THE<em> SERVER. </em></strong></span></p>
<p><span style="font-size: x-small;"><strong>WHOAH&hellip;Hang on a minute&hellip;the client machine?</strong></span></p>
<p><span style="font-size: x-small;">Yes. It holds the path to the uploaded file on the machine of the user using the website, not the server. It&rsquo;s the path you see in the file upload controls text box. It&rsquo;s the user&rsquo;s <em>local path </em>to the file. The user may have an M drive with a pics folder. The FileName property will then hold something like M:/pics/pic1.png. The server may not even have an M drive with a pics folder.</span></p>
<p><strong><span style="font-size: x-small;">That SUCKS!!!! What good is that?</span></strong></p>
<p><span style="font-size: x-small;">This is a decision made by the creators of IE. It was done to enable easier automation of intranet apps where file operations across the network could be accomplished easier. </span></p>
<p><strong><span style="font-size: x-small;">Hmmm&hellip;Why do them FF guys not support it?</span></strong></p>
<p><span style="font-size: x-small;">Them FF guys think that letting the server know about the folder structure of a user&rsquo;s machine is a serious breach of security. For example, if for some reason you had a pic with the path D:\My Racy Photos With X\pic112.jpg (which may be an official photo for whatever reason) and you upload that to the office server, you wouldn&rsquo;t want your boss (or anyone else) to know that you have a folder called &ldquo;My Racy Photos With X&rdquo; on your pc. That&rsquo;s private info and would be embarrassing if leaked. The worst case scenario would be a few blushes. Now think if you had a folder with some secure info in the name. Even if the contents were secure and not the folder name, you wouldn&rsquo;t want to make it easier on anyone to get to them. Hence, this can be considered a security flaw.</span></p>
<p><strong><span style="font-size: x-small;">Ok&hellip;err&hellip;So where does my uploaded file go to?</span></strong></p>
<p><span style="font-size: x-small;">The uploaded file is uploaded to a temp location on the server. If you process it, fine. If not, it&rsquo;s purged (read deleted).</span></p>
<p><strong><span style="font-size: x-small;">Wait&hellip;how do I store the uploaded file then?</span></strong></p>
<p><span style="font-size: x-small;">Simple. The FileUpload control has a SaveAs(string path) method. You can use that method to save the file to any path using any filename you want. Just make sure asp.net has permissions to write to that path. So, you can do something like this to save the uploaded file to the uploads folder in the root of your website:</span></p>
<p><span style="font-size: x-small;">fileUpload1.SaveAs(Server.MapPath(&ldquo;~/uploads/&rdquo;) + System.IO.Path.GetFilename(fileUpload1.FileName));</span></p>
<p><span style="font-size: x-small;">[The reason I&rsquo;m doing a Path.GetFileName is to ensure that even in IE, I only pass the filename and not the entire client side path to the SaveAs method.]</span></p>
<p><span style="font-size: x-small;">If you wish to provide a different username, you can do that just by changing the parameter. If you wish to append a unique key to each upload, you can use something like Guid.NewGuid().ToString() &ndash;or any other technique you want.</span></p>
<p><strong><span style="font-size: x-small;">Ok smarty pants&hellip;saving to the file system is old school. I wanna save it to a database as an array of bytes. If I don&rsquo;t have the file location, how do I get the array of bytes? How do I use File.ReadAllBytes if I don&rsquo;t have a path? Huh? Huh?</span></strong></p>
<p><span style="font-size: x-small;">Glad you asked. The FileUpload control ha a FileBytes property that gives you an array of bytes with all the contents of the file. Just use that.</span></p>
<p><strong><span style="font-size: x-small;">Now I want to do some processing on the uploaded file. How do I get a Stream without the path?</span></strong></p>
<p><span style="font-size: x-small;">The FileUpload control has a FileContent property that gives you a stream to the file. You can use it like this:</span></p>
<p><em>Stream str = fileUpload1.FileContent; <br />StreamReader sr = new StreamReader(str); <br />string contentText = sr.ReadToEnd();</em></p>
<p><strong>What about the normal &lt;input type=file&gt; file uploads? I don&rsquo;t like the FileUpload control. Can I get the full filename from that?</strong></p>
<p>The FileUpload control uses the &lt;input type=&rsquo;file&rsquo;&gt; tag behind the covers. You&rsquo;ll get no added info about the path from using a basic &lt;input type=&rsquo;file&rsquo;&gt; tag. The same restrictions apply. Using the input tag, you&rsquo;ll have to do a bit more work to get to the file bytes. You can get to each uploaded file in a postback by using Request.Files. This will give you every HttpFileUpload in the request, regardless of if you use a &lt;input type=&rsquo;file&rsquo;&gt; or a FileUpload control. If using the input tag, you can set the runat=&rsquo;server&rsquo; and id attributes to access it from server side code. You can then access it&rsquo;s PostedFile in postback. Each object in Request.Files and each PostedFile of each &lt;input type=&rsquo;file&rsquo; runat=&rsquo;server&gt; and also each PostedFile of each FileUpload is of type HttpFileUpload. That class also has a SaveAs method that does exactly what you&rsquo;d expect it to. If you need to access the stream, then it has an Inputstream property. You can use it to read the contents using a StreamReader as I&rsquo;ve shown above. If you need to access the bytes, the following code will help [please note that HttpFileUpload does not have any direct way to give you all the bytes in a byte array, like the FileUpload control does]:</p>
<p><em>HttpPostedFile file = Request.Files[0]; </em><em>//you can use input</em><em>id.PostedFile too <br />Stream str = file.InputStream; <br />byte[] bytes = new byte[file.ContentLength]; <br />str.Read(bytes, 0, file.ContentLength);</em></p>
<p>After that, bytes will hold the binary content of the file.</p>
<p>Hope that helps.</p>
        