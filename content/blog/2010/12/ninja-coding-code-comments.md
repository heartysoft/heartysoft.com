
+++
title = "Ninja Coding: Code Comments and Self Explanatory Code"
description = "It’s been a while since I last blogged and thought I would get back into the habit by starting off a ..."
tags = [ "NinjaCoding", "Software Craftsmanship", "blog" ]
date = "2010-12-20 07:06:41"
slug = "ninja-coding-code-comments"
+++
<p>It’s been a while since I last blogged and thought I would get back into the habit by starting off a new category of posts. The title for this category is “Ninja Coding” and will cover topics from software craftsmanship and better design to very simple tricks and tricks. Today, we will be visiting code comments.</p>  <p>When I was starting out with programming many many moons ago, the wiser elders I learnt from and the books I read seemed to suggest proper professional code is always properly commented so that a new user (or yourself coming back a few months later) could easily understand the intent of the code. It is a practice that generally seems like a good idea and a lot of people practice it. And I was one of them.</p>  <p>I guess there are many examples of companies that comment their code aggressively. Few even talk of Microsoft having comments in their code and tools like StyleCop that bring up issues if code isn’t commented “properly”. Lately though, I’m veering more and more away from comments.</p>  <p>Have I gone mad? Probably, but that’s a different issue ;)</p>  <h1></h1>  <h2>Why I don’t like comments</h2>  <p>The main reason I’m trying to avoid commenting my code is the concept of self explanatory code. Code should be self explanatory – anyone reading the code should understand what’s going on (given some domain knowledge, of course). Relying on comments to explain what’s going on is hardly a good idea. Code is going to change far more often than comments getting updated. No matter how vigilant your team is, this is bound to happen. At best, this can result in slightly outdated comments. At worst, stale comments may be entirely misleading to what the code is actually doing. Let’s take a look at a quick example from code from a certain company based in Redmond. XmlDocument has a Load method which has a number of overloads, one them being:</p>  <p>&#160;</p>  <p></p>  <p><a href="/media/default/images/XmlDocument.jpg"><img title="XmlDocument" style="border-top-width: 0px; display: inline; border-left-width: 0px; border-bottom-width: 0px; border-right-width: 0px" height="111" alt="XmlDocument" src="/media/default/images/XmlDocument_thumb.jpg" width="685" border="0" /></a> </p>  <p>The parameter name suggests it’s looking for a file whereas the comment immediately below it specifies a URL. Only by looking at the detailed comment for filename can we ascertain that the parameter can be either. I’m forced to have to read the entire comment to have to know what’s going on. I’d rather not have to. But this is only part of the problem here. Microsoft later introduced Linq to Xml. The new API also has a Load method and one of its (many) overloads look like this:</p>  <p><a href="/media/default/images/XDocument.jpg"><img title="XDocument" style="border-top-width: 0px; display: inline; border-left-width: 0px; border-bottom-width: 0px; border-right-width: 0px" height="139" alt="XDocument" src="/media/default/images/XDocument_thumb.jpg" width="559" border="0" /></a> </p>  <p>Hmm…this time the parameter name reads uri and the comment reads file. The detailed comment this time is slightly less useful – a URI string that references the file. Granted all Xml documents can be thought of as files but the obvious lack of consistency is what troubles me the most. Will it through an exception if a parameter named uri is not an http uri? Will it through if a parameter named filename is passed an http uri? In cases like this, the only real solution seems to be some throwaway code to actually test what’s going on. Had there been methods like LoadFromFile or LoadFromUri, it would have been much more obvious.</p>  <h2>So, err…no comments?</h2>  <p>Please note, however, I’m not advocating simply removing all comments. I’m suggesting that self explanatory code is better than using comments to convey intent. If the code itself isn’t self explanatory, then having no comments is far far worse. Let’s go over another quick example, this time from CTP5 of Entity Framework Code First. The DbContext class’s Database property has a very nice CreateIfNotExists method. While the method itself is a nice feature, let’s have a look at its signature:</p>  <p><a href="/media/default/images/Create.jpg"><img title="Create" style="border-top-width: 0px; display: inline; border-left-width: 0px; border-bottom-width: 0px; border-right-width: 0px" height="101" alt="Create" src="/media/default/images/Create_thumb.jpg" width="224" border="0" /></a> </p>  <p>Hmm..the method returns a bool. (And there are no detailed comments either explaining what that bool is for). What does the bool represent? Does it signify that there was not a database and it successfully created it? Does it mean that there is already a database present no recreation is unnecessary? Would it return false if there was already a database present or if an exception was thrown? Will a failure to create a database throw an exception, or simply return false? [On another note, Database.Delete() also returns a bool. Will it do pokemon exception handling and return false in case of errors?] So many questions…and there aren’t comments explaining anything. Will these comments be added in RTM? In that case, will a team of people (who might not have been the developers) go through the code and do their best to put comments in? And even then, will the comments explain everything accurately? And even then, how much will the programmer have to read after hitting “Database.” to understand what his code is going to do? </p>  <p>In this case, the code is not conveying intent / meaning and there are no comments either. Comments for intellisense might be added in the RTM (I mean, it is a CTP still, right?). And even if they aren’t, there are going to be some official guidance, samples etc. that will show users how to use the library. While this is not ideal, it is something Microsoft can get away with (most of the time) – their libraries are going to be used in the same manner by developers worldwide and implicit customs will be (or become) the “standard”. Anyone learning to use these libraries will put themselves through the pain of having to learn these idioms. That is a luxury the creators of .NET have – it is not one developers in companies should indulge themselves in. </p>  <h2>An example of self explanatory code</h2>  <p>Let us look at the following code:</p>  <p><pre class='brush:c#'>   
    
