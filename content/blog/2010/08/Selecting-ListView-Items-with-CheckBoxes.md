
+++
title = "Selecting ListView Items with CheckBoxes"
description = "Often we need to enable users to select multiple items from a ListView control and do a batch operat ..."
tags = [ ".NET", "ASP.NET", "blog" ]
date = "2010-08-14 09:57:28"
slug = "Selecting-ListView-Items-with-CheckBoxes"
+++
<p>Often we need to enable users to select multiple items from a ListView control and do a batch operation on them. While the ListView control does enable selection, it only supports selecting one row (item) at a time. This article shows a nice, easy and reusable way to enable multiple selection with checkboxes.</p>  <h3>The Project</h3>  <p>I've created a very simple &quot;Empty ASP.NET WebForms Application&quot; and added a standard Default.aspx page. I've created an App_Data folder and added a database called PeopleDB.mdf into it. The database consists of one simple table holding the Id, Name and AwesomePoints of a number of people* and also if they are attending some event. We want to list them in a simple ListView with pagination, and be able to select some of them and set all of the selected people's AwesomePoints in one go. We also want to be able to select some other people and set the boolean stating whether or not they're attending.</p>  <p><em>*these people are purely fictional and any resemblance to real people are entirely coincidental ;)</em></p>  <h3>The Page Layout</h3>  <p>The page layout is fairly simple. It's ugly and OMG I'm using tables for layout – help – the Earth is gonna explode…yeah right. At least it's apparent what needs doing. We show a list of people and provide checkboxes for each row to select the person for either batch update of their AwesomePoints or batch update of their IsAttending value. At the end of the page, we provide controls to actually carry out the batch updates. Here's the page body's markup so far:</p>  <p><pre class='brush:xml'>    
    
&lt;body&gt;     
&#160;&#160;&#160; &lt;form id=&quot;form1&quot; runat=&quot;server&quot;&gt;     
&#160;&#160;&#160; &lt;div&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;asp:ListView runat=&quot;server&quot; DataKeyNames='Id' ID='lvPeople' ItemPlaceholderID='ph1'&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;LayoutTemplate&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;table&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;thead&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;tr&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;th&gt;Name&lt;/th&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;th&gt;Awesome Points&lt;/th&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;th&gt;Is Attending&lt;/th&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;th&gt;Select for Updating Awesome Points&lt;/th&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;th&gt;Select for Updating Attendee Status&lt;/th&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/tr&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/thead&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;tbody&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;asp:PlaceHolder runat=&quot;server&quot; ID='ph1' /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/tbody&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/table&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/LayoutTemplate&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;ItemTemplate&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;tr&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;td&gt;&lt;%# Eval(&quot;Name&quot;) %&gt;&lt;/td&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;td&gt;&lt;%# Eval(&quot;AwesomePoints&quot;) %&gt;&lt;/td&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;td&gt;&lt;%# Eval(&quot;IsAttending&quot;) %&gt;&lt;/td&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;td&gt;&lt;asp:CheckBox runat=&quot;server&quot; ID='chkAwesomePoints' /&gt;&lt;/td&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;td&gt;&lt;asp:CheckBox runat=&quot;server&quot; ID='chkAttending' /&gt;&lt;/td&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/tr&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/ItemTemplate&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/asp:ListView     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;div&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;asp:DataPager ID=&quot;DataPager1&quot; runat=&quot;server&quot; PagedControlID='lvPeople' PageSize='5'&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;Fields&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;asp:NumericPagerField /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/Fields&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/asp:DataPager&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/div&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;br /&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;table border='none'&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;tbody&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;tr&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;td&gt;Set Awesome Points to: &lt;asp:TextBox runat=&quot;server&quot; ID='txtAwesomePoints' /&gt;&lt;/td&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;td&gt;&lt;asp:Button Text=&quot;Update Awesome Points&quot; runat=&quot;server&quot; /&gt;&lt;/td&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/tr&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;tr&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;td&gt;Set Attendee Status to: &lt;asp:CheckBox runat=&quot;server&quot; ID='chkAttending' /&gt;&lt;/td&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;td&gt;&lt;asp:Button Text=&quot;Update Attending Status&quot; runat=&quot;server&quot; /&gt;&lt;/td&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/tr&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/tbody&gt;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; &lt;/table&gt;     
&#160;&#160;&#160; &lt;/div&gt;     
&#160;&#160;&#160; &lt;/form&gt;     
&lt;/body&gt;     
    
