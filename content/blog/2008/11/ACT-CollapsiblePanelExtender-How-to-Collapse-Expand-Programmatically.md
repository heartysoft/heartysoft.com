
+++
title = "ACT: CollapsiblePanelExtender - How to Collapse / Expand Programmatically"
description = "The CollapsiblePanelExtender is a cool way to convert simple panels to collapsible ones. It usually  ..."
tags = [ "AJAX", "AJAX Control Toolkit", "ASP.NET", "blog" ]
date = "2008-11-21 02:00:00"
slug = "ACT-CollapsiblePanelExtender-How-to-Collapse-Expand-Programmatically"
+++
<p>The CollapsiblePanelExtender is a cool way to convert simple panels to collapsible ones. It usually has a target panel, a control which can make it collapse, another (or the same) control to make it expand etc. Clicking the said controls will trigger the collapse/open behaviour.</p>
<p>We may need to sometimes control this behaviour from code - both client side javascript and server side code behind. Suppose we have a CollapsiblePanelExtender with the Id 'cpe1'. We wish to manipulate the state via code.</p>
<p><strong>Client Side</strong></p>
<p>Client side manipulation is pretty simple and can be found in the tutorials section in <a href="http://www.asp.net/learn">www.asp.net/learn</a> - we just need to call these methods:</p>
<p>To open: $find('cpe1')._doOpen(); <br />To close: $find('cpe1')._doClose();</p>
<p>The $find tracks down the behaviour object and calls the _doOpen or _doClose methods.</p>
<p><strong>Server Side</strong></p>
<p>This is surprisingly slightly trickier. You'd think that just calling</p>
<p><em>ScriptManager.RegisterClientScriptBlock(this, typeof(Page), DateTime.Now.ToString(), "$find('cpe1')._doOpen();", true);</em></p>
<p>would work. It doesn't. Any attempt to call $find('cpe1') results in a null. This is likely because of the fact that behaviours are added at the end and our script gets registered before the behaviour is available. The actual way to do it is actually quite simple (once you know how):</p>
<p>To open: cpe1.Collapsed = false; cpe1.ClientState = "false"; <br />To close: cpe1.Collapsed = true; cpe1.ClientState = "true";</p>
<p>Setting just the Collapsed property is not enough, you need to set both. It's actually a lot less messy than registering javascript, but it is very annoying until you know it.</p>
<p>Hope that helps.</p>
        