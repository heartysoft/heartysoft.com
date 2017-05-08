
+++
title = "ASP.NET MVC: ModelBinding Multiple File Uploads to an Array"
description = "Web apps often need to upload files. ModelBinding a file upload to an HttpPostedFileBase action para ..."
tags = [ "ASP.NET", "ASP.NET MVC", "blog" ]
date = "2010-05-06 19:46:00"
slug = "ASPNET-MVC-ModelBinding-Multiple-File-Uploads-to-an-Array"
+++
<p>Web apps often need to upload files. ModelBinding a file upload to an HttpPostedFileBase action parameter is straightforward as long as you know the exact name of the file upload html control. There may be some situations where there are quite a few file upload controls and for whatever reason, they have very different names (they could be autogenerated on the page for example). In such a case, it becomes difficult to know what names to give to the action parameters at compile time. The usual option would be to hook into the Request parameters. That works, but is not very testable (without a good mocking framework like SharpMock, Moles, TypeMock Isolator etc.). In such situations, we can use a model binder to bind all file uploads to a single upload parameter.</p>
<p><strong>The Container</strong></p>
<p>The first thing to do is to create the class to hold the information relating to the uploaded files:</p>
<p><pre class='brush:c#'> 

public class UploadedFileInfo 
{ 
&nbsp;&nbsp;&nbsp; public string Name { get; private set; } 
&nbsp;&nbsp;&nbsp; public HttpPostedFileBase File { get; private set; } 

&nbsp;&nbsp;&nbsp; public UploadedFileInfo(string name, HttpPostedFileBase file) 
&nbsp;&nbsp;&nbsp; { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Name = name; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; File = file; 
&nbsp;&nbsp;&nbsp; }&nbsp;&nbsp; 
} 

</pre></p>
<p>The class is pretty simple. It holds the name of the file upload control that uploaded a file and the uploaded file itself.</p>
<p><strong>The ModelBinder</strong></p>
<p>Next comes the ModelBinder:</p>
<p><pre class='brush:c#'> 

public class UploadedFileInfoArrayBinder : IModelBinder 
{ 
&nbsp;&nbsp;&nbsp; public object BindModel(ControllerContext controllerContext, ModelBindingContext bindingContext) 
&nbsp;&nbsp;&nbsp; { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var files = controllerContext.HttpContext.Request.Files; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var list = new List&lt;UploadedFileInfo&gt;(); 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; for (int i = 0; i &lt; files.Count; i++) 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var file = files[i]; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var name = files.AllKeys[i];&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var fileInfo = new UploadedFileInfo(name, file);&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; list.Add(fileInfo); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return list.ToArray(); 
&nbsp;&nbsp;&nbsp; } 
} 

</pre></p>
<p>Basically, the binder looks into Request.Files and for each file present, it creates and instance of UploadedFileInfo and adds it to a list. It simply returns the list as an array. Please note that if a user does not select a file for an upload control, it will still have an entry in Request.Files, only the uploaded file's ContentLength would be zero.</p>
<p><strong>The Controller</strong></p>
<p>And the controller:</p>
<p><pre class='brush:c#'> 

public class UploadTestController : Controller 
{ 
&nbsp;&nbsp;&nbsp; [HttpGet] 
&nbsp;&nbsp;&nbsp; public ActionResult Index() 
&nbsp;&nbsp;&nbsp; { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return View(); 
&nbsp;&nbsp;&nbsp; } 

&nbsp;&nbsp;&nbsp; [HttpPost] 
&nbsp;&nbsp;&nbsp; public ActionResult Index(UploadedFileInfo[] files) 
&nbsp;&nbsp;&nbsp; { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var uploadedCount = files.Where(x =&gt; x.File.ContentLength &gt; 0).Count(); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ViewData["Message"] = string.Format("{0} file(s) uploaded successfully", uploadedCount); 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return View(); 
&nbsp;&nbsp;&nbsp; } 
} 

</pre></p>
<p>The GET Index() is trivial. The Post Index() fetches the number of files in the model-bound files array that have a non-zero content length and passes the info to the View.</p>
<p><strong>The View</strong></p>
<p>And the view:</p>
<p><pre class='brush:xml'> 

&lt;body&gt; 
&nbsp;&nbsp;&nbsp; &lt;div&gt; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;form enctype="multipart/form-data" method="post"&gt; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;input type='file' name='file1' id='file1' /&gt; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;input type='file' name='file2' id='file2' /&gt; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;input type="submit" value='upload' /&gt; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;/form&gt; 
&nbsp;&nbsp;&nbsp; &lt;/div&gt; 
&nbsp;&nbsp;&nbsp; &lt;br /&gt; 
&nbsp;&nbsp;&nbsp; &lt;%: ViewData["Message"] %&gt; 
&lt;/body&gt; 

</pre></p>
<p>The View is the same for the GET and the POST (don't mind the fact that I should be redirecting after the post &ndash; this is a demo after all!). There's a form that has two file uploads in a form. The file uploads have different names. The View also spits ViewData["Message"] if present.</p>
<p><strong>Assigning the ModelBinder in Global.asax.cs</strong></p>
<p>At this point, our code still won't work as the ModelBinder is not being applied to the files parameter of the POST Index(). We can add an attribute to the parameter, but there's a cleaner, nicer way to set it up in the global.asax.cs file:</p>
<p><pre class='brush:c#'> 

protected void Application_Start() 
{ 
&nbsp;&nbsp;&nbsp; AreaRegistration.RegisterAllAreas(); 
&nbsp;&nbsp;&nbsp; RegisterRoutes(RouteTable.Routes); 
&nbsp;&nbsp;&nbsp; ModelBinders.Binders[typeof(UploadedFileInfo[])] = new UploadedFileInfoArrayBinder(); 
} 

</pre></p>
<p>In the last line, we're basically telling MVC that whenever it encounters a parameter to an ActionMethod that is an array of UploadedFileInfo, it should use an UploadedFileInfoArrayBinder. The binder will then fetch the files from the Request and assign it to the parameter.</p>
<p><strong>Running the App</strong></p>
<p>With all this in place, we can run the app, select a couple of files and hitting upload should tell us that the files have been uploaded.</p>
<p><strong>Source</strong></p>
<p><a href="http://cid-9d8a7728356393b1.skydrive.live.com/self.aspx/.Public/code/MultipleFileUploadsMVC.zip">download</a></p>
<p><em>---------------------------------------------------------</em></p>
<p><em><span style="color: #ff0000;">Questions&nbsp; and comments relating to this article are welcome. Comments completely unrelated to the article and posted with the sole intention of putting your link here are not. <br /></span></em><em><span style="color: #ff0000;"><br />If you spam, your comment will not be approved, will be deleted and your IP blocked. I maintain my site almost daily and such comments &ndash; even if they pass the spam filter &ndash; will get removed as soon as possible. If this gets too tedious, I may disable comments entirely. Please don't ruin it for everybody else.</span></em></p>
<p><em>---------------------------------------------------------</em></p>
        