</pre></p>  <p>Notice that I've set the DataKeyNames fo the ListView to Id. I'll explain this part later.</p>  <h3></h3>  <h3></h3>  <h3>The DataSource</h3>  <p>We have the ListView, we now need to add a datasource. I could easily use a SqlDataSource or whatever else I want. But since I'll be updating the database as well, I'll just add an Entity Framework model to the project (and use an EntityDataSource control to populate the ListView). Right click the project in solution explorer and select &quot;Add New Item&quot;. I chose ADO.NET Entity Data Model from the popup and name it PeopleModel.edmx.</p>  <p><a href="/media/default/images/1_adding_EF.png"><img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="1_adding_EF" border="0" alt="1_adding_EF" src="/media/default/images/1_adding_EF_thumb.png" width="831" height="577" /></a> </p>  <p>From the next window, I select &quot;Generate From Database&quot;. I get an option to select the connection to be used. Since I don't have one to the mdf file in App_Data, I select &quot;New Connection&quot;. I ensure that the &quot;Microsft SQL Serv Database File (SqlClient)&quot; is selected and I &quot;Browse&quot; to my .mdf file in the App_Code folder. You could obviously use other databases and you'd configure the connection here. Finishing this dialog takes you back to the previous one, where you can click &quot;Next&quot;. </p>  <p><a href="/media/default/images/2_EF_connection.png"><img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="2_EF_connection" border="0" alt="2_EF_connection" src="/media/default/images/2_EF_connection_thumb.png" width="633" height="565" /></a> </p>  <p>The next window gives you options of selecting a subset or all of the tables, sprocs and views in the target database. We're only interested in the People table, so select it and click &quot;Finish&quot;.</p>  <p><a href="/media/default/images/3_select_table.png"><img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="3_select_table" border="0" alt="3_select_table" src="/media/default/images/3_select_table_thumb.png" width="570" height="509" /></a> </p>  <p>And that's it – our EF4 model is done. Let's now add an EntityDataSource to the page.To do this, add the following markup right after the ListView:</p>  <p><pre class='brush:xml'>    
    
&lt;asp:EntityDataSource runat=&quot;server&quot; ID='edsPeople'     
&#160;&#160;&#160; ConnectionString=&quot;name=PeopleDBEntities&quot;     
&#160;&#160;&#160; DefaultContainerName=&quot;PeopleDBEntities&quot; EnableFlattening=&quot;False&quot;     
&#160;&#160;&#160; EntitySetName=&quot;People&quot;&gt;&lt;/asp:EntityDataSource&gt;     
    
