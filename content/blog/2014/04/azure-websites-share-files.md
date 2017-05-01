
+++
title = "Azure Website Instances Share the Same Files"
description = "Azure Website Instances Share the Same Files"
tags = [ ".NET", "blog", "ASP.NET", "Azure" ]
date = "2014-04-13 21:41:14.617000"
slug = "azure-websites-share-files"
+++
<p>During a conversation on twitter, I learnt something that kind of surprised me. You know the fancy feature where you can drag a slider and increase the number of instances of your website that serves your customers? </p> <p><a href="http://www.heartysoft.com/Media/Default/Windows-Live-Writer/1bc2wj0mfp3bwq2vdrj4abrr/image%5B3%5D.png"><img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" border="0" alt="image" src="http://www.heartysoft.com/Media/Default/Windows-Live-Writer/vubsgdnxcqv54hqwkorwh5n5/image_thumb%5B1%5D.png" width="601" height="46"></a> </p> <p>Well, it turns out that all the instances are served off the same copy of the files. Obviously these files are served off virtualised drives, so it’s not like a disk failure is going to wipe away your websites. However, it does mean that changing the underlying web.config does recycle the app pools of all the instances. As such, any sort of A/B deployments of similar that require any differences in the files will not work. You can still make use of the nifty <a href="http://weblogs.asp.net/scottgu/archive/2014/01/16/windows-azure-staging-publishing-support-for-web-sites-monitoring-improvements-hyper-v-recovery-manager-ga-and-pci-compliance.aspx">staging feature</a> to prepare a deployment before it goes live, but when you push the trigger to go live, all the instances will get it.</p>
        