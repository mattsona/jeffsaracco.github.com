---
layout: post
title: "ASP.NET C# - Catch the SelectedIndexChanged of a DropDownList inside a Repeater"
date: 2011-11-28 18:56
comments: true
categories: [ASP.NET,C#,DropDownList,ItemDataBound,Repeater,SelectedIndexChanged]
---

When there is a Dropdownlist inside a repeater the dropdownlist will not throw the SelectedIndexChanged event even when the AutoPostback property is set to true. To get this event to throw I had to bind the dropdownlist on the ItemDataBound event of the repeater and set the event handler for the SelectedIndexChanged event, as seen below:

``` c#
protected void rptrCandidates_ItemDataBound(object sender, RepeaterItemEventArgs e)
{
     if ((e.Item.ItemType == ListItemType.Item) || (e.Item.ItemType == ListItemType.AlternatingItem))
     {
         DropDownList actions = e.Item.FindControl("actionsDDL") as DropDownList;
         if ((actions != null))
         {
               ListItemCollection _items = new ListItemCollection();
               foreach (string s in options)
               {
                    _items.Add(new ListItem(s, row.ProjectCandidateId.ToString()));
               }
               actions.SelectedIndexChanged += new EventHandler(actionsDDL_SelectedIndexChanged);
               actions.DataSource = _items;
               actions.DataBind();
          }
     }
}
```

However, when the dropdownlist is bound the DataTextField and DataValueField are both filled with the Text value, and when I try to explicity set these, the SelectedIndexChanged event is not thrown. To get around this bizarre feature what I had to do was, next to the dropdownlist, put a hiddenfield that got bound to it the correct value that was supposed to be in the Value field of the list items (all were to have the same value):

``` html
<ItemTemplate>
    <asp:HiddenField ID="projectCandidateHF" runat="server" Value='<%# Eval("ProjectCandidateId") %>' />
    <asp:DropDownList ID="actionsDDL" runat="server" AutoPostBack="true" SelectedIndexChanged="actionsDDL_SelectedIndexChanged"> 
    </asp:DropDownList>
</ItemTemplate>
```

Then on the SelectedIndexChanged event that the dropdownlist throws use the sender to get the DropDownList that threw the event and then use the Parent property to get the RepeaterItem in which it lives, then find the HiddenField using FindControl method:

``` c#
protected void actionsDDL_SelectedIndexChanged(object sender, EventArgs e)
{
    DropDownList actionsDDL = sender as DropDownList;
    HiddenField projectCandidateIdHF = actionsDDL.Parent.FindControl("projectCandidateHF") as HiddenField;
    //do fun stuff here
}
```
