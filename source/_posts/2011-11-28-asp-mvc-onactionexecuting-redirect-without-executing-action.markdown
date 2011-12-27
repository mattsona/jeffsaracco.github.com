---
layout: post
title: "ASP MVC OnActionExecuting redirect without executing action"
date: 2011-11-28 18:54
comments: true
categories: [ASP.NET-MVC, C#, OnActionExecuting]
---
I had a bug where an exception was being thrown for a null object in an action, so I decided to make an ActionFilter to check that the user had the right role and if not, redirect to default.aspx (which, it turns out, redirects to the correct dashboard, super bonus!). It worked, however the action was still being executed and throwing the error before the redirect happened.

The fix for this, to stop execution of the action is set the result of the filterContext object like so:
``` c#
public class RequiresFirmRoleAttribute : ActionFilterAttribute
{
     public override void OnActionExecuting(ActionExecutingContext filterContext)
     {
        if (!((BaseFirmController)filterContext.Controller).IsFirmAdmin)
        {
            filterContext.Result = new RedirectResult("/Default.aspx");
        }
     }
}
```
