
+++
title = "OpenVPN Disconnecting on Windows - Reset WinSock"
description = "If OpenVPN disconnects after successfully connecting, this command might help."
tags = [ "blog", "General Software Development" ]
date = "2016-05-21 15:02:34.477000"
slug = "open-vpn-disconnecting-reset-winsock"
+++
<p>For the last few days, I just couldn’t connect to work VPN from my Windows 10 laptop. Rather, I could connect, and authenticate successfully, but as soon as I accessed anything, it would disconnect stating a tls-error, or decrypt-error. I tried numerous things – uninstalling VirtualBox, lowering MTUs to 1492, then to 1472, cleared out OpenVPN registry settings, tried Viscosity (which looks quite nice, and is very cheap – but not free). Nothing worked. </p> <p>In the end, a simple reset of WinSock sorted it out. I opened up an admin command prompt, and ran the following:</p> <p>&nbsp;</p> <p><em><strong><font size="5">netsh winsock reset catalog</font></strong></em></p> <p><strong><em><font size="5"></font></em></strong>&nbsp;</p> <p>In hindsight, I was playing around with socks proxies, and ssh tunneling to get into a dev private AWS subnet, but the issue was hard to trace down, and I hope this can help somebody else in the future. </p>
        