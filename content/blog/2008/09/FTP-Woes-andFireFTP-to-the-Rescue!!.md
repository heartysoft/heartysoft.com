
+++
title = "FTP Woes and....FireFTP to the Rescue!!"
description = "I'm running Vista x64 sp1. I've had IE 7 and then IE 8 (b2)&nbsp;installed. I tried uploading via ft ..."
tags = [ "ASP.NET", "General Software Development", "blog" ]
date = "2008-09-05 10:50:00"
slug = "FTP-Woes-andFireFTP-to-the-Rescue!!"
+++
<p>I'm running Vista x64 sp1. I've had IE 7 and then IE 8 (b2)&nbsp;installed. I tried uploading via ftp from a)Windows Explorer, b)Visual Studio ftp, c)FireZilla, d)SmartFTP. Small files uploaded perfectly but even 150KB files kept timing out and retrying. Active/passive didn't help. Nothing helped. I tried for 28 hours to upload a 340KB file. It was infuriating. I tried from another ISP's connection. I tried from XP. Nothing worked. God knows why. I contacted tech support, they said it was a problem on my ISP and that my connection was poor. While I agree a 100kbps upload isn't good at all, I do expect to upload 100KB in a few minutes and not failing for hours. I can't be too sure about their response&nbsp;as people using my ISP DO upload via ftp. I sent my files to a friend in the US and he graciously uploaded for me. He mentioned it was slow. And then I needed changes on my site. Enter frustration.</p>
<p>&nbsp;Well...just now I found a brilliant solution that's just plain mind boggling. I learned of FireFTP, an ftp plugin for FireFox. Now, (this may surprise a lot of folks), I don't like FireFox much. On my dev machine, it's painfully slow loading pages from the VS dev server compared to IE7&nbsp;/ IE8 / Chrome. But this is one thing I'll always be using for ftp from now on. You see, FireFTP is slick, fast and it uploaded my 350KB file in 25 seconds flat(!) - something the bigger players failed to do.</p>
<p>&nbsp;I'm guessing this is coz of some OS configuration or firewall or whatever. I tried turing off vista auto tuning. Nothing helped. I faced the same problem from XP though. I wonder if it's my good buddy Kaspersky that's causing me the pain. I did try turning off the firewall, but that didn't help either.</p>
<p>Anyhow, I'm glad I found fireftp. I noticed some people on the forums were complaining about slow ftp too. I hope fireftp can help them too.</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p><strong>EDIT: As Josh points out...I really should have posted this link here:</strong></p>
<p><a href="http://fireftp.mozdev.org/">http://fireftp.mozdev.org/</a></p>
<p><strong>FireFTP: The BEST and SIMPLEST FTP software I've EVER used - and one that just "works".</strong></p>
<p><strong>There you go :D</strong></p>
        