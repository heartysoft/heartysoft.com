
+++
title = "ASP.NET MVC Gotcha – String as Model"
description = "A very common task in ASP.NET MVC is to return a ViewResult from a controller Action. This is usuall ..."
tags = [ ".NET", "ASP.NET", "ASP.NET MVC", "blog" ]
date = "2011-04-19 04:25:00"
slug = "aspnet-mvc-gotcha-string-model"
+++
<p>A very common task in ASP.NET MVC is to return a ViewResult from a controller Action. This is usually achieved by a simple:</p>  <p><pre class='brush:c#'>   
return View(“ViewName”, modelForTheView);    
</pre></p>  <p>And this works fine…usually. Have you ever had a view where the model is simply a string? In that case, your code may look like this:</p>  <p><pre class='brush:c#'>   
var model = “some string”;    
return View(“ViewName”, model);    
</pre></p>  <p>Only in this case, it won’t work. Don’t believe me? Try it. It’s going to show an obscure error message saying that it could not find the view or master / layout page for ViewName.cshtml [or aspx if you’re using that inferior view engine]. Yet if the model is anything but a string, it works perfectly. This can result in confusion, anger, despair, hair-pulling, keyboard pounding and more importantly wasted time. And if you ever find yourself doing any of those things, then the reason for the error will annoy you even further. You see, Controll.View() has a number of overloads. Two of them are as follows:</p>  <p><pre class='brush:c#'>   
View(String, Object)    
View(String, String)    
</pre></p>  <p>And yes, they are different. The first one expects the model as the second parameter. The second one expects the MASTER/LAYOUT name as the second parameter. When you pass anything but a string as the second parameter, the first overload is triggered. When you pass a string, .NET sees that it matches the second overload and that match is “stronger” than the first. As a result, it calls the second overload. As a result, it will look for a master / layout with the same name as the data you’re passing to it and chances are, it won’t find it. And you’ll get that confusing error.</p>  <h2>The Solution</h2>  <p>The solution is quite simple. You could use the other overload:</p>  <p><pre class='brush:c#'>   
View(String, String, Object)    
</pre></p>  <p>where the second parameter is the master / layout name and the third is the actual model whenever you have a View expecting a string in the model. Or you could use a named parameter:</p>  <p><pre class='brush:c#'>   
return View(“ViewName”, model: “some string”);    
</pre></p>  <p>Or simply use a separate ViewModel class.</p>  <p><em></em></p>  <p><em>The same principle applies to</em></p>  <p><em><pre class='brush:c#'>     
View(Object)      
View(String)      
</pre></em></p>  <p><em>i.e. If you pass anything but a string as the first parameter, it’s considered the model and the view name is inferred from the executing action. If you pass only&#160; a string, it’s taken to be the view name and the model is null.</em></p>  <p>&#160;</p>  <h2>Why This Annoys Me</h2>  <p>With due respect, this design is simply bonkers to me. It seems like the MVC team wanted to add some flexibility but in the process, they introduced an open invitation for off by one errors. Suddenly the type of the model is dictating how the View method responds. Simply the fact that a call to:</p>  <p><pre class='brush:c#'>   
return View(“Index”, model)    
</pre></p>  <p>can have two radically different consequences depending on what’s in the model variable is mindboggling. Besides, what benefit is there from having an overload of the View method that sets the master / layout? The same could easily be achieved by simply putting the master / layout name in Viewbag / ViewData and simply assigning it in the view. Assigning the master / layout in the view is not a good practice in general and should not be encouraged by having an overload of the View method which does exactly that while taking precedence over another overload that is far more commonly used [i.e. second parameter’s the model].</p>  <p>That said, for good or for bad, the annoying overload is there in the framework. So if you have a string as the “model”,&#160; be aware that what you think is the model may actually not be interpreted as such. </p>
        