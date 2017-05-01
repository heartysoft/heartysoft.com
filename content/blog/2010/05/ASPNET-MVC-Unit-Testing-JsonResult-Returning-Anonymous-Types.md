
+++
title = "ASP.NET MVC – Unit Testing JsonResult Returning Anonymous Types"
description = "Unit testing of ASP.NET MVC JsonResults can be a source of confusion. The problem arises from the fa ..."
tags = [ ".NET", "ASP.NET", "ASP.NET MVC", "blog" ]
date = "2010-05-25 20:41:54"
slug = "ASPNET-MVC-Unit-Testing-JsonResult-Returning-Anonymous-Types"
+++
<p>Unit testing of ASP.NET MVC JsonResults can be a source of confusion. The problem arises from the fact that an Action Method itself doesn't produce any html / json / string output – it simply returns an Action Result. ASP.NET MVC then calls the ExecuteResult() method on that Action Result. The ExecuteResult() method is what causes output to be written to the Response stream. Let's take the following controller and action method for example:</p>  <p><pre class='brush:c#'>    
    
public class HomeController : Controller     
{     
&#160;&#160;&#160; public ActionResult Index()     
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var people = new List&lt;Person&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; new Person{ Name = &quot;Adam&quot;, Age=21},     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; new Person{ Name = &quot;Eve&quot;, Age=20},     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; new Person{ Name = &quot;Joe&quot;, Age=50}     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; };&#160; 
    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; return Json(new     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; People = people,     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Count = people.Count,     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Success = true     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; }, JsonRequestBehavior.AllowGet);     
&#160;&#160;&#160; }     
}     
    
</pre></p>  <p>Here's the (very simple) Person class:</p>  <p><pre class='brush:c#'>    
    
public class Person     
{     
&#160;&#160;&#160; public string Name { get; set; }     
&#160;&#160;&#160; public int Age { get; set; }     
}     
    
</pre></p>  <p>As you can see, the Index method is simply returning an anonymous type which holds a list of people, the count, indicates success and wraps the whole thing in a JsonResult. When the Index() action method executes (i.e. is called from the browser), it produces the following json:</p>  <p>{&quot;People&quot;:[{&quot;Name&quot;:&quot;Adam&quot;,&quot;Age&quot;:21},{&quot;Name&quot;:&quot;Eve&quot;,&quot;Age&quot;:20},{&quot;Name&quot;:&quot;Joe&quot;,&quot;Age&quot;:50}],&quot;Count&quot;:3,&quot;Success&quot;:true}</p>  <p>Nothing new there. Now let's say we wish to unit test this method [I know, I know…in a real app, the list of people would come from a service or repository or whatever…but for unit tests, you'd have mock whatevers or stub whatevers and would know what the data being returned would be]. We naively write something like this:</p>  <p><pre class='brush:c#'>    
    
[TestMethod]     
public void IndexTestWhichShouldFail()     
{     
&#160;&#160;&#160; //arrange     
&#160;&#160;&#160; HomeController controller = new HomeController();     
    
&#160;&#160;&#160; //act     
&#160;&#160;&#160; var result = controller.Index() as JsonResult;     
    
&#160;&#160;&#160; //assert     
&#160;&#160;&#160; Assert.AreEqual(@&quot;{&quot;&quot;People&quot;&quot;:[{&quot;&quot;Name&quot;&quot;:&quot;&quot;Adam&quot;&quot;,&quot;&quot;Age&quot;&quot;:21},{&quot;&quot;Name&quot;&quot;:&quot;&quot;Eve&quot;&quot;,&quot;&quot;Age&quot;&quot;:20},{&quot;&quot;Name&quot;&quot;:&quot;&quot;Joe&quot;&quot;,&quot;&quot;Age&quot;&quot;:50}],&quot;&quot;Count&quot;&quot;:3,&quot;&quot;Success&quot;&quot;:true}&quot;,     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; result.Data.ToString());     
}     
    
