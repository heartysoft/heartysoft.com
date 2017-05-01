
+++
title = "Encrypted Hidden Inputs in ASP.NET MVC"
description = "I have written an update to this article that shows a better way of doing what is presented here. Th ..."
tags = [ "ASP.NET", "ASP.NET MVC", ".NET", "Security", "ASP.NET", "ASP.NET MVC", "Security", "blog" ]
date = "2010-02-25 07:27:09"
slug = "Encrypted-Hidden-Inputs-in-ASPNET-MVC"
+++
<h1><strong><font color="#ff0000">I have written an </font></strong><a href="http://www.heartysoft.com/post/2010/03/13/encrypted-hidden-with-salt.aspx"><strong><font color="#ff0000">update to this article</font></strong></a><strong><font color="#ff0000"> that shows a better way of doing what is presented here. The core concepts are pretty much the same and I would request you to read this one before the improved version.</font></strong></h1>  <p>There are many situations where we need to persist some data during the user's session. There are a few options in achieving this:</p>  <p>1. Session: Using the Session object will persist the data in server resources. By default, they are persisted to server RAM. Using Session is very simple as to persist, you just go Session[&quot;key&quot;] = &quot;value, and to retrieve, var val = Session[&quot;key&quot;] as [type]. There are a couple of problems in this approach, however. Firstly, it consumes server resources. If you anticipate large volumes of traffic, you can't store the data in the server's RAM cause it'll simply be too much of a resource hog. Second, if you're using a web farm setup where multiple physical machines (or virtual machines) are hosting your site, then data stored on one machine's RAM will not be available to the other machine. So, if requests from the same user go to different machines, complications are bound to arise. The workaround to this can be using a state server. That also needs setting it up and maintaining, load balancing etc. Furthermore, using a state server would result in a (however small) time delay where state information is persisted and retrieved during requests.</p>  <p>2. Cookies: Information can easily be persisted by using cookies. You can't rely on them though, as many users have them disabled – and they're very easy to disable.</p>  <p>3. Hidden Inputs: Using this method, you store the data as the value of hidden input controls. These values get posted during form submission and you can retrieve them easily. The data is stored in the html sent to the browser and thus does not consume any server resources. There's a slight amount of work required to store the value and retrieve it, but other than that, it's an ideal way to persist session specific data in a scalable way. We are going to explore this method in this article.</p>  <p><em><font color="#008000">I don't discuss ViewState and ControlState here as the discussion is about ASP.NET MVC and not Webforms.</font></em></p>  <h3>The Problem</h3>  <p>Although hidden inputs sound great, there's one major flaw. The user can just go to the browser's &quot;view source&quot; option and actually see what value is persisted. This may not be a problem if you're simply storing a user's colour choice for example, but for sensitive data (for example, some id) that needs to be hidden from prying eyes (of the user and / or eavesdroppers), it can be a major issue. </p>  <h3>The Solution</h3>  <p>We try to solve the security issue by creating an Html Helper which will encrypt the data to be stored in the hidden input control. When the data is posted back, we need to ensure that consuming the data in the hidden input control is no more difficult than using a plain hidden input. To achieve this, we create a custom controller factory.</p>  <h3>The Project</h3>  <p>Let's create a default ASP.NET MVC 2 Web Application in VS 2010. Name it &quot;HiddenEncryptDemo&quot;.</p>  <p><em><font color="#008000">I installed MVC 2 RC 2 before installing VS 2010 RC. If you have some other combination, slight changes may be necessary. </font></em></p>  <p>The first thing I'm going to do is to create a very simple Computer.cs class in the Model folder:</p>  <p><pre class='brush:c#'>    
public class Computer     
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; public string Setting { get; set; }     
&#160;&#160;&#160; }     
</pre></p>  <p>The reason I'm adding this class is to show that our approach will work not just on simple action parameters, but also on more complex types without a need for a separate model binder.</p>  <p>Next, add this to the Index.aspx view (in the /Views/Home folder):</p>  <p><pre class='brush:xml'>    
&lt;% Html.BeginForm(&quot;Something&quot;, &quot;Home&quot;, FormMethod.Post); %&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;%= Html.Hidden(&quot;computer.Setting&quot;, &quot;hello world!&quot;) %&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;input type=&quot;submit&quot; value='submit' /&gt;     
&lt;% Html.EndForm(); %&gt;     
</pre></p>  <p>After that, add the following method to the Home controller:</p>  <p><pre class='brush:c#'>    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; [HttpPost]     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; public ActionResult Something(Computer computer)     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; ViewData[&quot;Message&quot;] = computer.Setting;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; return View(&quot;Index&quot;);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; }     
</pre></p>  <p>Nothing fancy, we're just setting the ViewData[&quot;Message&quot;] to the computer.Setting value. Notice that in the code we added to Index.aspx, the form submitted to &quot;Something&quot; (the name of our action) and the name of the hidden input was &quot;computer.Setting&quot;. The default model binder will find a value for &quot;computer.Something&quot; in the request parameters and upon seeing that it can set the Setting property of the computer parameter, it's going to set computer.Setting to the value it found (in our case, &quot;hello world!&quot;). If you run the project, you should see the index page with a submit button. Clicking the button will result in a post to the server where the Something action will get called. The Something action will then set the ViewData[&quot;Message&quot;] and return the Index view. As such, you will see the words &quot;hello world!&quot; displayed on the page:</p>  <p><a href="/media/default/images/hello.png"><img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="hello" border="0" alt="hello" src="/media/default/images/hello_thumb.png" width="360" height="270" /></a> </p>  <p>On the initial Index page (or the page got after clicking the button), if you right click anywhere and select view source, you should see this (among other things):</p>  <p><a href="/media/default/images/hidden_2.png"><img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="hidden" border="0" alt="hidden" src="/media/default/images/hidden_thumb_2.png" width="718" height="75" /></a> </p>  <p>Notice how the helper added a hidden input control, set its id to &quot;computer_Setting&quot; and name to &quot;computer.Setting&quot;. The name is used as the key of the data in the post variables. Also notice how the value is &quot;hello world!&quot;. The value is clearly visible to anyone looking at the source. Our goal is to come up with a way to hide that value from clear sight while at the same time ensuring everything works as smoothly and easily as it just did. A bonus would be if no change was required in the controller's code (i.e. no specific attributes needed on the consuming action or controller, no special model binding needed for the parameter etc.) – it would be unobtrusive.</p>  <h3></h3>  <p></p>  <h3>The Encryption Provider</h3>  <p>The first thing we'll need is an encrypting mechanism that's easy to use. Create a new folder in the project called &quot;Helpers&quot;. [I know, I know, I should structure it better – but this is a demo :) ] Add a new interface called IEncryptString to the folder. The interface is very simple:</p>  <p><pre class='brush:c#'>    
public interface IEncryptString     
{     
&#160;&#160;&#160; string Encrypt(string value);     
&#160;&#160;&#160; string Decrypt(string value);     
&#160;&#160;&#160; string Prefix { get; }     
}     
</pre></p>  <p>Add an implementation of that interface called ConfigurationBasedEncryptionProvider:</p>  <p><pre class='brush:c#'>    
public class ConfigurationBasedStringEncrypter : IEncryptString     
{     
&#160;&#160;&#160; private static readonly ICryptoTransform _encrypter;     
&#160;&#160;&#160; private static readonly ICryptoTransform _decrypter;     
&#160;&#160;&#160; private static string _prefix;     
&#160;&#160;&#160; static ConfigurationBasedStringEncrypter()     
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; //read settings from configuration     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var key = ConfigurationManager.AppSettings[&quot;EncryptionKey&quot;];     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var useHashingString = ConfigurationManager.AppSettings[&quot;UseHashingForEncryption&quot;];     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; bool useHashing = true;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; if (string.Compare(useHashingString, &quot;false&quot;, true) == 0)     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; useHashing = false;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; } 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; var prefix = ConfigurationManager.AppSettings[&quot;EncryptionPrefix&quot;];    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; if (string.IsNullOrWhiteSpace(prefix))     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; _prefix = &quot;encryptedHidden_&quot;;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; }     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; byte[] keyArray = null; 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; if (useHashing)    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var hashmd5 = new MD5CryptoServiceProvider();     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; keyArray = hashmd5.ComputeHash(UTF8Encoding.UTF8.GetBytes(key));     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; hashmd5.Clear();     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; }     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; else     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; keyArray = UTF8Encoding.UTF8.GetBytes(key);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; } 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; TripleDESCryptoServiceProvider tdes = new TripleDESCryptoServiceProvider();    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; tdes.Key = keyArray;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; tdes.Mode = CipherMode.ECB;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; tdes.Padding = PaddingMode.PKCS7; 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; _encrypter = tdes.CreateEncryptor();    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; _decrypter = tdes.CreateDecryptor(); 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; tdes.Clear();    
&#160;&#160;&#160; } 
  &#160;&#160;&#160; #region IEncryptionSettingsProvider Members 
  &#160;&#160;&#160; public string Encrypt(string value)    
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var bytes = UTF8Encoding.UTF8.GetBytes(value);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var encryptedBytes = _encrypter.TransformFinalBlock(bytes, 0, bytes.Length);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var encrypted = Convert.ToBase64String(encryptedBytes);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; return encrypted;     
&#160;&#160;&#160; } 
  &#160;&#160;&#160; public string Decrypt(string value)    
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var bytes = Convert.FromBase64String(value);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var decryptedBytes = _decrypter.TransformFinalBlock(bytes, 0, bytes.Length);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var decrypted = UTF8Encoding.UTF8.GetString(decryptedBytes);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; return decrypted;     
&#160;&#160;&#160; } 
  &#160;&#160;&#160; public string Prefix    
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; get { return _prefix; }     
&#160;&#160;&#160; } 
  &#160;&#160;&#160; #endregion 
  }    
