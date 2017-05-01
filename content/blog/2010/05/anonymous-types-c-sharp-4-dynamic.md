
+++
title = "Anonymous Types are Internal, C# 4.0 Dynamic Beware!"
description = "C# 4.0 has introduced the dynamic keyword. You can declare a variable as dynamic and regardless of w ..."
tags = [ "C#", ".NET", "blog" ]
date = "2010-05-26 19:10:47"
slug = "anonymous-types-c-sharp-4-dynamic"
+++
<p>C# 4.0 has introduced the dynamic keyword. You can declare a variable as dynamic and regardless of what can be inferred at compile time, you can access any properties and call any methods and your code will still compile. Resolving of those properties and methods will be done at runtime. If at runtime, they aren't found, you'd get a runtime exception. If they are found, your code will run fine.</p>  <p><strong>An Example</strong></p>  <p>Consider a class called Person that has a property called Name. Consider the following piece of code:</p>  <p><pre class='brush:c#'>    
    
Person p = new Person { Name = &quot;John&quot; };     
object o = p;     
dynamic d = o;     
Console.WriteLine(d.Name);     
    
</pre></p>  <p>On line 2, we're assigning p to an object reference o. Before C# 4.0, if you wanted to get the value of o's Name (and it does have a Name, since we know we assigned a Person object to o) you would have had to resort to reflection. With C# 4.0 however, we can use a dynamic variable like we did on line 3, and simply access the Name property on the dynamic variable. This program would compile fine, but it would require that the object assigned to d must (at runtime) have a property called Name. If it doesn't, it'll throw an exception.</p>  <p>So basically, we can assign a dynamic variable with any object and call properties and methods we expect it to have and it should run fine. Right? Not quite.</p>  <p><strong>The Problem</strong></p>  <p>The dynamic approach has one limitation – it must know the type of object it's dealing with. This isn't a problem for most cases, but it can be an issue when dealing with anonymous types. Let's get started building the project so you can see what I mean.</p>  <p><strong>The Project</strong></p>  <p>Create a basic .Net 4.0 console application called DynamicTest. Add a class library called ClassLibrary1 to the solution. Make class1.cs look like this:</p>  <p><pre class='brush:c#'>    
    
namespace ClassLibrary1     
{     
&#160;&#160;&#160; public class Person     
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; public string Name { get; set; }     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; public int Age { get; set; }     
&#160;&#160;&#160; }&#160; 
    
&#160;&#160;&#160; public class Repository     
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; Person _person = new Person { Name = &quot;Adam&quot;, Age = 21 };     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; 
&#160;&#160;&#160;&#160;&#160;&#160;&#160; public object GetPerson()     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; return _person;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; }     
    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; public object GetPersonWrappedInAnonymousType()     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; return new { Person = _person };     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; }     
&#160;&#160;&#160; }     
}     
    
</pre></p>  <p>We have a repository that has two methods. One simply returns a person as an object while the other wraps it in an anonymous type and returns an instance of that anonymous type. Going back to the main project's Program.cs, add the following code:</p>  <p><pre class='brush:c#'>    
    
class Program     
{     
&#160;&#160;&#160; static void Main(string[] args)     
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; new Program().Run();     
&#160;&#160;&#160; }&#160;&#160;&#160;&#160; 
    
&#160;&#160;&#160; private void Run()     
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var rep = new Repository();     
    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; dynamic data = rep.GetPerson();     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; Console.WriteLine(data.Name);&#160; 
    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; dynamic data2 = rep.GetPersonWrappedInAnonymousType();     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; Console.WriteLine(data2.Person.Name);     
&#160;&#160;&#160; }     
}     
    
