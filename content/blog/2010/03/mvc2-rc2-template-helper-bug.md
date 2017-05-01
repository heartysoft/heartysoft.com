
+++
title = "MVC 2 RC 2 Templated Helper Bug and a Potential Solution"
description = "The Problem  ASP.NET MVC 2 introduces templated helpers. They're a convenient and type safe way to r ..."
tags = [ ".NET", "ASP.NET MVC", "ASP.NET", "blog" ]
date = "2010-03-01 18:43:00"
slug = "mvc2-rc2-template-helper-bug"
+++
<h2>The Problem</h2>  <p>ASP.NET MVC 2 introduces templated helpers. They're a convenient and type safe way to render model data. There is a potential problem in the way in which the system processes the lambda expression passed in to the helper. The problem causes overridden properties to be ignored and uses the base class' declaration of the property instead. This can have adverse reactions where the client side validation feature of ASP.NET MVC 2 is used. </p>  <h2>An Example</h2>  <p>Create a very simple BasePerson class:</p>  <p><pre class='brush:c#'>    
public class BasePerson     
{     
&#160;&#160;&#160; public virtual string FirstName { get; set; }     
}     
</pre></p>  <p>Create a Person class that derives from the BasePerson class, but adds some data annotations (of course, you could do anything here that you required):</p>  <p><pre class='brush:c#'>    
public class Person : BasePerson     
{     
&#160;&#160;&#160; [Required(ErrorMessage = &quot;FirstName is required.&quot;)]     
&#160;&#160;&#160; public override string FirstName { get; set; }     
}     
</pre></p>  <p>Add a very simple controller:</p>  <p><pre class='brush:c#'>    
public class HomeController : Controller     
{     
&#160;&#160;&#160; public ActionResult Index()     
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var p = new Person();     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; return View(p);     
&#160;&#160;&#160; }     
}     
</pre></p>  <p>Create the Index.htm view:</p>  <p><pre class='brush:xml'>    
&lt;%@ Page Language=&quot;C#&quot; Inherits=&quot;System.Web.Mvc.ViewPage&lt;MvcApplication1.Models.BasePerson&gt;&quot; %&gt;     
&lt;!DOCTYPE html PUBLIC &quot;-//W3C//DTD XHTML 1.0 Transitional//EN&quot; &quot;<a href="http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd&quot;">http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd&quot;</a>&gt;     
&lt;html xmlns=&quot;<a href="http://www.w3.org/1999/xhtml&quot;">http://www.w3.org/1999/xhtml&quot;</a> &gt;     
&lt;head runat=&quot;server&quot;&gt;     
&#160;&#160;&#160; &lt;title&gt;Index&lt;/title&gt;     
&#160;&#160;&#160; &lt;script src=&quot;/Scripts/MicrosoftAjax.js&quot; type=&quot;text/javascript&quot;&gt;&lt;/script&gt;     
&#160;&#160;&#160; &lt;script src=&quot;/Scripts/MicrosoftMvcAjax.js&quot; type=&quot;text/javascript&quot;&gt;&lt;/script&gt;     
&#160;&#160;&#160; &lt;script src=&quot;/Scripts/MicrosoftMvcValidation.js&quot; type=&quot;text/javascript&quot;&gt;&lt;/script&gt;     
&lt;/head&gt;     
&lt;body&gt;     
&#160;&#160;&#160; &lt;% Html.EnableClientValidation(); %&gt;     
&#160;&#160;&#160; &lt;% Html.BeginForm(); %&gt;     
&#160;&#160;&#160; &lt;div&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;%= Html.TextBoxFor(x=&gt;x.FirstName) %&gt;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;%= Html.ValidationMessageFor(x=&gt;x.FirstName) %&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;input type='submit' value='submit' /&gt;     
&#160;&#160;&#160; &lt;/div&gt;     
&#160;&#160;&#160; &lt;% Html.EndForm(); %&gt;     
&lt;/body&gt;     
&lt;/html&gt;     
</pre></p>  <p><em><font color="#008000">You could have made the ViewPage of type Person, instead of BasePerson, but the error would have still been present.</font></em></p>  <p>If you run the project now, the rendered html responsible for client side validation would look like this:</p>  <p><pre class='brush:xml'>    
//&lt;![CDATA[
  if (!window.mvcClientValidationMetadata) { window.mvcClientValidationMetadata = []; }
  window.mvcClientValidationMetadata.push({&quot;Fields&quot;:    
[{&quot;FieldName&quot;:&quot;FirstName&quot;,&quot;ReplaceValidationMessageContents&quot;:true,     
&quot;ValidationMessageId&quot;:&quot;form0_FirstName_validationMessage&quot;,&quot;ValidationRules&quot;:[]}],     
&quot;FormId&quot;:&quot;form0&quot;,&quot;ReplaceValidationSummary&quot;:false});
  //]]&gt;    