</pre></p>  <p>(If you're using the webforms designer to hook up the datasource, be sure to compile the project before attempting to do so. Without a recompile, the required metadata would not be available to the user.)</p>  <p>Next, set the DataSourceId of the ListView to match the Id of the EntityDataSource:</p>  <p><pre class='brush:xml'>    
    
&lt;asp:ListView runat=&quot;server&quot; ID='lvPeople' DataKeyNames='Id' ItemPlaceholderID='ph1'<strong> DataSourceID='edsPeople'</strong>&gt;     
    
</pre></p>  <p>At this point, we can run the page to see this:</p>  <p><a href="/media/default/images/4_initial_results.png"><img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="4_initial_results" border="0" alt="4_initial_results" src="/media/default/images/4_initial_results_thumb.png" width="741" height="253" /></a></p>  <h2></h2>  <h3>&#160;</h3>  <h3>The ListView Extension</h3>  <p>Add a folder called Extensions and in it, create a class called ListViewExtensions with the following code:</p>  <p><pre class='brush:c#'>    
    
public static class ListViewExtensions     
{     
&#160;&#160;&#160; public static List&lt;DataKey&gt; GetSelectedDataKeys(this ListView control, string checkBoxId)     
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; return control.Items.Where(x =&gt; IsChecked(x, checkBoxId))     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; .Select(x =&gt; control.DataKeys[x.DisplayIndex])     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; .ToList();     
&#160;&#160;&#160; } 
  &#160;&#160;&#160; private static bool IsChecked(ListViewDataItem item, string checkBoxId)    
&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var control = item.FindControl(checkBoxId) as CheckBox;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; if (control == null)     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; {     
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; return false;     
&#160;&#160;&#160;&#160;&#160;&#160;&#160; } 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; return control.Checked;    
&#160;&#160;&#160; }     
}     
    
</pre></p>  <p>This class is the heart of what we're trying to do. It basically finds the checkbox with the Id passed in and returns the DataKey of every item in the ListView that has said checkbox checked. Remember the DataKeyNames property of the ListView that we set earlier? This is where that comes in. You can pass the names of the properties of each data item that you wish to have access to later on to the DataKeyNames. In our case, we only need the Id property, as such we passed in &quot;Id&quot;. If for some reason, we wanted the AwesomePoints property too, we'd pass in &quot;Id, AwesomePoints&quot;. Notice that our extension method returns a list of DataKeys. What is a DataKey? It's basically an object that holds the values of the properties mentioned in DataKeyNames. It has two properties – Value and Values. Value hold the value of the first data key property. So, if we had passed in &quot;Id, AwesomePoints&quot;, Value would hold the value of the Id property for the associated item. Values on the other hand has an ordered dictionary. You would then find the value of the Id property in Values[0] and the value of the AwesomePoints property in Values[1] and so on.</p>  <h3>Putting it to Work</h3>  <p>We'll now add two handlers for the two button click events:</p>  <p><pre class='brush:c#'>   
    
protected void UpdateAwesomePointsButton_Click(object sender, EventArgs e)    
{    
&#160;&#160;&#160; var selectedKeys = lvPeople.GetSelectedDataKeys(&quot;chkAwesomePoints&quot;);    
&#160;&#160;&#160; var selectedIds = selectedKeys.Select(x =&gt; new Guid(x.Value.ToString())); 
  &#160;&#160;&#160; if (selectedIds.Count() &gt; 0)   
&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var targetPoints = int.Parse(txtAwesomePoints.Text);    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var db = new PeopleDBEntities(); 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; db.People.Where(x =&gt; selectedIds.Contains(x.Id))   
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; .ToList()    
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; .ForEach(x =&gt; x.AwesomePoints = targetPoints); 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; db.SaveChanges();   
&#160;&#160;&#160;&#160;&#160;&#160;&#160; lvPeople.DataBind();    
&#160;&#160;&#160; } 
  }    
    
protected void UpdateAttendeeStatusButton_Click(object sender, EventArgs e)    
{    
&#160;&#160;&#160; var selectedKeys = lvPeople.GetSelectedDataKeys(&quot;chkAttending&quot;);    
&#160;&#160;&#160; var selectedIds = selectedKeys.Select(x =&gt; new Guid(x.Value.ToString())); 
  &#160;&#160;&#160; if (selectedIds.Count() &gt; 0)   
&#160;&#160;&#160; {    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var targetValue = chkAttending.Checked;    
&#160;&#160;&#160;&#160;&#160;&#160;&#160; var db = new PeopleDBEntities(); 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; db.People.Where(x =&gt; selectedIds.Contains(x.Id))   
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; .ToList()    
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; .ForEach(x =&gt; x.IsAttending = targetValue); 
  &#160;&#160;&#160;&#160;&#160;&#160;&#160; db.SaveChanges();   
&#160;&#160;&#160;&#160;&#160;&#160;&#160; lvPeople.DataBind();    
&#160;&#160;&#160; }    
}    
    
</pre></p>  <p>All we've done here is used our extension method to get the datakeys of the selected items in the ListView and have updated those items accordingly. Granted there's no validation or anything fancy going on – hey, this is a demo – it shows how we can know which items of a ListView have been selected.</p>  <h3>Source</h3>  <p>You can download the source by <a href="http://cid-9d8a7728356393b1.office.live.com/self.aspx/.Public/code/ListViewSelectionDemo.zip" target="_blank">clicking here.</a></p>
        