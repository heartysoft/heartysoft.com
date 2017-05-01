
+++
title = "Fetching a Property Value via Reflection"
description = "Reflection is a method by which we can use metadata at runtime to dynamically create an instance of  ..."
tags = [ ".NET", "ASP.NET", "C#", "blog" ]
date = "2010-05-11 02:31:51"
slug = "Fetching-a-Property-Value-via-Reflection"
+++
<p>Reflection is a method by which we can use metadata at runtime to dynamically create an instance of a type, bind the type to an existing object, or get the type from an existing object and invoke its methods or access its fields and properties. It has many uses, ranging from allowing usage of attributes to creating instances of dynamically generated classes and using them. You can even use reflection to read and modify private member variables of an object. Today, we're going to look at a very simple example which will allow us to read the value of a public property given an instance.</p>  <p><strong>The Class</strong></p>  <p>Let's take a very simple class:</p>  <p><pre class='brush:c#'>   
    
namespace MyNamespace    
{    
&#160;&#160;&#160; public class MyClass    
&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; public int MyProperty { get; set; }    
&#160;&#160;&#160; }    
}    
    
</pre></p>  <p>Note, that I've put it in a separate namespace to show that it's easy to deal with this when using reflection.</p>  <p><strong>The Runner</strong></p>  <p>We'll run everything in a very simple console application:</p>  <p><pre class='brush:c#'>   
    
public class Program    
{    
&#160;&#160;&#160; static void Main(string[] args)    
&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; new Program().Run();    
&#160;&#160;&#160; }    
    
&#160;&#160;&#160; private void Run()    
&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; MyClass o = new MyClass { MyProperty = 25 };    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; object value = GetPropertyValue(o, &quot;MyProperty&quot;);    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; Console.WriteLine(value);    
&#160;&#160;&#160; }    
}    
    
</pre></p>  <p>We're basically creating an instance of MyClass and passing the instance and the string &quot;MyProperty&quot; to a method called GetProperty. The method is returning an object (which we could have cast to anything we needed). We're passing the returned object to Console.WriteLine. </p>  <p><strong>The Method</strong></p>  <p>This is where we use reflection to find the value of the &quot;MyProperty&quot; property given an instance. Add the following method to the class:</p>  <p><pre class='brush:c#'>   
    
private object GetPropertyValue(object o, string propertyName)    
{    
&#160;&#160;&#160; Type type = o.GetType();    
&#160;&#160;&#160; PropertyInfo info = type.GetProperty(propertyName);    
&#160;&#160;&#160; object value = info.GetValue(o, null);    
&#160;&#160;&#160; return value;    
}    
    
</pre></p>  <p>We're getting the type information by calling GetType() on the object. Next, we call GetProperty() on the type information. Note, this only gives us information about the property and not the property itself. We then have to call GetValue() on the property information passing in the instance from which to get the actual value. </p>  <p>And that's all it takes :)</p>  <p>PS: You'll need a reference to System.Reflection to use Reflection.</p>  <p>PPS: Yes, this is a very very simple example of using reflection, but I found quite a few questions over at the forums today that would be answered by this. We all have to start somewhere, right?</p>
        