
+++
title = "Display Modes of Validator Controls"
description = "By default, asp.net validators are positioned right next to the control they validate. You can move  ..."
tags = [ "ASP.NET", "Validation", "blog" ]
date = "2009-06-16 04:04:00"
slug = "Display-Modes-of-Validator-Controls"
+++
<p>By default, asp.net validators are positioned right next to the control they validate. You can move them in the markup, but wherever you put them, they occupy an area equal to the area required to display the Text property (or if the Text is not present, then the ErrorMessage property). We may not want that. We may want them to only occupy space when they&rsquo;re displayed, or not display them at all (and showing the error message in a validation summary only). The validators have a property called display, which can be set to one of three values: Static, Dynamic and None. Setting it to Static will mean the validator will occupy space even when there&rsquo;s no error. Dynamic means that it won&rsquo;t occupy space when there isn&rsquo;t an error, but will show up when there is. None means that the validator&rsquo;s Text (or ErrorMessage) isn&rsquo;t displayed at all and doesn&rsquo;t occupy any space. In this case, you&rsquo;ll need to use a ValidationSummary control to be able to display the ErrorMessage.</p>
<p>Let&rsquo;s look at a bit of markup that has three required field validators having different settings for Display. To the right of each validator, I&rsquo;ve added the text &ldquo;dummy&rdquo; to signify where the validator&rsquo;s display area ends. Markup:</p>
<p><a href="http://www.heartysoft.com/image.axd?picture=val-pos-1.png"><img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="val-pos-1" src="http://www.heartysoft.com/image.axd?picture=val-pos-1_thumb.png" border="0" alt="val-pos-1" width="603" height="384" /></a></p>
<p>&nbsp;</p>
<p>Now let&rsquo;s fire up the page and see what it look like in the browser:</p>
<p><a href="http://www.heartysoft.com/image.axd?picture=val-pos2.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="val-pos2" src="http://www.heartysoft.com/image.axd?picture=val-pos2_thumb.png" border="0" alt="val-pos2" width="604" height="199" /></a></p>
<p>Notice that since the first validator had a display of static, there&rsquo;s a space equal to the space needed to display the Text of the first validator next to the first text box. The second and third validators had display set to &ldquo;Static&rdquo; and &ldquo;None&rdquo; respectively. Hence, there&rsquo;s no space between the second and third textboxes and they&rsquo;re respective &ldquo;dummy&rdquo; texts. Now, lets hit submit:</p>
<p><a href="http://www.heartysoft.com/image.axd?picture=val-pos3.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="val-pos3" src="http://www.heartysoft.com/image.axd?picture=val-pos3_thumb.png" border="0" alt="val-pos3" width="610" height="312" /></a></p>
<p>Notice that the Text of the first two validators are displayed next to the text boxes, but the third one&rsquo;s Text is not shown at all. This is because the third validator&rsquo;s display was set to none. Notice that the error message of all three validators are displayed in the validation summary.</p>
<p>&nbsp;</p>
<p>Hope that helps.</p>
<div class="wlWriterHeaderFooter" style="margin:0px; padding:0px 0px 0px 0px;">
<div class="shoutIt"><a rev="vote-for" href="http://dotnetshoutout.com/Submit?url=http%3a%2f%2fwww.heartysoft.com%2fpost%2f2009%2f06%2f15%2fDisplay-Modes-of-Validator-Controls.aspx&amp;title=Display+Modes+of+Validator+Controls"><img style="border:0px" src="http://dotnetshoutout.com/image.axd?url=http://www.heartysoft.com/post/2009/06/15/Display-Modes-of-Validator-Controls.aspx" alt="Shout it" /></a></div>
</div>
        