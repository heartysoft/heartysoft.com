
+++
title = "Exposing a WCF Service for Ajax and Silverlight"
description = "One of the most attractive features of WCF is that you get to write a service once and expose it to  ..."
tags = [ ".NET", "AJAX", "AJAX Control Toolkit", "ASP.NET", "Silverlight", "jQuery", "WCF", "blog" ]
date = "2010-02-22 09:44:58"
slug = "exposing-wcf-ajax-silverlight"
+++
<p>One of the most attractive features of WCF is that you get to write a service once and expose it to be used by multiple clients using different technologies and protocols. How a WCF service is exposed can be changed simply by changing the configuration file, without needing to recompile the code. In this article, we're going to create a WCF service and expose it to Ajax. We'll test the Ajax service by calling it from ASP.NET Ajax 4.0 and jQuery. We'll then expose the service to Silverlight through binary encoding, which will increase data transfer efficiency. Finally, we'll create a small Silverlight 3.0 application that will call the service.</p>  <p><em><font color="#008000">Note: I'm using VS 2010 RC, ASP.NET Ajax 4.0, jQuery 1.4.2 and Silverlight 3.0. Everything WCF related should work similarly for other versions, but some changes may be necessary. Features specific to a version (for example client templates in ASP.NET Ajax 4.0) will obviously not work in previous versions.</font></em></p>  <h3>The Service</h3>  <p>Open up VS 2010 and create a new project. I'm using the ASP.NET Empty Web Application template. Name the project WCFEndpoints.</p>  <p><a href="/media/default/images/NewProject_1.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="NewProject" border="0" alt="NewProject" src="/media/default/images/NewProject_thumb_1.png" width="707" height="491" /></a> </p>  <p>This gives us an empty ASP.NET application with the following (very simple) web.config:</p>  <p><pre class='brush:xml'>   
&lt;?xml version=&quot;1.0&quot;?&gt; 
  &lt;!--    
&#160; For more information on how to configure your ASP.NET application, please visit     
<a href="http://go.microsoft.com/fwlink/?LinkId=169433">http://go.microsoft.com/fwlink/?LinkId=169433</a>     
&#160; --&gt; 
  &lt;configuration&gt;    
&#160;&#160;&#160; &lt;system.web&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;compilation debug=&quot;true&quot; targetFramework=&quot;4.0&quot; /&gt;     
&#160;&#160;&#160; &lt;/system.web&gt; 
  &lt;/configuration&gt;   
    
</pre></p>  <p>Add a folder called Model to the project and add the following two classes:</p>  <p><pre class='brush:c#'>   
using System;     
    
namespace WCFEndpoints.Model     
{     
&#160;&#160;&#160; public class Book     
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; public int BookId { get; set; }     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; public string Title { get; set; }     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; public string Author { get; set; }     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; public decimal Price { get; set; }     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; public DateTime PublishDate { get; set; }     
&#160;&#160;&#160; }     
}    
    
</pre></p>  <p>&#160;</p>  <p><pre class='brush:c#'>   
using System;     
using System.Collections.Generic;     
using System.Linq;     
    
namespace WCFEndpoints.Model     
{     
&#160;&#160;&#160; public class BookService     
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; public IEnumerable&lt;Book&gt; GetBooks()     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; yield return new Book { BookId = 1, Title = &quot;LOTR&quot;, Author = &quot;Tolkien&quot;, Price = 180.00M, PublishDate = new DateTime(1954, 1, 1) };     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; yield return new Book { BookId = 2, Title = &quot;C#&quot;, Author = &quot;Hejlsberg&quot;, Price = 280.00M, PublishDate = new DateTime(2003, 1, 1) };     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; yield return new Book { BookId = 3, Title = &quot;DDD&quot;, Author = &quot;Evans&quot;, Price = 200.00M, PublishDate = new DateTime(2003, 8, 30) };     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; yield return new Book { BookId = 4, Title = &quot;Architecture&quot;, Author = &quot;Fowler&quot;, Price = 250.00M, PublishDate = new DateTime(2002, 11, 15) };     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; } 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; public IEnumerable&lt;Book&gt; GetBooksPublishedAfter(DateTime date)    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; return GetBooks().Where(x =&gt; x.PublishDate &gt; date);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; }     
&#160;&#160;&#160; }     
}    
    
</pre></p>  <p><em><font color="#008000">Note: I'm not exactly following best practices in terms of the model. I just wanted to keep that part simple as it's not the focus of the article.</font></em></p>  <p>Right click the WCFEndpoints project (not solution) in the solution explorer and select &quot;Add New Item&quot;. From there, add a new &quot;AJAX-enabled WCF Service&quot; called LibraryService.</p>  <p><a href="/media/default/images/AddService_1.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="AddService" border="0" alt="AddService" src="/media/default/images/AddService_thumb_1.png" width="527" height="361" /></a> </p>  <p>&#160;</p>  <p>Open the newly generated LibraryService.svc.cs file and modify it so that it looks like this:</p>  <p><pre class='brush:c#'>   
using System;     
using System.Collections.Generic;     
using System.Linq;     
using System.ServiceModel;     
using System.ServiceModel.Activation;     
using WCFEndpoints.Model;     
    
namespace WCFEndpoints     
{     
&#160;&#160;&#160; [ServiceContract(Namespace = &quot;Library&quot;)]     
&#160;&#160;&#160; [AspNetCompatibilityRequirements(RequirementsMode = AspNetCompatibilityRequirementsMode.Allowed)]     
&#160;&#160;&#160; public class LibraryService     
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; BookService svc = new BookService(); 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; [OperationContract]    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; public List&lt;Book&gt; GetAllBooks()     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; return svc.GetBooks().ToList();     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; } 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; [OperationContract]    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; public List&lt;Book&gt; GetBooksPublishedAfter(DateTime date)     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; return svc.GetBooksPublishedAfter(date).ToList();     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; }     
&#160;&#160;&#160; }     
}    
    
