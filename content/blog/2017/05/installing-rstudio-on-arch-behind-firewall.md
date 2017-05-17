+++
title = "Installing RStudio on Arch behind Firewall"
description = "How to install packages on arch from source when your firewall blocks git://."
tags = [ "blog", "linux", "git", "r" ]
date = "2017-05-17 11:44:05.00"
slug = "installing-rstudi-on-arch-firewall"
+++

I've been using Arch (more specifically Antergos) for a while now. This morning I was trying to set up RStudio, and saw that the package I likely want is rstudio-desktop-bin. I tried to yaourt -S it, and during the install, it was trying in install a dependency called gstream. The specific version it was trying to install was built from source, and that source repo had a submodule named common. Unfortunately, the submodule used the git:// protocol that our office firewall blocked. Once I identified this, the solution was simple. I just added the following to my global git config:

```
[url "https://anongit.freedesktop.org/git/gstreamer/common"]
insteadOf = git://anongit.freedesktop.org/gstreamer/common
```
 
When I tried the install again, git used the https url instead of the git:// one, and everything installed fine. This would obviously work whenever you need to use a specific git url that you don't have access to.

