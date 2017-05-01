
+++
title = "Memorystream Not Expandable: Invalid Operation Exception"
description = "There's a little&nbsp;gotcha with the MemoryStream class that I just found out. It has 7 constructor ..."
tags = [ ".NET", "ASP.NET", "blog" ]
date = "2008-08-24 19:31:00"
slug = "Memorystream-Not-Expandable-Invalid-Operation-Exception"
+++
<p>There's a little&nbsp;gotcha with the MemoryStream class that I just found out. It has 7 constructors. The default constructor has the stream set as expandable, with an initial capacity of 0. The ctors that take the capacity set the capacity to the param, but also keeps the stream expandable. If, however, you use a ctor with the byte[] param, it initializes the stream with the contents of the buffer, but the stream becomes non-expandable.</p>
<p>So, if you have something like this:</p>
<p><pre class='brush:c#'>

byte[] buffer = File.ReadAllBytes("filaname.docx");
MemoryStream ms = new MemoryStream(buffer);
MemoryStream ms2 = new MemoryStream();
ms2.Write(buffer, 0, buffer.Length);

</pre></p>
<p>then, ms will NOT be expandable, ms2 will be. So, if you are modifying the stream and the size may increase, the first ctor will not work, but the second approach will work just fine.</p>
<p>&nbsp;</p>
<p>Hope that helps :)</p>
        