public class BusinessException : Exception    
{    
&#160;&#160;&#160; public BusinessException(string message) : base(message)    
&#160;&#160;&#160; {    
&#160;&#160;&#160; }    
}    
    
public class Calculator    
{    
&#160;&#160;&#160; public double Divide(double a, double b)    
&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; //is the divisor 1    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; if (b == 1)    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; //divisor is one, simply return the dividend    
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; return a;    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; }    
    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; //is the divisor zero    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; if (b == 0)    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; //throw business exception for divide by zero    
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; throw new BusinessException(&quot;divisor cannot be zero&quot;);    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; }    
    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; //divide for other divisors    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; return a / b;    
&#160;&#160;&#160; }    
}    
    
</pre> </p>  <p>Never mind the trivialness of the code or how better it could have been implemented mathematically. The code is quite simple, yet it requires a certain knowledge on the reader’s part to understand what’s going on (what’s the meaning of 1 and 0?). The code may be refactored as follows:</p>  <p><pre class='brush:c#'>   
public class Calculator    
{    
&#160;&#160;&#160; public double Divide(double dividend, double divisor)    
&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; if (ShouldReturnTheDividend(divisor))    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; return dividend;    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; }    
    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; if (IsAttemptToDivideByZero(divisor))    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; ThrowDivideByZeroException();    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; }    
    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; return DivideForOtherDivisors(dividend, divisor);    
&#160;&#160;&#160; }    
    
&#160;&#160;&#160; private static bool ShouldReturnTheDividend(double divisor)    
&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; return divisor == 1;    
&#160;&#160;&#160; }    
    
&#160;&#160;&#160; private static bool IsAttemptToDivideByZero(double divisor)    
&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; return divisor == 0;    
&#160;&#160;&#160; }    
    
&#160;&#160;&#160; private static void ThrowDivideByZeroException()    
&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; throw new BusinessException(&quot;divisor cannot be zero&quot;);    
&#160;&#160;&#160; }    
&#160;&#160;&#160; 
&#160;&#160;&#160; private static double DivideForOtherDivisors(double dividend, double divisor)    
&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; return dividend / divisor;    
&#160;&#160;&#160; }    
}    
</pre></p>  <p>The refactorings are quite simple, yet see how much readable the code in the Divide() method has become. The reader can simply read through the code and understand what’s going on. If they need to alter the code somewhat, it’d be clear which portion is doing what. And when changes are made, nobody has to worry about updating comments.</p>  <h2>So, err…no comments at all?</h2>  <p>There are situations when comments can be useful. Every public API / library needs to be understandable and usable. How the API should be used needs to be conveyed to the user of the API / library. This requires communication and this can be done through documentation. Documentation does not necessarily mean separate files. In fact, I can think of many things that can be used for this task. First and foremost, the names of classes, methods, parameters etc. can convey intent. This would be the very first stage of documentation. Effort should be taken so that this stage is enough to convey all intent. Another form of documentation could be a suite of unit tests. They themselves are an example of how the code is intended to be used. Examples can also serve as documentation. Lastly, separate files can also be used to describe the API / library. A feature of VS is xml comments – these can be used to generate separate files (and also intellisense) for documenting. If you do find that separate files are the only way of getting your message across, only then would those xml comments come in handy. Note that these comments are only for generating separate documentation and will need to be kept in sync. Yes, they can also be used to generate intellisense, but even then – if the name of a class, method or entity can reflect the intent, further reading of intellisense descriptions would become redundant. Forcing other people to read your intellisense comments is making others pay for your sloppiness.</p>  <p>Simply put, if you feel the need for comments in explaining to your team (or future self) what you’re code is doing, consider refactoring the code by renaming stuff or moving code into methods with names that reflect what you wanted to convey via the comment. For public APIs where you’re trying to communicate something to a third party that will not have the source code, try to name classes, methods, parameters etc. in such a manner that comments won’t be necessary. If the intent is still ambiguous, only then succumb to commenting and ensure the API is properly documented – via xml documentation, separate documents, examples or simply a suite of unit tests.</p>  <h2>What about StyleCop</h2>  <p>StyleCop is a tool that can be used to enforce coding styles in a team. It is configurable and by default it enforces the rules in accordance with Microsoft’s Framework Design Guidelines. While that’s all good, you need to think about whether or not you’re building a framework for public consumption (like the .NET framework). And also the fact that your team might not have the resources to keep comments updated and in sync(and even if you do, things might slip through the net – as we saw in the xml example earlier). StyleCop is simply a tool to ensure a certain style of coding within a team. Personally, if it were a choice between self explanatory code and more points on the default StyleCop rules, I’d choose self explanatory code any day. A better option might be to set up StyleCop rules to enforce no comments in non-public code!</p>
        