</pre></p>  <p>As you can see, the validation rules are empty and as such, there's no client side validation occurring. To see this, just click the submit button. The form will post back without any problems. That should not be happening if the FirstName property is empty.</p>  <h2>The Reason</h2>  <p>The cause of the bug is a single line of code in the method <strong>FromLambdaExpression </strong>in the file <strong>ModelMetaData.cs</strong> in the source code of MVC 2 RC 2. Here's the relevant portion of code:</p>  <p><pre class='brush:c#'>    
MemberExpression memberExpression = (MemberExpression)expression.Body;     
string propertyName = memberExpression.Member is PropertyInfo ? memberExpression.Member.Name : null;     
Type containerType = memberExpression.Member.DeclaringType;     
</pre></p>  <p>While this looks pretty normal, the third line is the culprit. The DeclaringType refers to the class that declares a member – it's not necessarily the type of the class that's in the Model of our ViewData. In our example, although we passed in a Person object as the model, the DeclaringType of FirstName will always be BasePerson. The consequence of using the DeclaringType member to get the containerType is that the attribute added to the overriden FirstName of the Person class is totally ignored. Any other attributes on an overridden property will also be ignored. </p>  <p><font color="#008000"><em>Note that if you were to use TextBoxFor(model=&gt;model) or TextBox(&quot;FirstName&quot;), then this would not be a problem until and unless        <br />somethingFor(x=&gt;x.FirstName)         <br />is encountered.         <br />Also, server side validation isn't affected by the bug as the bug appears to be centred on that specific third line of code.</em></font></p>  <h2>A Possible Solution</h2>  <p>The main reason for the problem is that FromLambdaExpression is not using the runtime type of the container. Using the runtime type can be achieved in a few lines of code. Let's first add a helper method:</p>  <p><pre class='brush:c#'>    
private static ParameterExpression GetParameterExpression(Expression expression)     
{     
&#160;&#160;&#160; while (expression.NodeType == ExpressionType.MemberAccess)     
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; expression = ((MemberExpression)expression).Expression;     
&#160;&#160;&#160; }     
&#160;&#160;&#160; if (expression.NodeType == ExpressionType.Parameter)     
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; return (ParameterExpression)expression;     
&#160;&#160;&#160; }     
&#160;&#160;&#160; return null;     
}     
</pre></p>  <p><em><font color="#008000">I've taken the GetParameterExpression method from here:        <br /><a href="http://geekswithblogs.net/EltonStoneman/archive/2009/11/05/retrieving-nested-properties-from-lambda-expressions.aspx">http://geekswithblogs.net/EltonStoneman/archive/          <br />2009/11/05/retrieving-nested-properties-from-lambda-expressions.aspx</a>         <br />Credits for that go to Elton Stoneman</font></em></p>  <p><font color="#008000"><font color="#000000">The last thing to do is to add a bit of code to retrieve the runtime type of the container:</font></font></p>  <p><pre class='brush:c#'>    
MemberExpression memberExpression = (MemberExpression) expression.Body;     
string propertyName = memberExpression.Member is PropertyInfo ? memberExpression.Member.Name : null;     
Type containerType = memberExpression.Member.DeclaringType; 
  bool isContainerValueType = typeof (TParameter).IsValueType; 
  if(isContainerValueType || (isContainerValueType == false &amp;&amp; container != null))    
{     
&#160;&#160;&#160; var parameter = GetParameterExpression(memberExpression.Expression);     
&#160;&#160;&#160; var lambda = Expression.Lambda(memberExpression.Expression, parameter);     
&#160;&#160;&#160; var compiled = lambda.Compile();     
&#160;&#160;&#160; var containerInstance = compiled.DynamicInvoke(container);     
&#160;&#160;&#160; containerType = containerInstance.GetType();     
}     
</pre></p>  <p>Running the page and viewing the source, we should see this:</p>  <p><pre class='brush:xml'>    
//&lt;![CDATA[
  if (!window.mvcClientValidationMetadata) { window.mvcClientValidationMetadata = []; }
  window.mvcClientValidationMetadata.push({&quot;Fields&quot;:    
[{&quot;FieldName&quot;:&quot;FirstName&quot;,&quot;ReplaceValidationMessageContents&quot;:true,     
&quot;ValidationMessageId&quot;:&quot;form0_FirstName_validationMessage&quot;,     
&quot;ValidationRules&quot;:[{&quot;ErrorMessage&quot;:&quot;FirstName is required.&quot;,&quot;ValidationParameters&quot;:{},     
&quot;ValidationType&quot;:&quot;required&quot;}]}],&quot;FormId&quot;:&quot;form0&quot;,&quot;ReplaceValidationSummary&quot;:false});
  //]]&gt;    
