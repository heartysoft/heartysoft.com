
+++
title = "LINQ to SQL dbml - More than just for accessing a db"
description = "Hi guys...I've been gone nearly a month now. I've finished the last of my classes of my last year of ..."
tags = [ "ASP.NET", "blog" ]
date = "2008-11-20 18:02:00"
slug = "LINQ-to-SQL-dbml-More-than-just-for-accessing-a-db"
+++
<p>Hi guys...I've been gone nearly a month now. I've finished the last of my classes of my last year of my BSc degree (final exams are in January) and we were having all sorts of grad parties. Add to that finishing off a freelance project and my new role as one of the moderators of <a href="http://forums.asp.net">http://forums.asp.net</a>, blogging had to take a back seat. Anyway, I'm back :)</p>
<p>I recently formatted my dev machine with a fresh install of everything. I always keep my documents and all projects in a drive other than my OS drive, so I didn't think much of it. To my horror, I found that a pet project that I had worked on a couple of months ago had its db in the default instance of SQL Server. I had backed up scripts for each db other than that one. I'd worked quite a bit on that project before putting it on hold. And there I was - having lost the database. The db was relatively complex and it would've been a major pain trying to recreate the db. I could start afresh and try to make it work with the site code I had - but I really really didn't want to. Looking through my source files, I found that amazing dbml file. And I was overjoyed. Why? Pretty simple.</p>
<p>Linq to SQL dbml files can be used to create the database, as well as access it. I fired up VS, added a page, made it the default and added two simple lines of code in Page_Load:</p>
<p><strong>StudioMorphDBDataContext dc = new StudioMorphDBDataContext(); <br />dc.CreateDatabase();</strong></p>
<p>I made sure that there was correct connection info in the web.config and ran the page. And behold - I had a database with all the tables and stuff. Of course, things like stored procs and functions won't get recovered, but at least it gave me the schema (with all the relations) back. It saved me from having to design and create the tables from scratch.</p>
<p>It's a miracle :D</p>
        