
+++
title = "Rescuing Linux Install"
description = "Some steps that can help fix a broken Linux installation."
tags = [ "blog", "linux" ]
date = "2017-06-05 23:14:30.00"
slug = "rescuing-linux-install"
+++

Last week, I messed up. I was doing some stuff with react redux, and got annoyed at having to sudo install npm stuff. Now there are a few ways around this, but instead of using those, I followed the official page at [https://docs.npmjs.com/getting-started/fixing-npm-permissions](https://docs.npmjs.com/getting-started/fixing-npm-permissions). Here's the steps I followed:

<img src='/Media/Default/images/rescuing-linux.png' style='width:40%' alt='instructions'></img>

Notice that warning that looks like an innocent piece of text? I somehow foolishly missed that. As a result, I got in a bit of a pickle. It applied the permissions to the folders specified, and in my cases that included /usr/bin/* . Which means it applied permissions to /usr/bin/sudo. Now, sudo won't run for anybody else but root, and it looks for specific permissions. However, you can't change permissions on sudo unless you're sudo. Chicken and egg problem, eh? Now if you've enabled logging in as the root user, you can do that, and then change the permission back; you don't need sudo to login as root. In my case, that was disabled, and to enable it - you guessed it - you had to sudo. 

A lot of answers around the internet at this point tell you that the only way to recover from this is to reinstall Linux. But damn, that seems like such a waste. I tried to see if I could somehow boot into a terminal as root instead. Turns out I could. On the GRUB boot menu, I hit 'e' (without quotes, and this is on Antergos... the key might differ for other systems)). This shows you a screen with some details of what's being booted into. A line there should be like:

```
linux /boot/vmlinuz-linux root=....
```

From that line, I deleted "quiet" and added "text rescue". That did what I wanted. I was in a terminal, I could assign permissions to sudo. You too can use this technique to boot to a terminal as sudo. 

With this awesome power, I made my next utterly idiotic move. I blanketly gave sudo ownership of everything in /usr/bin . Coz that must have been what it was originally, right? WRONG. Whereas previously I could boot and run applications, and my only problem was I couldn't use sudo, not things won't boot at all. I couldn't even use the previous trick to get to a useful terminal. Well, it did put me into a terminal as root, but a lot of things hadn't loaded properly, and I couldn't do anything to fix it. Time to call out the heavy artillery. 

I created an Antergos OS Live USB stick, and booted to it. I chose Live mode, and not Install for obvious reasons. The Live environment mounted the existing hardrive, and I had access to a terminal from which I could change permissions to the files on my hard drive. It turns out /usr/bin/write and /usr/bin/wall need root:tty (and not root:root). Also, /usr/bin/sudo needs 4655 (sudo chmod 4655 /usr/bin/sudo), and /usr/bin/write and /usr/bin/wall need 2655 permissions. With those permissions set, I rebooted and everything on my original install worked. 

It was dumb of me to execute a script from a web page (be it npm docs) blindly like that. But if you're ever in a bind you can try out these steps and see if it can salvage a setup.

