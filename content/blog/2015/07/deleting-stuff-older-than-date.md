
+++
title = "Deleting stuff older than date"
description = "A bash script to delete things in a folder that haven't changed since a certain date."
tags = [ "blog", "Linux" ]
date = "2015-07-06 14:27:58.570000"
slug = "deleting-stuff-older-than-date"
+++
<p>I seem to be looking for this frequently, and can never remember it. So, more of a reminder for me (and for others!)</p> <p>&nbsp;</p> <p><strong><em><font size="5">find ./ –not –newermt 2015-06-23 | xargs rm –rf</font></em></strong></p> <p><strong><em><font size="5"></font></em></strong>&nbsp;</p> <p><font size="3">This finds all the stuff in the current path (./) that hasn’t changed after 2015-06-23, and deletes them.</font></p>
        