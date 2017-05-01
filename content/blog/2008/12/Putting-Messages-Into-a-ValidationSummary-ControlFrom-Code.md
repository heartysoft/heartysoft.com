
+++
title = "Putting Messages Into a ValidationSummary Control...From Code"
description = "I love the asp.net validation controls. They're chick and really useful. We sometimes have to show s ..."
tags = [ "ASP.NET", "blog" ]
date = "2008-12-11 19:49:00"
slug = "Putting-Messages-Into-a-ValidationSummary-ControlFrom-Code"
+++
<p>I love the asp.net validation controls. They're chick and really useful. We sometimes have to show some messages that are not the ErrorMessages of validators. An approach to do this could be to</p>
<p>a) ScriptManager.RegisterClientScriptBlock to show a popup (YUCK!!)</p>
<p>b) Put in a label and set its Text from code. This could be an option, only the error message will be in the label and not inside the ValidationSummary. This does not look good as some errors will be shown neatly in a ValidationSummary whereas others will eb shown in a separate label.</p>
<p>c) This is my favourite. Here's what you do:</p>
<p>In code behind, when you want to add a message to the ValidationSummary, just do this:</p>
<p><em>RequiredFieldValidator req = new RequiredFieldValidator(); <br />req.ValidationGroup = "vgInput"; <br />req.ErrorMessage = "Your message goes here."; <br /><strong>req.IsValid = false; <br />Page.Form.Controls.Add(req); <br />req.Visible = false;</strong></em></p>
<p>Notice the last three lines. One causes page validation to fail, the next adds the validator to the page's controls so it can have effect (validators must be inside the form tag). The last hides the validator so the error message is shown only in the ValidationSummary and not at the location of the validator (which is at the end of the form as it's added dynamically.</p>
<p>Hope that helps.</p>
        