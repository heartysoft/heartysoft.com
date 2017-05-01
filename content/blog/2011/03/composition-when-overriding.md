
+++
title = "Ninja Coding: Composition over Inheritance–Even when Overriding"
description = "We’ve all heard it a million times – composition is favourable to inheritance. But inheritance can s ..."
tags = [ ".NET", "C#", "Software Craftsmanship", "blog" ]
date = "2011-03-17 22:08:45"
slug = "composition-when-overriding"
+++
<p>We’ve all heard it a million times – composition is favourable to inheritance. But inheritance can sometimes come with its own charms. One of its main attractions is to do some grunt work in a base class and have the ability to override that behaviour in a derived class. Framework developers favour an abstract base class as it leaves the potential to add features in the future without breaking all client code. The usual approach towards to polymorphism is achieved by declaring a method (or property) as virtual in the base class and overriding in the derived one(s). And this usually works out quite well (if you do find yourself using inheritance). This would typically look like this:</p>  <p><pre class='brush:c#'>    
public class Command     
{     
&#160;&#160;&#160; public virtual void Validate()     
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var validator = ServiceLocator.Get&lt;Validator&gt;();     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; validator.Validate(this);     
&#160;&#160;&#160; }     
}     
    
public class SomeDerivedCommand : Command     
{     
&#160;&#160;&#160; public override void Validate()     
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; //some other validation logic     
&#160;&#160;&#160; }     
}     
</pre></p>  <p>While this works quite well in a lot of cases, one area where this is annoying is unit testing. Say you have a class that has the following method:</p>  <p><pre class='brush:c#'>    
public class SomeCommandHandler     
{     
&#160; public void Handle(SomeDerivedCommand command)     
&#160; {     
&#160;&#160;&#160;&#160;&#160; command.Validate();     
&#160;&#160;&#160;&#160;&#160; //do other stuff     
&#160; }     
}     
</pre></p>  <p>and you wished to unit test the Handle method. You first wish to test how the Handle method would behave if the command.Validate() failed. You have a number of options:</p>  <p>1. Mock a command with a mocking tool. This adds a dependency, might have more-than-desired syntax, usually requires the method to be mocked to be virtual or the reference be an interface.   <br />2. Actually create a valid command for the test and an invalid command for another test. This is tedious and more importantly, if the validation logic for the command changes, you’ll have to go back and change tests which are not testing the validation logic. This means brittle tests and is quite bad (usually).    <br />3. Write a manual mock.</p>  <p>Now option 3 usually involves writing a derived class and overriding. Since this is a few lines of code for a test, this is mostly avoided in favour of a tool like moq, fake-it-easy or similar. With one simple step we can make this much simpler.</p>  <h2>Functional Override</h2>  <p>Let’s refactor the code, but this time with a somewhat functional approach:</p>  <p><pre class='brush:c#'>   
public class Command    
{    
&#160;&#160;&#160; public Action Validate { get; protected internal set; }
  &#160;&#160;&#160; public Command()   
&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; Validate = _Validate;    
&#160;&#160;&#160; }
  &#160;&#160;&#160; private void _Validate()   
&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var validator = ServiceLocator.Get&lt;Validator&gt;();    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; validator.Validate(this);    
&#160;&#160;&#160; }    
}
  public class SomeDerivedCommand : Command   
{    
&#160;&#160;&#160; public SomeDerivedCommand()    
&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; Validate = _SomeOtherValidationLogic;    
&#160;&#160;&#160; }
  &#160;&#160;&#160; private void _SomeOtherValidationLogic()   
&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; //do other validation here    
&#160;&#160;&#160; }    
}
  public class SomeCommandHandler   
{    
&#160; public void Handle(SomeDerivedCommand command)    
&#160; {    
&#160;&#160;&#160;&#160;&#160; command.Validate();    
&#160;&#160;&#160;&#160;&#160; //do other stuff    
&#160; }    
}    
</pre></p>  <p>We haven’t changed much – just converted Validate in the command class to an Action as opposed to a virtual method. The derived class can still override the base class behaviour. If overriding behaviour is not something the derived class wants to do, it doesn’t have to do anything. </p>  <h2>Hang on…that’s more code…what’s the benefit?</h2>  <p>Let’s revisit the scenario where we were testing the SomeCommandHandler class’ Handle method. To create a valid command, we can simply do this:</p>  <p><pre class='brush:c#'>   
var stubCommand = new SomeDerivedCommand();    
stubCommand.Validate = () =&gt; { };    
</pre></p>  <p>And for an invalid command, we can do this:   <br /></p>  <p><pre class='brush:c#'>   
var stubCommand = new SomeDerivedCommand();    
stubCommand.Validate = () =&gt; { throw new SomeException(); };    
</pre></p>  <p>No mocking tools, no derived classes for stubbing out a parameter – so much nicer. And any code that’s calling Validate() will continue to build.</p>  <p>Another benefit you get is that you can now have a library of functional elements (Action, Func, Predicate) and hook them up in the base class and “override” them is derived classes simply by setting a property in the constructor. So even though you’re specifying default functionality in the base class, you have a more flexible approach when overriding behaviour.</p>  <h2>Gotchas</h2>  <p>If you haven’t noticed yet, the Validate Action has a protected internal setter. This means derived classes OR classes in the same assembly can set it. The reason I’m allowing internal mutator is because I would have the test project able to access the internal symbols in the main project. For this I would simply need to set [assembly:InternalsVisibleTo(..)]. That way, the test assembly will be able to set the Action. This does mean encapsulation is broken within the main assembly but judging by the fact that it’s composing behaviour, I’m willing to accept that. If you don’t like that, just make the mutator protected and not internal. You’ll lose the stubbing benefit but you will still get the benefits of composition (use reusable libraries of functional elements as mentioned in the last section). </p>
        