</pre></p>  <p>We run the test, and it fails miserably:</p>  <p>Assert.AreEqual failed. Expected:&lt;{&quot;People&quot;:[{&quot;Name&quot;:&quot;Adam&quot;,&quot;Age&quot;:21},{&quot;Name&quot;:&quot;Eve&quot;,&quot;Age&quot;:20},{&quot;Name&quot;:&quot;Joe&quot;,&quot;Age&quot;:50}],&quot;Count&quot;:3,&quot;Success&quot;:true}&gt;. Actual:&lt;{ People = System.Collections.Generic.List`1[MvcApplication1.Models.Person], Count = 3, Success = True }&gt;.</p>  <p>What went wrong? The reason that the test failed was because we were comparing apples to oranges. We expected a json string of results, but we were passing the ToString() value of the JsonResult's Data property. But shouldn't Data.ToString() simply give us the json string? Erm…no. The Data property simply holds the object passed into the Json() method in Index(). As such, it's simply an instance of an anonymous type. Calling ToString() on it does not cause serialization to json…it simply tells .NET to convert it to a string. The Actual value of the test result above shows what that looks like (i.e. instead of the People list getting serialzed to json, we get an ugly List`1 of Person). So how come it gives us proper json when running from a browser? The reason is that ASP.NET MVC takes the JsonResult and calls its ExecuteResult() method, which (among other things) writes the json to the Response.</p>  <p>So how do we unit test that?</p>  <p>Well, if we were returning non-anonymous types, then we could simply cast JsonResult.Data to the expected type and check its properties. But we're returning an anonymous type, aren't we?</p>  <p>So how do we unit test that?</p>  <p>Well, there're three approaches we can take.</p>  <p><strong>Approach 1: The Easy Way</strong></p>  <p>We know that we have the object and we know what the json for it should be. We can just serialize the object and see if it matches what we expect. Here's the test:</p>  <p><pre class='brush:c#'>    
    
[TestMethod()]     
public void IndexTest()     
{     
&#160;&#160;&#160; //arrange     
&#160;&#160;&#160; HomeController controller = new HomeController();&#160; 
&#160;&#160;&#160; 
&#160;&#160;&#160; //act     
&#160;&#160;&#160; var result = controller.Index() as JsonResult;     
&#160;&#160;&#160; var serializer = new JavaScriptSerializer();     
&#160;&#160;&#160; var output = serializer.Serialize(result.Data);     
    
&#160;&#160;&#160; //assert     
&#160;&#160;&#160; Assert.AreEqual(@&quot;{&quot;&quot;People&quot;&quot;:[{&quot;&quot;Name&quot;&quot;:&quot;&quot;Adam&quot;&quot;,&quot;&quot;Age&quot;&quot;:21},{&quot;&quot;Name&quot;&quot;:&quot;&quot;Eve&quot;&quot;,&quot;&quot;Age&quot;&quot;:20},{&quot;&quot;Name&quot;&quot;:&quot;&quot;Joe&quot;&quot;,&quot;&quot;Age&quot;&quot;:50}],&quot;&quot;Count&quot;&quot;:3,&quot;&quot;Success&quot;&quot;:true}&quot;,     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; output);     
}     
    
</pre></p>  <p>Nothing complicated there. It's basically doing the same thing as the ExecuteResult() method of JsonResult does (in terms of producing the Json string via the JavaScriptSerializer). Now some of you may argue that this is not a unit test as it tests serialization as well as the method under test (i.e. Index). I would disagree. The reason being that you should only be testing your code. Testing JavaScriptSerializer should not be your focus when you're testing the Index method. And since checking the serialzed result is easier for you (i.e. you know what the json should be), you may as well check against that. </p>  <p><strong>Approach 2: The &quot;Pure&quot; Way</strong></p>  <p>This approach executes pretty much what the MVC framework does to produce output. For this, we're going to use Moq to fake a controller context and response and capture the content written to Response during execution in a StringBuilder. We will execute the result with the fake context, which will fill our StringBuilder with the serialized output. We will then check to see if it is what we want it to be. Here's the test:</p>  <p><pre class='brush:c#'>    
    
