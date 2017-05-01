
+++
title = "How to Send Emails from .Net [example uses GMail SMTP]"
description = "We frequently see questions about sending emails using .Net in the asp.net forums. The process of se ..."
tags = [ ".NET", "ASP.NET", "blog" ]
date = "2009-04-28 05:10:00"
slug = "How-to-Send-Emails-from-Net-example-uses-GMail-SMTP"
+++
<p>We frequently see questions about sending emails using .Net in the asp.net forums. The process of sending mail is the same for Windows apps and asp.net websites as the same .Net classes are used. The process can be slightly shortened by specifying default SMTP settings in the web.config or app.config file. Here, I&rsquo;m showing the full version of the code and it does not rely on any configuration settings. The code also specifies unicode encoding for the subject and body.</p>
<p><pre class='brush:c#'>
using System.Net.Mail;
using System.Net;

//Create the mail message&nbsp;
MailMessage mail = new MailMessage();&nbsp;
mail.Subject = "Subject";&nbsp;
mail.Body = "Main body goes here";

//the displayed "from" email address
mail.From = new System.Net.Mail.MailAddress("you@live.com");&nbsp;
mail.IsBodyHtml = false;&nbsp;
mail.BodyEncoding = System.Text.Encoding.Unicode;&nbsp;
mail.SubjectEncoding = System.Text.Encoding.Unicode;

//Add one or more addresses that will receive the mail&nbsp;
mail.To.Add("me@live.com");

//create the credentials
NetworkCredential cred = new NetworkCredential(
"you@live.com", //from email address of the sending account
"password"); //password of the sending account

//create the smtp client...these settings are for gmail&nbsp;
SmtpClient smtp = new SmtpClient("smtp.gmail.com");&nbsp;
smtp.UseDefaultCredentials = false;&nbsp;
smtp.EnableSsl = true;

//credentials (username, pass of sending account) assigned here&nbsp;
smtp.Credentials = cred; &nbsp;
smtp.Port = 587;

//let her rip&nbsp;
smtp.Send(mail);
</pre></p>
<p align="left"><em></em></p>
<p align="left">Hope that helps.</p>
<p align="left"><strong>EDIT: I just added the namespaces. MailMessage exists in both System.Net.Mail and System.Web.Mail. System.Web.Mail has been deprecated and you should use System.Net.Mail.</strong></p>
        