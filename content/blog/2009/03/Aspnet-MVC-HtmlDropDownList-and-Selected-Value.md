
+++
title = "Asp.net MVC, Html.DropDownList and Selected Value"
description = "I recently ran into the altogether common problem of the Html.DropDownList helper rendering a drop d ..."
tags = [ "ASP.NET", "ASP.NET MVC", "blog" ]
date = "2009-03-27 03:53:00"
slug = "Aspnet-MVC-HtmlDropDownList-and-Selected-Value"
+++
<p>I recently ran into the altogether common problem of the Html.DropDownList helper rendering a drop down list with no value selected. This is a major problem when editing data as by default, the first value is selected and saving would mean the first value is used.</p>
<p>There have been a few issues resulting in the same error. My issue was that I was setting the Name of the drop down list to be equal to the property on my model. I was using the Entity Framework, and had an Image class with a navigation property called Category. I was using this to render the ddl:</p>
<p><em><strong>&lt;%= Html.DropDownList("Category", (IEnumerable&lt;SelectListItem&gt;)ViewData["categories"])%&gt;</strong></em></p>
<p>In my controller, I was setting the ViewData like this:</p>
<p><strong><em>this.ViewData["categories"] = new SelectList(db.CategorySet.ToList(), "CategoryId", "Title", img.CategoryReference.EntityKey);</em></strong></p>
<p>Unfortunately, even though I had set the selected value (third parameter to the SelectList constructor), the ddl had no value selected.</p>
<p>The fix was quite simple:</p>
<p><strong><em>&lt;%= Html.DropDownList("CategoryId", (IEnumerable&lt;SelectListItem&gt;)ViewData["categories"])%&gt;</em></strong></p>
<p>I just changed the Name of the drop down and handled the assignment in the controller.</p>
<p>The reason behind this problem is that asp.net MVC first looks for a match between the name of the drop down and a property on the model. If there&rsquo;s a match, the selected value of the SelectList is overridden. Changing the name of the drop down is all it takes to remedy the issue.</p>
<p>&nbsp;</p>
<p>Hope that helps.</p>
        