[TestMethod()]     
public void IndexTestWithMock()     
{     
&#160;&#160;&#160; //arrange     
&#160;&#160;&#160; HomeController controller = new HomeController();     
&#160;&#160;&#160; var result = controller.Index();     
&#160;&#160;&#160; var sb = new StringBuilder();     
&#160;&#160;&#160; Mock&lt;HttpResponseBase&gt; response = new Mock&lt;HttpResponseBase&gt;();     
&#160;&#160;&#160; response.Setup(x =&gt; x.Write(It.IsAny&lt;string&gt;())).Callback&lt;string&gt;(y =&gt;     
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; sb.Append(y);     
&#160;&#160;&#160; });     
&#160;&#160;&#160; Mock&lt;ControllerContext&gt; controllerContext = new Mock&lt;ControllerContext&gt;();     
&#160;&#160;&#160; controllerContext.Setup(x =&gt; x.HttpContext.Response).Returns(response.Object);     
    
&#160;&#160;&#160; //act     
&#160;&#160;&#160; result.ExecuteResult(controllerContext.Object);     
    
&#160;&#160;&#160; //assert     
&#160;&#160;&#160; Assert.AreEqual(@&quot;{&quot;&quot;People&quot;&quot;:[{&quot;&quot;Name&quot;&quot;:&quot;&quot;Adam&quot;&quot;,&quot;&quot;Age&quot;&quot;:21},{&quot;&quot;Name&quot;&quot;:&quot;&quot;Eve&quot;&quot;,&quot;&quot;Age&quot;&quot;:20},{&quot;&quot;Name&quot;&quot;:&quot;&quot;Joe&quot;&quot;,&quot;&quot;Age&quot;&quot;:50}],&quot;&quot;Count&quot;&quot;:3,&quot;&quot;Success&quot;&quot;:true}&quot;,     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; sb.ToString());     
}     
    
</pre></p>  <p>Notice that our &quot;act&quot; of the test is not when we call controller.Index(), rather when we call ExecuteResult() on the JsonResult. In the previous approach, we simply called into the controller's action method. In this approach, we are actually doing what MVC does when dealing with a Request (at least, parts of it). To make it play the way we want it to, we created Mocks that behave the way we want them to. [Of course, the &quot;arrange&quot;, &quot;act&quot; and &quot;assert&quot; parts are just nomenclature and one could say that the &quot;act&quot; is when Index() is called and everything afterwards is part of &quot;assert&quot;…]</p>  <p><strong>Approach 3: Reflection</strong></p>  <p>The previous two approaches compared the generated json against the expected json. As I said, that might upset some people as it kind of tests serialization too. I'm fine with it, but if someone wanted to test the JsonResult itself and not the generated output, one approach they can take is reflection. Here's a test that does just that:</p>  <p><pre class='brush:c#'>    
    
[TestMethod]     
public void IndexTestWithReflection()     
{     
&#160;&#160;&#160; //arrange     
&#160;&#160;&#160; HomeController controller = new HomeController();     
    
&#160;&#160;&#160; //act     
&#160;&#160;&#160; var result = controller.Index() as JsonResult;     
    
&#160;&#160;&#160; //assert     
&#160;&#160;&#160; var data = result.Data;     
&#160;&#160;&#160; var type = data.GetType();     
&#160;&#160;&#160; var countPropertyInfo = type.GetProperty(&quot;Count&quot;);     
&#160;&#160;&#160; var expectedCount = countPropertyInfo.GetValue(data, null);     
    
&#160;&#160;&#160; Assert.AreEqual(3, expectedCount);     
}     
    
