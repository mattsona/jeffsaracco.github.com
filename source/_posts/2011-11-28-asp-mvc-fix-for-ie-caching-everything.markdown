---
layout: post
title: "ASP MVC - Fix for IE caching everything"
date: 2011-11-28 18:34
comments: true
categories: [ASP.NET-MVC,C#]
---
So it turns out that IE aggressively caches every AJAX GET response, which makes for some interesting and hard to fix bugs since we moved a lot of functionality to AJAX calls with MVC.

Fear not, I have found a solution for this. The solution is to put a custom attribute on your MVC GET action if it is being called through AJAX.
For Instance, the GET method for getting a list may look like this:
``` c#
[CacheControl(HttpCacheability.NoCache), HttpGet]
public ActionResult MyAction(int param1, int param2)
```

`CacheControl(HttpCacheability.NoCache)` is the attribute you should put on your MVC action in order for IE to not cache anything.

If all you want to know is how to fix this bug, you can stop reading here, if you want to know what it is doing, keep going.

The way this works is creating a class that inherits from the ActionFilterAttribute class and overriding the OnActionExecuted method and adding some no-cache information to the headers being sent out.
Full class below

``` c#
public class CacheControlAttribute : ActionFilterAttribute
{
        public CacheControlAttribute(HttpCacheability cacheability)
        {
            _cacheability = cacheability;
        }
 
        private readonly HttpCacheability _cacheability;
 
        public override void OnActionExecuted(ActionExecutedContext filterContext)
        {
            HttpCachePolicyBase cache = filterContext.HttpContext.Response.Cache;
            cache.SetCacheability(_cacheability);
            cache.SetExpires(DateTime.Now);
            cache.SetAllowResponseInBrowserHistory(false);
            cache.SetNoServerCaching();
            cache.SetNoStore();
           
	}
}
```