</pre>     <br />    <br />That looks simple enough. data and data2 are both dynamic variables. As such, their Name and Person.Name properties will be resolved at runtime. The program compiles fine, but when we run it, we get this:</p>  <p><em><strong>Adam </strong></em></p>  <p><em><strong>Unhandled Exception: Microsoft.CSharp.RuntimeBinder.RuntimeBinderException: 'obj        <br />ect' does not contain a definition for 'Person'         <br />&#160;&#160; at CallSite.Target(Closure , CallSite , Object )         <br />&#160;&#160; at System.Dynamic.UpdateDelegates.UpdateAndExecute1[T0,TRet](CallSite site, T         <br />0 arg0)         <br />&#160;&#160; at DynamicTest.Program.Run() in F:\MyTests\C#\DynamicTest\DynamicTest\Program         <br />.cs:line 24         <br />&#160;&#160; at DynamicTest.Program.Main(String[] args) in F:\MyTests\C#\DynamicTest\Dynam         <br />icTest\Program.cs:line 13         <br />Press any key to continue . . .</strong></em></p>  <p>How come? The call to data.Name resolved fine, but the call to data2.Person failed. The error message states that the Person property could not be found at runtime. We can obviously see by looking at the code that data2 most definitely has a Person property and that in turn has a Name property. [And if you don't trust your eyes, you can run in debug mode, set a breakpoint just after data2 is initialized and if you use the watch window to view the properties of data2, you'll see that data2 does indeed have a data2.Person property but if you type in &quot;data2.Person&quot; in the watch window, you'll get the same exception…magic!]. So what gives?</p>  <p><strong>Explanation</strong></p>  <p>The reason the call to data2.Person fails is that the type information of data2 is not available at runtime. The reason it's not available is because anonymous types are not public. When the method is returning an instance of that anonymous type, it's returning a System.Object which references an instance of an anonymous type -&#160; a type who's info isn't available to the main program. The dynamic runtime tries to find a property called Person on the object, but can't resolve it from the type information it has. As such, it throws an exception. The call to data.Name works fine since Person is a public class, that information is available and can be easily resolved. </p>  <p>This can affect you in any of the following cases (if not more):</p>  <p>1. You're returning a non-public, non-internal type using System.Object.    <br />2. You're returning a non-public, non-internal derived type via a public Base type and accessing a property in the derived type that's not in the base type.     <br />3. You're returning anything wrapped inside an anonymous type from a different assembly.</p>  <p>i.e. you need to be able to access the type info for the object at the point where its property / method is being resolved at runtime.</p>  <p><strong>Solution</strong></p>  <p>The solution is actually quite simple. All we have to do is open up AssemplyInfo.cs of the ClassLibrary1 project and add the following line to it:</p>  <p><pre class='brush:c#'>    
[assembly:InternalsVisibleTo(&quot;DynamicTest&quot;)]     
</pre>     <br />    <br />What this does is allow the main project to &quot;look into&quot; the internal types of ClassLibrary1. This means the dynamic runtime can find the type information of the anonymous type being returned and data2.Person.Name resolves perfectly. Anonymous types are internal. Adding that piece of code makes the type information &quot;visible&quot; to our main project.</p>  <p><strong>Drawback</strong></p>  <p>As you can see, the solution means you actually have to have control over the class library's source code for this to work. If you don't, then I guess you'd need to resort to good old fashioned reflection.</p>  <p><strong>Application</strong></p>  <p>This approach can come in handy for unit testing. You would expose the internal types of the assembly under test to the tests, and as such should be able to take advantage of the dynamic functionality of C# 4.0 when evaluating results from methods that return anonymous types.</p>  <p><strong>Source</strong></p>  <p><a href="http://cid-9d8a7728356393b1.skydrive.live.com/self.aspx/.Public/code/DynamicTest.zip">download</a></p>  <p><a href="http://dotnetshoutout.com/Heartysoftcom-Anonymous-Types-are-Internal-C-40-Dynamic-Beware" rev="vote-for"><img style="border-right-width: 0px; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" alt="Shout it" src="http://dotnetshoutout.com/image.axd?url=http%3A%2F%2Fwww.heartysoft.com%2Fpost%2F2010%2F05%2F26%2Fanonymous-types-c-sharp-4-dynamic.aspx" /></a>&#160;<a href="http://www.dotnetkicks.com/kick/?url=http%3a%2f%2fwww.heartysoft.com%2fpost%2f2010%2f05%2f26%2fanonymous-types-c-sharp-4-dynamic.aspx"><img border="0" alt="kick it on DotNetKicks.com" src="http://www.dotnetkicks.com/Services/Images/KickItImageGenerator.ashx?url=http%3a%2f%2fwww.heartysoft.com%2fpost%2f2010%2f05%2f26%2fanonymous-types-c-sharp-4-dynamic.aspx" /></a></p>  <p><em>---------------------------------------------------------</em></p>  <p><em><font color="#ff0000">Questions&#160; and comments relating to this article are welcome. Comments completely unrelated to the article and posted with the sole intention of putting your link here are not.        <br /></font></em><em><font color="#ff0000">       <br />If you spam, your comment will not be approved, will be deleted and your IP blocked. I maintain my site almost daily and such comments – even if they pass the spam filter – will get removed as soon as possible. If this gets too tedious, I may disable comments entirely. Please don't ruin it for everybody else.</font></em></p>  <p><em>---------------------------------------------------------</em></p>
        