</pre></p>  <p>Since result.Data is an instance of an anonymous type, you can't really cast it to anything useful. You can check the values of result.Data via reflection. It can be a bit messy, but you could write a utility library to get to the properties in an easier fashion. There is one problem though…you're checking for strings (i.e. the property &quot;Count&quot;) and as such, refactorings like renaming of a property will cause your tests to fail.</p>  <p><strong>Approach 4: Dynamic</strong></p>  <p>One way to avoid reflection would be to use .Net 4.0's dynamic capabilities. This does, however, require a bit of tinkering. We'll need to open up AssemblyInfo.cs of the MVC application and add this to the bottom:</p>  <p><pre class='brush:c#'>    
[assembly:InternalsVisibleTo(&quot;TestProject1&quot;)]     
</pre></p>  <p>Why we need to do this is explained in this <a href="http://www.heartysoft.com/post/2010/05/26/anonymous-types-c-sharp-4-dynamic.aspx" target="_blank">article</a>.</p>  <p>Once that's done, we can write our test like this:</p>  <p><pre class='brush:c#'>    
    
[TestMethod]     
public void IndexTestWithDynamic()     
{     
&#160;&#160;&#160; //arrange     
&#160;&#160;&#160; HomeController controller = new HomeController();&#160;&#160;&#160;&#160; 
    
&#160;&#160;&#160; //act     
&#160;&#160;&#160; var result = controller.Index() as JsonResult;&#160;&#160;&#160;&#160; 
    
&#160;&#160;&#160; //assert     
&#160;&#160;&#160; dynamic data = result.Data;&#160; 
    
&#160;&#160;&#160; Assert.AreEqual(3, data.Count);     
&#160;&#160;&#160; Assert.IsTrue(data.Success);     
&#160;&#160;&#160; Assert.AreEqual(&quot;Adam&quot;, data.People[0].Name);     
}     
    
</pre></p>  <p>Note how we're avoiding reflection, but still are testing the data that went into the JsonResult as opposed to the json string gotten from executing the JsonResult.</p>  <hr />  <p>All three of these approaches will work. Pick whichever one suits your ideologies.</p>  <p>And yes, I'm aware that the Index() method would be better off returning a PagedList&lt;Person&gt;, and that Success=true is for the most part redundant (if there was an error, it should throw an exception and the calling javascript code should be able to understand that there was an error). The Index() method is for illustrative purposes only ;)</p>  <p><strong>Update</strong></p>  <p>I've added a fourth approach (i.e. dynamic). The source code download below contains the update.</p>  <p><strong>Code</strong></p>  <p><a href="http://cid-9d8a7728356393b1.skydrive.live.com/self.aspx/.Public/code/JsonResultTesting.zip" target="_blank">Click here to download the source code.</a></p>  <p><a href="http://dotnetshoutout.com/Heartysoftcom-ASPNET-MVC-Unit-Testing-JsonResult-Returning-Anonymous-Types" rev="vote-for"><img style="border-right-width: 0px; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" alt="Shout it" src="http://dotnetshoutout.com/image.axd?url=http%3A%2F%2Fwww.heartysoft.com%2Fpost%2F2010%2F05%2F25%2FASPNET-MVC-Unit-Testing-JsonResult-Returning-Anonymous-Types.aspx" /></a>&#160;<a href="http://www.dotnetkicks.com/kick/?url=http%3a%2f%2fwww.heartysoft.com%2fpost%2f2010%2f05%2f25%2fASPNET-MVC-Unit-Testing-JsonResult-Returning-Anonymous-Types.aspx"><img border="0" alt="kick it on DotNetKicks.com" src="http://www.dotnetkicks.com/Services/Images/KickItImageGenerator.ashx?url=http%3a%2f%2fwww.heartysoft.com%2fpost%2f2010%2f05%2f25%2fASPNET-MVC-Unit-Testing-JsonResult-Returning-Anonymous-Types.aspx" /></a></p>  <p><em>---------------------------------------------------------</em></p>  <p><em><font color="#ff0000">Questions&#160; and comments relating to this article are welcome. Comments completely unrelated to the article and posted with the sole intention of putting your link here are not.        <br /></font></em><em><font color="#ff0000">       <br />If you spam, your comment will not be approved, will be deleted and your IP blocked. I maintain my site almost daily and such comments – even if they pass the spam filter – will get removed as soon as possible. If this gets too tedious, I may disable comments entirely. Please don't ruin it for everybody else.</font></em></p>  <p><em>---------------------------------------------------------</em></p>
        