</pre></p>  <p>That's all we are going to write for the service. The service is currently usable from an Ajax client and we'll only need to change the configuration in web.config to expose it to Silverlight. Let's take a quick look at the auto generated WCF specific section the VS created for us when adding the service:</p>  <p><pre class='brush:xml'>   
&lt;configuration&gt;     
&#160;&#160;&#160; &lt;system.web&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;compilation debug=&quot;true&quot; targetFramework=&quot;4.0&quot; /&gt;     
&#160;&#160;&#160; &lt;/system.web&gt; 
  &#160;&#160;&#160; &lt;system.serviceModel&gt;    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;behaviors&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;endpointBehaviors&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;behavior name=&quot;WCFEndpoints.LibraryServiceAspNetAjaxBehavior&quot;&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;enableWebScript /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/behavior&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/endpointBehaviors&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/behaviors&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;serviceHostingEnvironment aspNetCompatibilityEnabled=&quot;true&quot;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; multipleSiteBindingsEnabled=&quot;true&quot; /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;services&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;service name=&quot;WCFEndpoints.LibraryService&quot;&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;endpoint address=&quot;&quot; behaviorConfiguration=&quot;WCFEndpoints.LibraryServiceAspNetAjaxBehavior&quot;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; binding=&quot;webHttpBinding&quot; contract=&quot;WCFEndpoints.LibraryService&quot; /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/service&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/services&gt;     
&#160;&#160;&#160; &lt;/system.serviceModel&gt;     
&lt;/configuration&gt;    
    
</pre></p>  <p>The WCF sepcific section is the part inside the &lt;system.serviceModel&gt; tags. The first change we're going to do is to rename &quot;WCFEndpoints.LibraryServiceAspNetAjaxBehavior&quot; to &quot;AjaxBehavior&quot;. Just make sure you change both instances. Next, we're going to add a service behaviour and assign it to our service. The service behaviour we're adding will allow us to publish the service metadata via HTTP GET. We will also change the address of the endpoint to &quot;ajax&quot; (I will explain this later).&#160; After these changes, the &lt;system.serviceModel&gt; section will look like this:</p>  <p><pre class='brush:xml'>   
&lt;system.serviceModel&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;behaviors&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;endpointBehaviors&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;behavior name=&quot;AjaxBehavior&quot;&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;enableWebScript /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/behavior&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/endpointBehaviors&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;serviceBehaviors&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;behavior name=&quot;metadataGetBehavior&quot;&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;serviceMetadata httpGetEnabled=&quot;true&quot;/&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;serviceDebug includeExceptionDetailInFaults=&quot;true&quot;&gt;&lt;/serviceDebug&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/behavior&gt;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/serviceBehaviors&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/behaviors&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;serviceHostingEnvironment aspNetCompatibilityEnabled=&quot;true&quot;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; multipleSiteBindingsEnabled=&quot;true&quot; /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;services&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;service name=&quot;WCFEndpoints.LibraryService&quot; behaviorConfiguration=&quot;metadataGetBehavior&quot;&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;endpoint address=&quot;ajax&quot; behaviorConfiguration=&quot;AjaxBehavior&quot;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; binding=&quot;webHttpBinding&quot; contract=&quot;WCFEndpoints.LibraryService&quot; /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/service&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/services&gt;     
&#160;&#160;&#160; &lt;/system.serviceModel&gt;    
    
</pre></p>  <p>Save the config file, right click the LibraryService.svc file in solution explorer and select &quot;View in Browser&quot;. If everything went ok, you should see something like this:</p>  <p><a href="/media/default/images/browserTest_1.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="browserTest" border="0" alt="browserTest" src="/media/default/images/browserTest_thumb_1.png" width="638" height="347" /></a> </p>  <h3>&#160;</h3>  <h3>Ajax Test 1: ASP.NET Ajax</h3>  <p>I'll be using the latest release of ASP.NET Ajax (currently 0911 beta). If you don't have it, download it from <a href="http://ajax.codeplex.com/">http://ajax.codeplex.com/</a>, extract the zip, create a new folder called Scripts in the main project, add a subfolder called MSAjax and paste in everything inside the Scripts folder of the extracted zip to the MSAjax folder. Add a new htm file called Index.htm to the project. I'm using a plain htm file to show the pure client side aspects of ASP.NET Ajax. First, we're going to create a client side template:</p>  <p><pre class='brush:xml'>   
&lt;fieldset&gt;     
&#160;&#160;&#160; &lt;legend&gt;All Books&lt;/legend&gt;     
&#160;&#160;&#160; &lt;div&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;ul id='books-template' class='sys-template'&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;li&gt;{{Title}} - {{Author}}, {{ String.format(&quot;${0:n}&quot;, Price) }}, {{ PublishDate.format(&quot;dd-MMM-yyyy&quot;)     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }} &lt;/li&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/ul&gt;     
&#160;&#160;&#160; &lt;/div&gt;     
&lt;/fieldset&gt;    
    
</pre></p>  <p>Next, add the following style declaration:</p>  <p><pre class='brush:xml'>   
&lt;style type=&quot;text/css&quot;&gt;     
&#160;&#160;&#160;&#160;&#160;&#160; .sys-template, .hidden     
&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; display: none;     
&#160;&#160;&#160;&#160;&#160;&#160; }     
&lt;/style&gt;    
    
</pre></p>  <p>Add a reference to Start.js:</p>  <p><pre class='brush:xml'>   
&lt;script src=&quot;/Scripts/MSAjax/Start.js&quot; type=&quot;text/javascript&quot;&gt;&lt;/script&gt;    
    
