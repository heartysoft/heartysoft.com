
+++
title = "Hey Kaspersky, Stop Slowing Down My VS"
description = "I'm finally starting off this blog about a month after Joe gave it to me...yay, awesome, yahoo (or s ..."
tags = [ "General Software Development", "Performance Tips", "blog" ]
date = "2008-07-14 04:39:00"
slug = "Hey-Kaspersky-Stop-Slowing-Down-My-VS"
+++
<p>I'm finally starting off this blog about a month after Joe gave it to me...yay, awesome, yahoo (or should that be MS yahoo?).</p>
<p>One thing that has always frustrated me was that whenever I install Vista (or XP), everything is zippy and fast. Gradually, things become really sluggish. Then, it becomes so unbearable that I have to reinstall the darned thing. The thing that slowed down the most (or at least, is a frontline contender for slowing down) seemed to be VS - the one thing I needed the most (well, maybe after IE). This really was getting on my nerves, and I blamed it on my hardware. So imagine my surprise when, after a recent upgrade, it seemed even zippier than before, but the sluggishness "somehow" returned. Now don't get me wrong - when I say "upgrade", I mean a total renovation - a quad core Q6600, 4GB of DDR2-800MHz (5-5-5 timings to boot), 2 10Krpm Western Digital Raptors in RAID 0, HD 3870 and a Thermaltake PSU that broke my back as well as my bank - some of the best things I could buy back in last September (the HD3870 was bought later). And then I found the culprit.</p>
<p>I've always loved Kaspersky, awesome little thing. But, like any other antivirus, it likes to check every opened file and make sure it's safe. This is all fine and good, but I don't think there's anything lurking inside my .cs files just waiting to pounce on my system. So, I go ahead and add VS to my trusted stuff, give it full permissions, and tell Kaspersky to not even monitor files opened by VS or any of VS's activity. And guess what, it's been months now, and VS is still as zippy as when I first installed Vista. I went ahead and put some other obviously-not-virus-carrier programs into my safe zone, and they work like a dream, too.</p>
<p>I always thought that turning the antivirus off for VS wouldn't give me "that" much of a boost since I had some kick-ass hardware. Boy was I pleased to be proven wrong. I installed Vista x64 in February, and not a single frustrating moment since then.</p>
<p>I use Kaspersky, but the principal should hold for all other virus guards if they have a trusted apps feature or something.</p>
<p>Hope that helps.</p>
        