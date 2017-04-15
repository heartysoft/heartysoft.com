+++
title = "title-hello-2"
description = "Summary of hello-1."
tags = [ ".NET", "NuGet", "blog", "news" ]
date = "2013-11-18T19:20:00-05:00"
categories = [
  "Development",
  "VIM"
]
slug = "slug-hello-2"
+++


We’ve seen the fights on twitter about strong naming of assemblies. There are arguments from both parties. Strong named assemblies can be GAC-ed; but hey, the trend is pretty much anti-GAC – dnx lets you ship the runtime with your app. Right? Right? I’ve not bothered much with strong naming – a) I can’t see what it gives you, b) it’s a hassle that’ll take time (however little) to set up. Most places “requiring” strong naming do it to satisfy some checklist. Anyhow, enough of that.

Proponents of strong naming mention that you have nothing to lose. Strong name something, and Nuget updates, etc. work with binding redirects, etc. if you mention the correct public key. Entity Framework’s package is an example of this. I hear it’s strong named (Haven’t checked in a while…barge pole distance from that sort of stuff and all…), yet things seem to work. So, might as well make things SNed, right?

Oh God, no. Please no. At least for existing public packages. And here’s why:

Taking a Package from Non SN to SN will Break End App Nuget Updates
Let me explain.

I have a nice little Sql Server based event store supporting subscriptions, competing consumers, etc. called Res. Res runs as a Windows service providing TCP access to publish, store, and subscribe to events (shameless plug): https://github.com/heartysoft/res 

To not have to deal with utter networking krap, I used NetMQ (which is the .NET native port of ZeroMQ). And as with most things TCP, for easy consumption, I also provide a NuGet package – Res.Client. The server runs as its own process (though Res.Core allows for a self hosted version useful for testing). However, the client is a library. And here lies the issue – it uses NetMQ too.

The current (0.0.15) version of Res.Client uses NetMQ 3.3.0.8. If an app adds the Res.Client nuget, that’s the version of NetMQ they get. NetMQ 3.3.0.8 is NOT strongly named. Now if the app developer goes to “update” the NuGet package (or say, does an Update All), NetMQ will update itself to 3.3.2.0. Being a patch release, that should be fine. No breaking changes. All should be peachy. And the NuGet update goes smoothly. Even running the app goes smoothly, until anything in Res.Client is used. Why?

The reason is that 3.3.2.0 (actually 3.3.0.10 onwards), NetMQ started strong naming the NuGet dll. What does that bring about? Well, it means ANY app using ANY library built targeting 3.3.0.8 can never update the NetMQ package unless that library updates itself to use a SNed version of NetMQ as well. Even binding redirects won’t help here – since 3.3.0.8 has no public key token, and 3.3.0.10+ has, as far as .NET is concerned, they’re completely different. And making the same NuGet package provide completely different (in .NET’s eyes) libraries should be a no no.