</pre></p>  <p>Add the following javascript in a &lt;script&gt; tag inside the &lt;head&gt;:</p>  <p>[code:js]   <br />Sys.require([Sys.scripts.WebServices, Sys.components.dataView, Sys.scripts.Globalization], function () {     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Sys.create.dataView(&quot;#books-template&quot;, {     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'dataProvider': '/LibraryService.svc/ajax',     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'fetchOperation': 'GetAllBooks',     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'autoFetch': true     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });     <br />&#160;&#160;&#160;&#160;&#160;&#160; });    <br />    <br /></pre></p>  <p>What this piece of code is doing is it's loading the scripts necessary for calling web services, using the dataview and formatting asynchronously and when all are loaded, it's creating a dataview that calls out to our service's GetAllBooks method. When it gets back the result, it uses our template to add a &lt;li&gt; for each Book returned. Notice that for the dataProvider property, I specify '/LibraryService.svc/<font color="#008000">ajax</font><font color="#000000">'. We specified &quot;ajax&quot; as the address for our service's Ajax endpoint in web.config – this is where that comes in. Had we kept it empty, we would have set the dataProvider to '/LibraryService.svc'. Since our endpoint's address is &quot;ajax&quot;, we're specifying 'LibraryService.svc/ajax'. The reason we're doing this is that later on we'll add another endpoint that will use binary encoding to be consumed by Silverlight 3. </font></p>  <p>At this point, if you run the page, you should see something like this:</p>  <p><a href="/media/default/images/results1_1.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="results1" border="0" alt="results1" src="/media/default/images/results1_thumb_1.png" width="438" height="179" /></a> </p>  <p>Look at that – less than 10 lines of html and very little javascript and you're displaying a formatted list of results got from a web service call. And you had to do zero html building in javascript. Splendid :)</p>  <p>Let's take this one step further and add a calendar which would allow the user to select a date and a button that, when clicked, will call the 'GetBooksPublishedAfter' method of our web service and display only the books that were published after the selected date. Add the following html right after the closing &lt;/fieldset&gt; tag:</p>  <p><pre class='brush:xml'>   
&lt;br /&gt;     
&#160;&#160; &lt;fieldset&gt;     
&#160;&#160;&#160;&#160;&#160;&#160; &lt;legend&gt;Books Published After:&lt;/legend&gt;     
&#160;&#160;&#160;&#160;&#160;&#160; &lt;div&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Please select a date:     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;input type='text' id='txtDate' /&gt;&lt;br /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;input type='button' id='btnFilter' value='fetch books' /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160; &lt;/div&gt;     
&#160;&#160;&#160;&#160;&#160;&#160; &lt;div&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;ul id='filtered-books'&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;li class='hidden'&gt;&lt;/li&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/ul&gt;     
&#160;&#160;&#160;&#160;&#160;&#160; &lt;/div&gt;     
&#160;&#160; &lt;/fieldset&gt;    
    
</pre></p>  <p>We want to attach a calendar, which is part of the Ajax Control Toolkit, to the textbox. In order to do so, we need to ensure that the script loader can load the calendar script on demand and that the css for the calendar is loaded. Add the following two lines right after the registration of Start.js:</p>  <p><pre class='brush:xml'>   
&lt;script src=&quot;/Scripts/MSAjax/extended/ExtendedControls.js&quot; type=&quot;text/javascript&quot;&gt;&lt;/script&gt;     
&lt;link href=&quot;/Scripts/MSAjax/extended/Calendar/Calendar.css&quot; rel=&quot;stylesheet&quot; type=&quot;text/css&quot; /&gt;    
    
</pre></p>  <p>The Start.js file has logic to load scripts in parallel and on-the-fly as well as data about the core Ajax Library scripts and jQuery. It does not contain data about the toolkit controls. This data is present in ExtendedControls.js. Unfortunately, the script loader currently cannot load css at runtime. It is a feature the ASP.NET Ajax team are looking into. For the time being, we'll need to specify the css file explicitly. </p>  <p>To attach a calendar to the textbox, add the following javascript:</p>  <p>[code:js]   <br />Sys.require([Sys.components.calendar], function () {     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var selectedDate = Date.parseInvariant('1-1-2000', 'd-M-yyyy');     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Sys.create.calendar('#txtDate', {     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'format': 'dd-MM-yyyy',     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'selectedDate': selectedDate,     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'id': 'cal-txtDate'     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });     <br />&#160;&#160;&#160;&#160;&#160;&#160; });    <br />    <br /></pre></p>  <p>Finally, we need to bind some code to the button's click handler so that when it's clicked, our web service will be called and the UI updated:</p>  <p>[code:js]   <br />Sys.addHandler('#btnFilter', 'click', function () {     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var date = $find('cal-txtDate').get_selectedDate();     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Sys.create.dataView('#filtered-books', {     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'dataProvider': '/LibraryService.svc/ajax',     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'fetchOperation': 'GetBooksPublishedAfter',     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'fetchParameters': { 'date': date },     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'itemTemplate': '#books-template',     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'autoFetch': true     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });    <br />    <br /></pre></p>  <p>Notice that we're using our previously declared template and passing it in via the itemTemplate parameter. This is quite nice (and DRY) and ensures that we don't have to repeat the template just because the data changes. Since the code is getting slightly complex, here's the whole page for your convenience:</p>  <p><pre class='brush:xml'>   
&lt;!DOCTYPE html PUBLIC &quot;-//W3C//DTD XHTML 1.0 Transitional//EN&quot; &quot;<a href="http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd&quot;">http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd&quot;</a>&gt;     
&lt;html xmlns=&quot;<a href="http://www.w3.org/1999/xhtml&quot;">http://www.w3.org/1999/xhtml&quot;</a>&gt;     
&lt;head&gt;     
&#160;&#160;&#160; &lt;title&gt;&lt;/title&gt;     
&#160;&#160;&#160; &lt;style type=&quot;text/css&quot;&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; .sys-template, .hidden     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; display: none;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; }     
&#160;&#160;&#160; &lt;/style&gt;     
&#160;&#160;&#160; &lt;script src=&quot;/Scripts/MSAjax/Start.js&quot; type=&quot;text/javascript&quot;&gt;&lt;/script&gt;     
&#160;&#160;&#160; &lt;script src=&quot;/Scripts/MSAjax/extended/ExtendedControls.js&quot; type=&quot;text/javascript&quot;&gt;&lt;/script&gt;     
&#160;&#160;&#160; &lt;link href=&quot;/Scripts/MSAjax/extended/Calendar/Calendar.css&quot; rel=&quot;stylesheet&quot; type=&quot;text/css&quot; /&gt;     
&#160;&#160;&#160; &lt;script type='text/javascript'&gt; 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; //initialize the first dataview    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; Sys.require([Sys.scripts.WebServices, Sys.components.dataView, Sys.scripts.Globalization], function () {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Sys.create.dataView(&quot;#books-template&quot;, {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'dataProvider': '/LibraryService.svc/ajax',     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'fetchOperation': 'GetAllBooks',     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'autoFetch': true     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; }); 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; //initialize the calender and button handler    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; Sys.require([Sys.components.calendar], function () {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var selectedDate = Date.parseInvariant('1-1-2000', 'd-M-yyyy');     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Sys.create.calendar('#txtDate', {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'format': 'dd-MM-yyyy',     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'selectedDate': selectedDate,     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'id': 'cal-txtDate'     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }); 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; //add the handler, call the service when the button is clicked and update the UI.    
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Sys.addHandler('#btnFilter', 'click', function () {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var date = $find('cal-txtDate').get_selectedDate();     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Sys.create.dataView('#filtered-books', {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'dataProvider': '/LibraryService.svc/ajax',     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'fetchOperation': 'GetBooksPublishedAfter',     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'fetchParameters': { 'date': date },     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'itemTemplate': '#books-template',     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'autoFetch': true     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; }); 
  &#160;&#160;&#160; &lt;/script&gt;    
&lt;/head&gt;     
&lt;body&gt;     
&#160;&#160;&#160; &lt;fieldset&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;legend&gt;All Books&lt;/legend&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;div&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;ul id='books-template' class='sys-template'&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;li&gt;{{Title}} - {{Author}}, {{ String.format(&quot;${0:n}&quot;, Price) }}, {{ PublishDate.format(&quot;dd-MMM-yyyy&quot;)     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }} &lt;/li&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/ul&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/div&gt;     
&#160;&#160;&#160; &lt;/fieldset&gt;     
&#160;&#160;&#160; &lt;br /&gt;     
&#160;&#160;&#160; &lt;fieldset&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;legend&gt;Books Published After:&lt;/legend&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;div&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Please select a date:     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;input type='text' id='txtDate' /&gt;&lt;br /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;input type='button' id='btnFilter' value='fetch books' /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/div&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;div&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;ul id='filtered-books'&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;li class='hidden'&gt;&lt;/li&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/ul&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/div&gt;     
&#160;&#160;&#160; &lt;/fieldset&gt;     
&lt;/body&gt;     
&lt;/html&gt;    
    
</pre></p>  <p>Running the page, you should see the textbox initialized to 1-1-2000. Clicking in the textbox, you should see a nice little popup calendar which lets you select a date. Clicking on the button will show you the results of the filtered query. This should look something like this:</p>  <p><a href="/media/default/images/results2_1.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="results2" border="0" alt="results2" src="/media/default/images/results2_thumb_1.png" width="418" height="328" /></a> </p>  <p>&#160;</p>  <h3>Ajax Test 2: jQuery</h3>  <p>Add a new page for the jQuery test. Call it Index2.htm. Add the following html:</p>  <p><pre class='brush:xml'>   
&lt;fieldset&gt;     
&#160;&#160;&#160;&#160;&#160;&#160; &lt;legend&gt;All Books&lt;/legend&gt;     
&#160;&#160;&#160;&#160;&#160;&#160; &lt;div id='all-books'&gt;&lt;/div&gt;     
&lt;/fieldset&gt;    
    
</pre></p>  <p>We'll also add the script loader in ASP.NET Ajax, so add the following script reference:</p>  <p><pre class='brush:xml'>   
&lt;script src=&quot;/Scripts/MSAjax/Start.js&quot; type=&quot;text/javascript&quot;&gt;&lt;/script&gt;    
    
</pre></p>  <p>Download the latest version of jQuery (currently 1.4.2) from <a href="http://www.jquery.com">www.jquery.com</a> and add it to the Scripts folder. With that done, add the following javascript:</p>  <p>[code:js]   <br />Sys.loadScripts(['/Scripts/jquery-1.4.2.js'], function () { </p>  <p>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $(function () {    <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $.post('/LibraryService.svc/ajax/GetAllBooks', null, function (res) {     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Sys.require([Sys.scripts.Serialization, Sys.scripts.Globalization], function () {     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; res = Sys.Serialization.JavaScriptSerializer.deserialize(res);     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var data = res.d; //data is in d     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var list = $('&lt;ul&gt;&lt;/ul&gt;'); </p>  <p>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; for (var i = 0; i &lt; data.length; i++) {    <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; list.append('&lt;li&gt;' + data[i].Title + '- ' + data[i].Author + ', $' + data[i].Price.format('n') + ', ' + data[i].PublishDate.format('dd-MMM-yyyy'));     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $('#all-books').empty().html(list);     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }, 'text');     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }); </p>  <p>&#160;&#160;&#160;&#160;&#160;&#160; });   <br />    <br /></pre></p>  <p>I'm loading jQuery via the script loader and making a $.post to the web service. I'm using POST as that's what WCF supports out of the box for web service calls. In the callback function, I'm cheating slightly as I'm using the ASP.NET Ajax JavaScriptSerializer to deserialize the response. There are other ways of deserializing the response (check out <a href="http://www.west-wind.com/weblog/posts/324917.aspx">http://www.west-wind.com/weblog/posts/324917.aspx</a>), but using the ASP.NET Ajax deserializer is so simple and easy, I decided to use that one. Do we need to deserialize all the time? Well, no. You would need it when returning any DateTime object from WCF. JSON doesn't have a native way to express dates. ASP.NET Ajax has a way of serializing dates to strings so that they can be deserialized to a javascript Date object if the serializer being used knows about it. And the Sys.Serialization.JavaScriptSerializer does know about it. I'm also referencing Sys.scripts.Globalization to be able to format the price and publish date information.</p>  <p>Another thing to notice is that in the $.post call, I'm mentioning 'text' as the type of data returned. The reason I'm specifying 'text' and not 'json' is that we're deserializing the response right after getting the response. If we had specified 'json', we would be trying to deserialize a javascript object, which doesn't make sense. </p>  <p>Lastly, after deserializing the result, we're only working with the result's &quot;d&quot; member. The reason for this is that WCF envelopes all returned data into a variable called &quot;d&quot;. So, we get our books array in res.d and not res. The &quot;d&quot; envelope is used to prevent some javascript exploits.</p>  <p>If we run the page now, we should see this:</p>  <p><a href="/media/default/images/results3_1.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="results3" border="0" alt="results3" src="/media/default/images/results3_thumb_1.png" width="482" height="195" /></a> </p>  <p>&#160;</p>  <p>For the next part, add the following html after the closing &lt;/fieldset&gt; tag:</p>  <p><pre class='brush:xml'>   
&lt;fieldset&gt;     
&#160;&#160;&#160;&#160;&#160;&#160; &lt;legend&gt;Filtered Books&lt;/legend&gt;     
&#160;&#160;&#160;&#160;&#160;&#160; &lt;div&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Please select a date:     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;input type='text' id='txtDate' /&gt;&lt;br /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;input type='button' id='btnFilter' value='fetch books' /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160; &lt;/div&gt;     
&#160;&#160;&#160;&#160;&#160;&#160; &lt;div id='filtered-books&gt;&lt;/div&gt;     
&lt;/fieldset&gt;    
    
</pre></p>  <p>Add the calender to the textbox just like we did before. Since we're already using jQuery, this becomes even easier. Add the reference to ExtenderControls.js and the Calendar.css:</p>  <p><pre class='brush:xml'>   
&lt;script src=&quot;/Scripts/MSAjax/extended/ExtendedControls.js&quot; type=&quot;text/javascript&quot;&gt;&lt;/script&gt;     
&lt;link href=&quot;/Scripts/MSAjax/extended/Calendar/Calendar.css&quot; rel=&quot;stylesheet&quot; type=&quot;text/css&quot; /&gt;    
    
</pre></p>  <p>The following javascript will then attach a calendar to the textbox:</p>  <p>[code:js]   <br />Sys.require([Sys.components.calendar], function () {     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var selectedDate = Date.parseInvariant('1-1-2000', 'd-M-yyyy');     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $('#txtDate').calendar({     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'format': 'dd-MM-yyyy',     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'selectedDate': selectedDate,     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'id': 'cal-txtDate'     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });    <br />    <br /></pre></p>  <p>We now need to add the event handler so that clicking the button will fire the service call and update the UI:</p>  <p>[code:js]   <br />$('#btnFilter').click(function () {     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var date = $find('cal-txtDate').get_selectedDate();     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var params = { 'date': date} ;     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var serializedParams = Sys.Serialization.JavaScriptSerializer.serialize(params);     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $.ajax({     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'url': '/LibraryService.svc/ajax/GetBooksPublishedAfter',     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'data': serializedParams,     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'type': 'POST',     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'processData': false,     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'contentType': 'application/json',     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'timeout': 10000,     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'dataType': 'text', //not JSON, we'll process     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'success': function (res) {     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Sys.require([Sys.scripts.Serialization, Sys.scripts.Globalization], function () {     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; res = Sys.Serialization.JavaScriptSerializer.deserialize(res);     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var data = res.d; //data is in d     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var list = $('&lt;ul&gt;&lt;/ul&gt;'); </p>  <p>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; for (var i = 0; i &lt; data.length; i++) {    <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; list.append('&lt;li&gt;' + data[i].Title + '- ' + data[i].Author + ', $' + data[i].Price.format('n') + ', ' + data[i].PublishDate.format('dd-MMM-yyyy'));     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $('#filtered-books').empty().html(list);     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; },     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'error': function () {     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; alert('An error occurred.');     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });     <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });    <br />    <br /></pre></p>  <p>Again, I'm serializing the input parameters to the web service call using the ASP.NET Ajax serializer. We need to serialize in cases where we're using dates or of we have deep nested javascript objects. Since we're passing in the parameter as a simple string (i.e. the output of serialization), we need to tell WCF that that's actually a JSON string. For this, we need to use $.ajax and not $.post. With $.ajax, we get to specify the contentType as 'application/json'. Again, we're specifying the dataType as 'text' for the same reason as before. The rest should be self explanatory.</p>  <p>Here's the whole page code at a glance:</p>  <p><pre class='brush:xml'>   
&lt;!DOCTYPE html PUBLIC &quot;-//W3C//DTD XHTML 1.0 Transitional//EN&quot; &quot;<a href="http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd&quot;">http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd&quot;</a>&gt;     
&lt;html xmlns=&quot;<a href="http://www.w3.org/1999/xhtml&quot;">http://www.w3.org/1999/xhtml&quot;</a>&gt;     
&lt;head&gt;     
&#160;&#160;&#160; &lt;title&gt;&lt;/title&gt;     
&#160;&#160;&#160; &lt;script src=&quot;/Scripts/MSAjax/Start.js&quot; type=&quot;text/javascript&quot;&gt;&lt;/script&gt;     
&#160;&#160;&#160; &lt;script type=&quot;text/javascript&quot; src=&quot;/Scripts/MSAjax/extended/ExtendedControls.js&quot;&gt;&lt;/script&gt;     
&#160;&#160;&#160; &lt;link href=&quot;/Scripts/MSAjax/extended/Calendar/Calendar.css&quot; rel=&quot;stylesheet&quot; type=&quot;text/css&quot; /&gt;     
&#160;&#160;&#160; &lt;script type='text/javascript'&gt; 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; Sys.loadScripts(['/Scripts/jquery-1.4.2.js'], function () {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $(function () {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $.post('/LibraryService.svc/ajax/GetAllBooks', null, function (res) {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Sys.require([Sys.scripts.Serialization, Sys.scripts.Globalization], function () {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; res = Sys.Serialization.JavaScriptSerializer.deserialize(res);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var data = res.d; //data is in d     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var list = $('&lt;ul&gt;&lt;/ul&gt;'); 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; for (var i = 0; i &lt; data.length; i++) {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; list.append('&lt;li&gt;' + data[i].Title + '- ' + data[i].Author + ', $' + data[i].Price.format('n') + ', ' + data[i].PublishDate.format('dd-MMM-yyyy'));     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $('#all-books').empty().html(list);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }, 'text');     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }); 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $(function () {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Sys.require([Sys.components.calendar], function () {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var selectedDate = Date.parseInvariant('1-1-2000', 'd-M-yyyy');     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $('#txtDate').calendar({     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'format': 'dd-MM-yyyy',     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'selectedDate': selectedDate,     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'id': 'cal-txtDate'     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }); 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $('#btnFilter').click(function () {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var date = $find('cal-txtDate').get_selectedDate();     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var params = { 'date': date} ;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var serializedParams = Sys.Serialization.JavaScriptSerializer.serialize(params);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $.ajax({     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'url': '/LibraryService.svc/ajax/GetBooksPublishedAfter',     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'data': serializedParams,     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'type': 'POST',     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'processData': false,     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'contentType': 'application/json',     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'timeout': 10000,     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'dataType': 'text', //not JSON, we'll process     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'success': function (res) {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Sys.require([Sys.scripts.Serialization, Sys.scripts.Globalization], function () {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; res = Sys.Serialization.JavaScriptSerializer.deserialize(res);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var data = res.d; //data is in d     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var list = $('&lt;ul&gt;&lt;/ul&gt;');&#160; 
    
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; for (var i = 0; i &lt; data.length; i++) {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; list.append('&lt;li&gt;' + data[i].Title + '- ' + data[i].Author + ', $' + data[i].Price.format('n') + ', ' + data[i].PublishDate.format('dd-MMM-yyyy'));     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $('#filtered-books').empty().html(list);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; },     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 'error': function () {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; alert('An error occurred.');     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }); 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; });    
&#160;&#160;&#160; &lt;/script&gt;     
&lt;/head&gt;     
&lt;body&gt;     
&#160;&#160;&#160; &lt;fieldset&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;legend&gt;All Books&lt;/legend&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;div id='all-books'&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/div&gt;     
&#160;&#160;&#160; &lt;/fieldset&gt;     
&#160;&#160;&#160; &lt;br /&gt;     
&#160;&#160;&#160; &lt;fieldset&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;legend&gt;Filtered Books&lt;/legend&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;div&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Please select a date:     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;input type='text' id='txtDate' /&gt;&lt;br /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;input type='button' id='btnFilter' value='fetch books' /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/div&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;div id='filtered-books'&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/div&gt;     
&#160;&#160;&#160; &lt;/fieldset&gt;     
&lt;/body&gt;     
&lt;/html&gt;    
    
</pre></p>  <p>Running the page, selecting a date and clicking the button should show something like this:</p>  <p><a href="/media/default/images/results4_1.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="results4" border="0" alt="results4" src="/media/default/images/results4_thumb_1.png" width="375" height="341" /></a> </p>  <p>&#160;</p>  <h3>Summary of Ajax Tests</h3>  <p>Our Ajax-exposed WCF service is callable both from ASP.NET Ajax and jQuery. While jQuery is usually a lot leaner than ASP.NET Ajax in terms of things like animation and DOM manipulation, the new features of ASP.NET Ajax 4.0 make the latter a very attractive library for cleaner client template based databinding deriving its data from a service call. While jQuery also supports templates via plugins, ASP.NET Ajax 4.0's dataview control makes fetching data and displaying it in a reusable template quite elegant. The mere use of the word &quot;elegant&quot; relating to ASP.NET Ajax would have caused rounds of laughter in the past. v4.0 is looking quite good indeed.</p>  <p>Furthermore, WCF services handle dates in a specific way and also add some security to network calls. ASP.NET Ajax handles these issues seamlessly. Using jQuery, we had to pass and receive raw data and as such had to process the data before sending and after receiving. I used ASP.NET's serializer for this, although you could just as easily have used modified JSON2 (like Rick Strahl mentions in the previously mentioned link) or any other suitable technique. But the bottom line is that you would have had to handle these things which ASP.NET Ajax handles for you out of the box.</p>  <p>By this, I'm in no way saying that ASP.NET Ajax is better than jQuery or that jQuery is dead – far from it. What I'm saying is that there are things jQuery does way way better than ASP.NET Ajax, but handling WCF calls is not one of them. Those thinking about the size of the ASP.NET Ajax Library should rest assured that the on demand and in-parallel loading of only the required parts of the ASP.NET Ajax Library will ensure script downloads are very fast. Microsoft acknowledges that jQuery is a really really useful library and as such has adopted jQuery into the ASP.NET developer's toolbox. And with ASP.NET Ajax 4.0, it does seem like the two will finally complement each other.</p>  <h3>Exposing to Silverlight</h3>  <p>We'll now modify our configuration to support binary encoding for Silverlight. Please note that since we exposed our service's metadata for HTTP-GET via the &quot;metadataGetBehavior&quot; service behaviour, our service is already exposed to Silverlight. What we want to do now is expose the service for binary encoding. This will mean that data transfers are done in binary format, which is much more efficient than basic http. The first step in doing this is to add a custom binding. Right after our closing &lt;/behaviors&gt; tag, add the following xml:</p>  <p><pre class='brush:xml'>   
&lt;bindings&gt;     
&#160;&#160;&#160;&#160;&#160; &lt;customBinding&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;binding name=&quot;binaryEncoding&quot;&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;binaryMessageEncoding /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;httpTransport /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/binding&gt;     
&#160;&#160;&#160;&#160;&#160; &lt;/customBinding&gt;     
&#160;&#160;&#160; &lt;/bindings&gt;    
    
</pre></p>  <p>The next step is to add a new endpoint for our service. Add the following to the &lt;endpoints&gt; section:</p>  <p><pre class='brush:xml'>   
&lt;endpoint address=&quot;binary&quot;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; binding=&quot;customBinding&quot; bindingConfiguration=&quot;binaryEncoding&quot;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; contract=&quot;WCFEndpoints.LibraryService&quot; /&gt;    
    
</pre> </p>  <p>Notice that we're specifying &quot;customBinding&quot; for the binding and passing in &quot;binaryEncoding&quot; for the binding configuration. &quot;binaryEncoding&quot; is the binding configuration we added to the &lt;customBinding&gt; node in the &lt;bindings&gt; section. </p>  <p>And that's all it takes to enable binary encoding on a WCF sercive. No messy attributes or changing the C# code – just a few lines of configuration. Your WCF section of the web.config should now look like this:</p>  <p><pre class='brush:xml'>   
&lt;system.serviceModel&gt;     
&#160;&#160;&#160; &lt;behaviors&gt;     
&#160;&#160;&#160;&#160;&#160; &lt;endpointBehaviors&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;behavior name=&quot;AjaxBehavior&quot;&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;enableWebScript /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/behavior&gt;     
&#160;&#160;&#160;&#160;&#160; &lt;/endpointBehaviors&gt;     
&#160;&#160;&#160;&#160;&#160; &lt;serviceBehaviors&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;behavior name=&quot;metadataGetBehavior&quot;&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;serviceMetadata httpGetEnabled=&quot;true&quot;/&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;serviceDebug includeExceptionDetailInFaults=&quot;true&quot;&gt;&lt;/serviceDebug&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/behavior&gt;     
&#160;&#160;&#160;&#160;&#160; &lt;/serviceBehaviors&gt;     
&#160;&#160;&#160; &lt;/behaviors&gt;     
&#160;&#160;&#160; &lt;bindings&gt;     
&#160;&#160;&#160;&#160;&#160; &lt;customBinding&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;binding name=&quot;binaryEncoding&quot;&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;binaryMessageEncoding /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;httpTransport /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/binding&gt;     
&#160;&#160;&#160;&#160;&#160; &lt;/customBinding&gt;     
&#160;&#160;&#160; &lt;/bindings&gt;     
&#160;&#160;&#160; &lt;serviceHostingEnvironment aspNetCompatibilityEnabled=&quot;true&quot;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; multipleSiteBindingsEnabled=&quot;true&quot; /&gt;     
&#160;&#160;&#160; &lt;services&gt;     
&#160;&#160;&#160;&#160;&#160; &lt;service name=&quot;WCFEndpoints.LibraryService&quot; behaviorConfiguration=&quot;metadataGetBehavior&quot;&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;endpoint address=&quot;ajax&quot; behaviorConfiguration=&quot;AjaxBehavior&quot;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; binding=&quot;webHttpBinding&quot; contract=&quot;WCFEndpoints.LibraryService&quot; /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;endpoint address=&quot;binary&quot;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; binding=&quot;customBinding&quot; bindingConfiguration=&quot;binaryEncoding&quot;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; contract=&quot;WCFEndpoints.LibraryService&quot; /&gt;     
&#160;&#160;&#160;&#160;&#160; &lt;/service&gt;     
&#160;&#160;&#160; &lt;/services&gt;     
&#160; &lt;/system.serviceModel&gt;    
    
</pre></p>  <h3>Silverlight Test</h3>  <p>Let's create the Silverlight project. Add one named SilverlightWCFClient to the solution:</p>  <p><a href="/media/default/images/SL_Add_1.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="SL_Add" border="0" alt="SL_Add" src="/media/default/images/SL_Add_thumb_1.png" width="548" height="387" /></a> </p>  <p>Keep the defaults for the popup that appears:</p>  <p><a href="/media/default/images/SL_Add2_1.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="SL_Add2" border="0" alt="SL_Add2" src="/media/default/images/SL_Add2_thumb_1.png" width="390" height="387" /></a> </p>  <p>Before we go any further, we'll want our main application to launch from the same port on the dev server. To do this, right click the WCFEndpoints project and select properties. From the Web tab, select the &quot;Specific port&quot; radio button, assign a port of your choice (that's free) and save.</p>  <p><a href="/media/default/images/port_1.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="port" border="0" alt="port" src="/media/default/images/port_thumb_1.png" width="600" height="372" /></a> </p>  <p>Now right click the SilverlightWCFClient project in the solution explorer and select &quot;Add Service Reference&quot;. Click on the &quot;Discover&quot; button. You should see something like this:</p>  <p><a href="/media/default/images/add_svc_ref_1.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="add_svc_ref" border="0" alt="add_svc_ref" src="/media/default/images/add_svc_ref_thumb_1.png" width="539" height="443" /></a> </p>  <p>To ensure your metadata is discoverable, click on LibraryService.svc from the left pane and expand the tree. If everything's ok, you should get a tree on the left and a list of operations on the right. Remember that we added a &quot;metadataGetBehavior&quot;? That ensures that our service is discoverable by the Add Service Reference tool (and any other tools that consume wsdl). Name the service namespace &quot;LibraryServiceReference&quot; and click Ok.</p>  <p><a href="/media/default/images/add_svc_ref2_1.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="add_svc_ref2" border="0" alt="add_svc_ref2" src="/media/default/images/add_svc_ref2_thumb_1.png" width="526" height="430" /></a> </p>  <p>The tool generates some proxy classes, details of which can be seen in the object browser by double clicking the LibraryServiceReference:</p>  <p><a href="/media/default/images/svc_ref_object_1.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="svc_ref_object" border="0" alt="svc_ref_object" src="/media/default/images/svc_ref_object_thumb_1.png" width="545" height="263" /></a> </p>  <p>It also adds a ServiceReferences.ClientConfig file:</p>  <p><pre class='brush:xml'>   
&lt;configuration&gt;     
&#160;&#160;&#160; &lt;system.serviceModel&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;bindings&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;customBinding&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;binding name=&quot;CustomBinding_LibraryService&quot;&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;binaryMessageEncoding /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;httpTransport maxReceivedMessageSize=&quot;2147483647&quot; maxBufferSize=&quot;2147483647&quot; /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/binding&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/customBinding&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/bindings&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;client&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;endpoint address=&quot;<a href="http://localhost:12171/LibraryService.svc/binary&quot;">http://localhost:12171/LibraryService.svc/binary&quot;</a>     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; binding=&quot;customBinding&quot; bindingConfiguration=&quot;CustomBinding_LibraryService&quot;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; contract=&quot;LibraryServiceReference.LibraryService&quot; name=&quot;CustomBinding_LibraryService&quot; /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/client&gt;     
&#160;&#160;&#160; &lt;/system.serviceModel&gt;     
&lt;/configuration&gt;    
    
</pre></p>  <p>Notice that the binding for the endpoint also has &quot;binaryMessageEncoding&quot; and &quot;httpTransport&quot;. The tool was smart enough to see that our service had a binary endpoint and used that to ensure optimum data transfer efficiency. Also note the the endpoint address specifies the server and port number. It's for this reason that we explicitly specified a fixed port number for our web project. </p>  <p>I won't walk you through creating the Silverlight page that consumes the service. The code is pretty trivial and you can download the whole project's source from the link at the end of the article. After creating the page, if we launch SilverlightWCFClientTestPage.html (of the WCFEndpoints project) and click the button, we get the following output:</p>  <p><a href="/media/default/images/SL_1.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="SL" border="0" alt="SL" src="/media/default/images/SL_thumb_1.png" width="497" height="320" /></a> </p>  <p>Running the page in Firefox with Firebug's Net tab activated, we should see this (after clicking the button):</p>  <p><a href="/media/default/images/firebug1_1.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="firebug1" border="0" alt="firebug1" src="/media/default/images/firebug1_thumb_1.png" width="620" height="251" /></a> </p>  <p>Notice that the last two items are &quot;POST binary&quot;. Expanding either one and selecting the Post tab, we get this:</p>  <p><a href="/media/default/images/firebug2_1.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="firebug2" border="0" alt="firebug2" src="/media/default/images/firebug2_thumb_1.png" width="628" height="139" /></a> </p>  <p>That shows us that the request data is being passed to the web service in binary format. Unfortunately, the version of firebug I'm using can't interpret the binary response, so the Response tab is pretty useless. Of course, had we been using basic http binding, we would have seen text data in the response, so our &quot;useless&quot; Response tab is actually suggesting that the response is not in a textual format.</p>  <h3>Conclusion</h3>  <p>In this article, we've seen how we can create a WCF service by coding once and expose it to different technologies merely by changing the configuration. We created the service and exposed it's metadata via Http GET. We had it expose to Ajax clients via configuration. We then created two plain html pages. In the first, we used ASP.NET Ajax 4.0's new features to consume the Ajax service quite elegantly. In the second, we used jQuery. In doing so, we saw how we needed to serialize / deserialize inputs / outputs to / from a WCF service when using jQuery. We used ASP.NET Ajax's serialization capabilities to do this. We then proceeded to add an endpoint to the WCF service that would allow communication in the more efficient binary format. Finally, we used firebug to show that the data transfer was indeed being done via binary and not basic http. </p>  <h3>Source Code</h3>  <p><a href="http://cid-9d8a7728356393b1.skydrive.live.com/self.aspx/.Public/code/WCFEndpoints.zip">Download</a></p>  <p>----------------------------------------------------------------------------------</p>  <p><em><font color="#ff0000">Questions relating to this article are welcome. Comments completely unrelated to the article and posted with the sole intention of putting your link here are not. If you spam, your comment will not be approved, will be deleted and your IP blocked. I maintain my site almost daily and such comments – even if they pass the spam filter – will get removed as soon as possible. If this gets too tedious, I may disable comments entirely. Please don't ruin it for everybody else.</font></em></p>  <p><em>----------------------------------------------------------------------------------</em></p>
        