</pre></p>  <p>You can see that in the class' static constructor, I'm reading in some configuration settings. While this goes against &quot;convention over configuration&quot;, extensibility isn't really needed in terms of encryption keys. I though about using the machine key as the encryption key, but considering sever farm environments and other issues that could arise, I just chose to go with an app setting instead. Anyway, you can see that I'm using the configuration data to set up a couple of ICryptoTransforms that will be used for encrypting and decrypting. The class has two instance methods, Encrypt and Decrypt, which do pretty much what you'd expect. I'll explain the prefix part later.</p>  <h3>The Html Helper</h3>  <p>With the encryption provider out of the way, let's add our Html Helper. Create a new file called InputExtensions.cs in our Helpers folder and add the following code:</p>  <p><pre class='brush:c#'>    
namespace HiddenEncryptDemo.Helpers     
{     
&#160;&#160;&#160; using System;     
&#160;&#160;&#160; using System.Web.Mvc;     
&#160;&#160;&#160; using System.Web.Mvc.Html; 
  &#160;&#160;&#160; public static class InputExtensions    
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; public static HeartysoftHtmlHelper Heartysoft(this HtmlHelper helper)     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; return new HeartysoftHtmlHelper(helper);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; }     
&#160;&#160;&#160; } 
  &#160;&#160;&#160; public partial class HeartysoftHtmlHelper    
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; private readonly HtmlHelper _helper;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; private static readonly IEncryptString _encrypter = new ConfigurationBasedStringEncrypter(); 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; public HeartysoftHtmlHelper(HtmlHelper helper)    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; _helper = helper;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; }     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; public MvcHtmlString EncryptedHidden(string name, object value)     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; if (value == null)     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; value = string.Empty;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; } 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; string strValue = value.ToString();    
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var encryptedValue = _encrypter.Encrypt(strValue);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var encodedValue = _helper.Encode(encryptedValue);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var newName = string.Concat(_encrypter.Prefix, name); 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; return _helper.Hidden(newName, encodedValue);    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; }     
&#160;&#160;&#160; }     
}     
</pre></p>  <p>Here, I'm firstly creating a &quot;Heartysoft&quot; helper which in turn has an EncryptedHidden method. This approach ensures that my helpers don't conflict with other helpers and in the view, I can do this: Html.Heartysoft().HiddenEncrypt(…). The EncryptedHidden helper is pretty simple. It takes the name and value to be used and encrypts the value. It also adds the prefix to the name. Remember our EncryptionProvider had a Prefix property? Here's where it comes in. So, if you pass in &quot;computer.Name&quot;, the hidden input would have a name of &quot;encryptedHidden_computer.Name&quot; and id of &quot;encryptedHidden_computer_Name&quot;. Of course, if you specify a different prefix in configuration, that would have been used instead. The prefix is used to determine which inputs need decrypted down the line. At the end of the EncryptedHidden method, we're using the default Html helper's Hidden method to generate the &lt;input&gt; tag.</p>  <h3></h3>  <h3>Configuration</h3>  <p>Go to web.config and ensure that under &lt;configuration&gt;, the following entries are present:</p>  <p><pre class='brush:xml'>    
&lt;appSettings&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;add key=&quot;EncryptionKey&quot; value=&quot;asdjahsdkhaksj dkashdkhak sdhkahsdkha kjsdhkasd&quot;/&gt;     
&lt;/appSettings&gt;     
</pre></p>  <h3>The View</h3>  <p>At this point, go to Index.aspx and change the html of the form to:</p>  <p><pre class='brush:xml'>    
&lt;% Html.BeginForm(&quot;Something&quot;, &quot;Home&quot;, FormMethod.Post); %&gt;     
&#160;&#160;&#160;&#160;&#160;&#160; &lt;%= Html.Heartysoft().EncryptedHidden(&quot;computer.Setting&quot;, &quot;hello world!&quot;) %&gt;     
&#160;&#160;&#160;&#160;&#160;&#160; &lt;%--&lt;%= Html.Hidden(&quot;computer.Setting&quot;, &quot;hello world!&quot;) %&gt;--%&gt;     
&#160;&#160;&#160;&#160;&#160;&#160; &lt;input type=&quot;submit&quot; value='submit' /&gt;     
&#160;&#160; &lt;% Html.EndForm(); %&gt;     
</pre></p>  <p>I've just commented out the old helper and used our new helper. If you run the page and click the button, you should see this:</p>  <p><a href="/media/default/images/encrypted-res.png"><img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="encrypted-res" border="0" alt="encrypted-res" src="/media/default/images/encrypted-res_thumb.png" width="382" height="258" /></a> </p>  <p>Notice that the message has disappeared. Viewing the html source, you should see that the html tag now looks like this:</p>  <p><pre class='brush:xml'>    
&lt;input id=&quot;encryptedHidden_computer_Setting&quot; name=&quot;encryptedHidden_computer.Setting&quot; type=&quot;hidden&quot; value=&quot;TA9p71fDeNMfjlCPGOtK9Q==&quot; /&gt;     
</pre></p>  <p>Notice that although the &quot;hello world!&quot; value has been encrypted, the name has changed to &quot;encryptedHidden_computer.Setting&quot;. When the form is posted back after the button click, the input parameter is thus &quot;encryptedHidden_computer.Setting&quot;. Since there's no longer a &quot;computer.Setting&quot; input parameter, the model binder doesn't assign the computer parameter's Setting property with any value. The ViewData[&quot;Message&quot;] thus stays empty and we se a blank on the result page. </p>  <p>So, our final challenge is to ensure that the model binder finds the original value. This will be done through a custom controller factory.</p>  <h3></h3>  <h3>The Controller Factory</h3>  <p>Our controller factory will look for any encrypted input parameters, decrypt them and add them to RouteData using their original names so that they're available to the model binder. Add a class called DecryptingControllerFactory:</p>  <p><pre class='brush:c#'>    
public class DecryptingControllerFactory : DefaultControllerFactory     
{     
&#160;&#160;&#160; private static IEncryptString _decrypter = new ConfigurationBasedStringEncrypter(); 
  &#160;&#160;&#160; public override IController CreateController(System.Web.Routing.RequestContext requestContext, string controllerName)    
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var parameters = requestContext.HttpContext.Request.Params;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var encryptedParamKeys = parameters.AllKeys.Where(x =&gt; x.StartsWith(_decrypter.Prefix)).ToList();     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; foreach (var key in encryptedParamKeys)     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var oldKey = key.Replace(_decrypter.Prefix, string.Empty);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var oldValue = _decrypter.Decrypt(parameters[key]);     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; requestContext.RouteData.Values[oldKey] =&#160; oldValue;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; } 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; return base.CreateController(requestContext, controllerName);    
&#160;&#160;&#160; }     
}     
</pre></p>  <p>As you can see, we're identifying encrypted input parameters by checking which keys start with our encryption prefix. For each match, we get the value by decrypting and the original name by removing the prefix. We're not really changing the generated controller in any way – the controller factory just acts as a point in the request-response cycle where we can add parameters that are to be consumed by the model binder. </p>  <p>The last change you need to do is open up the global.asax.cs file and ensure the Application_Start looks like this:</p>  <p><pre class='brush:c#'>    
protected void Application_Start()     
{     
&#160;&#160;&#160; AreaRegistration.RegisterAllAreas();     
&#160;&#160;&#160; RegisterRoutes(RouteTable.Routes);     
&#160;&#160;&#160; ControllerBuilder.Current.SetControllerFactory(typeof(DecryptingControllerFactory));     
}     
</pre></p>  <p>We're just registering our controller factory on that last line.</p>  <h3>Running the Page</h3>  <p>We'll need to stop the dev server (if you're using IIS, simply restart it). We need to do this because we made changes to the global.asax.cs page. Right click the dev server's icon in your traybar and click stop:</p>  <p><a href="/media/default/images/stop-server.png"><img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="stop-server" border="0" alt="stop-server" src="/media/default/images/stop-server_thumb.png" width="243" height="180" /></a> </p>  <p>You can now simply hit ctrl + F5 in VS to run the page. Click the button and you should see this:</p>  <p><a href="/media/default/images/result.png"><img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="result" border="0" alt="result" src="/media/default/images/result_thumb.png" width="348" height="263" /></a> </p>  <p>Finally, view the html source in the browser, and you should see that the hidden input's value is encrypted:</p>  <p><pre class='brush:xml'>    
&lt;input id=&quot;encryptedHidden_computer_Setting&quot; name=&quot;encryptedHidden_computer.Setting&quot; type=&quot;hidden&quot; value=&quot;TA9p71fDeNMfjlCPGOtK9Q==&quot; /&gt;     
</pre></p>  <p>That means that our helper has encrypted the value for us and on postback, the controller factory is decrypting the value and adding it to the input parameters so that the model binder can correctly do its thing.</p>  <h3></h3>  <h3>Possible Questions</h3>  <p><strong>1. Phil Haack already has a blog post stating how to add action method parameters here:      <br /><a href="http://haacked.com/archive/2010/02/21/manipulating-action-method-parameters.aspx">http://haacked.com/archive/2010/02/21/manipulating-action-method-parameters.aspx</a>       <br />Why didn't you use that method to add the input parameter?       <br /></strong>The method described in that post uses an action filter. There are two downsides to that. First, you'd need to go about and decorating your controllers and actions with some [Decrypt] attribute. That's obtrusive. Second-and-more-important, action filters run after model binding. That would work for simple parameters for action methods, but for complex parameters (i.e. viewmodels) or parameters that need complex model binding, that simply won't work. My approach here adds the input parameter before the model binding stage and thus, the model binders can access the correct values without changing anything.</p>  <p><strong>2. Why didn't you use a ModelBinder attribute for decryption?      <br /></strong>That could have worked, but would have required you to use the model binder whenever you intended to use the Html.Heartysoft().EncryptedHidden() helper. That's obtrusive and forgetting to set an attribute could have caused nightmares. Furthermore, your other model binders would not have recognised the values. The approach outlined here will work with any and all existing model binders.</p>  <p><strong>3. Is the data really secure?      <br /></strong>The value sent to the browser is encrypted using a key. The key is never sent to the browser. As long as the key is secure, the data is as secure as the encryption is.</p>  <p><strong>4. Your code structure is crappy.      <br /></strong>It's a demo…deal with it ;) The code is pretty straight forward and you can use your our design patterns and abstractions to make it more your-framework-friendly.</p>  <p><strong>5. I changed the key but the change isn't &quot;happening&quot;. What gives?      <br /></strong>The encryption key and whether to use hashing or not settings are read during the static constructor of our ConfigurationBasedEncryptionProvider. If you need to change the values, restart the server. I figured you didn't need to change these at runtime and thus put them in the static constructor so that they are read in only once.</p>  <p><strong>6. Why a ControllerFactory? Why not in Begin_Request in global.asax.cs?      <br /></strong>It seems that adding the parameters in Begin_Request doesn't seem to persist further down the line. Meaning, I tried that and the new parameters added seem to disappear. Hence the controller factory. Adding the parameters at that stage seems to allow the model binder access to the parameters without any problems. Perhaps a custom ActionInvoker would also have worked, but would have needed setting on each controller – or in a custom controller factory. If the latter, then why bother with the ActionInvoker when the factory can add the parameters itself?</p>  <p><strong>7. Why do you add the parameters to RouteData and not Request[…] or Request.Params[…] or Request.Form[…] etc.?      <br /></strong>Coz those are readonly.</p>  <h3></h3>  <h3>Source Code</h3>  <p><a href="http://cid-9d8a7728356393b1.skydrive.live.com/self.aspx/.Public/code/HiddenEncryptDemo.zip">Download</a></p>  <p><a href="http://www.dotnetkicks.com/kick/?url=http%3a%2f%2fwww.heartysoft.com%2fpost%2f2010%2f02%2f25%2fEncrypted-Hidden-Inputs-in-ASPNET-MVC.aspx"><img border="0" alt="kick it on DotNetKicks.com" src="http://www.dotnetkicks.com/Services/Images/KickItImageGenerator.ashx?url=http%3a%2f%2fwww.heartysoft.com%2fpost%2f2010%2f02%2f25%2fEncrypted-Hidden-Inputs-in-ASPNET-MVC.aspx" /></a>&#160; <a href="http://dotnetshoutout.com/Heartysoftcom-Encrypted-Hidden-Inputs-in-ASPNET-MVC" rev="vote-for"><img style="border-right-width: 0px; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" alt="Shout it" src="http://dotnetshoutout.com/image.axd?url=http%3A%2F%2Fwww.heartysoft.com%2Fpost%2F2010%2F02%2F25%2FEncrypted-Hidden-Inputs-in-ASPNET-MVC.aspx" /></a></p>  <p></p>  <p><em>---------------------------------------------------------</em></p>  <p><em><font color="#ff0000">Questions&#160; and comments relating to this article are welcome. Comments completely unrelated to the article and posted with the sole intention of putting your link here are not.        <br /></font></em><em><font color="#ff0000">       <br />If you spam, your comment will not be approved, will be deleted and your IP blocked. I maintain my site almost daily and such comments – even if they pass the spam filter – will get removed as soon as possible. If this gets too tedious, I may disable comments entirely. Please don't ruin it for everybody else.</font></em></p>  <p><em>---------------------------------------------------------</em></p>
        