</pre></p>  <p>As you can see, all the client validation info is now rendered.</p>  <p></p>  <h2>Explanation</h2>  <p>What we've done here isn't really complicated. We're basically taking the old expression (say x=&gt;x.Dummy.Car.Name), getting a lambda delegate for all but the last part of the old expression (so x=&gt;x.Dummy.Car). We're then invoking the delegate with viewData.Model (which is stored in the container variable by code earlier in the method). This gives us the instance of the entity holding the property to be displayed (so in this case, Model.Dummy.Car where the property to be displayed is Model.Dummy.Car.Name). After that, we call GetType() on the instance to get the runtime type. In our coded example, this gives Person and not BasePerson. As such, the validation script rendered honours the [Required] attribute on the overridden FirstName property of the Person class.</p>  <p>I've also added some checks to handle null values for the Model. This can happen if an invalid string is entered for a bool, for example. In these cases, we fall back to the expression derived type. Some unit tests fail without this check. Initially, I also checked to ensure that parameter was not null, but it seems that's not required as it's handled earlier (and all the unit tests pass without this check).</p>  <p>And yes, this method will work on complex types with deep nesting. Not just on direct members of the Model.</p>  <h2>Possible Problem</h2>  <p>I can think of one area where this method may cause problems. If you're encapsulating TempData into a viewmodel to be rendered, then this code will cause a read on the TempData, causing it to expire. As such it won't get rendered onto the page. Of course, if you didn't use this approach, then the TempData would vanish immediately after rendering to the page. So why would you want to use TempData for this in the first place? Use ViewModels and ViewData for rendering. Don't use TempData in the View.</p>  <h2>Does It Break Anything</h2>  <p>After making the changes to the MVC 2 RC 2 source code, all existing 2,281 unit tests of the ASP.NET MVC 2 RC 2 solution still pass. As such, I'm assuming nothing is getting broken.</p>  <h2>Does It Solve Anything Else</h2>  <p>I don't know. If there're other places that depend on FromLambdaExpression, this should fix them as well. Other than that, can't really say.</p>  <p><a href="http://www.dotnetkicks.com/kick/?url=http%3a%2f%2fwww.heartysoft.com%2fpost%2f2010%2f03%2f01%2fmvc2-rc2-template-helper-bug.aspx"><img border="0" alt="kick it on DotNetKicks.com" src="http://www.dotnetkicks.com/Services/Images/KickItImageGenerator.ashx?url=http%3a%2f%2fwww.heartysoft.com%2fpost%2f2010%2f03%2f01%2fmvc2-rc2-template-helper-bug.aspx" /></a>&#160; <a href="http://dotnetshoutout.com/MVC-2-RC-2-Templated-Helper-Bug-and-a-Potential-Solution" rev="vote-for"><img style="border-right-width: 0px; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" alt="Shout it" src="http://dotnetshoutout.com/image.axd?url=http%3A%2F%2Fwww.heartysoft.com%2Fpost%2F2010%2F03%2F01%2Fmvc2-rc2-template-helper-bug.aspx" /></a> </p>  <p>---------------------------------------------------------</p>  <p><font color="#ff0000">Questions&#160; and comments relating to this article are welcome.      <br />Comments completely unrelated to the article and posted with the sole intention of putting your link here are not.       <br /></font></em><em><font color="#ff0000">       <br />If you spam, your comment will not be approved, will be deleted and your IP blocked. I maintain my site almost daily and such comments – even if they pass the spam filter – will get removed as soon as possible. If this gets too tedious, I may disable comments entirely. Please don't ruin it for everybody else.</font></em> </p>  <p>---------------------------------------------------------</p>
        