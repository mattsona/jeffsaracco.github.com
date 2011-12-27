---
layout: post
title: "Dropping Partial MVC Views on Old School WebForms Pages"
date: 2011-11-28 18:47
comments: true
categories: [ASP.NET-MVC, C#, WebForms]
---
Original Source [here](http://stackoverflow.com/questions/702746/how-to-include-a-partial-view-inside-a-webform) with some changes due to difference in versions referenced.

I was refactoring some pages and needed to drop an MVC partial view on one of our WebForms (aspx) pages. However, this functionality does not exist natively so I had to get creative.
This solution, in my opinion, was best for now because it was easier than creating two of the same control, one for MVC and one for WebForms. It was also easier than refactoring all of the WebForms pages to be MVC just so that they could use this one control (however, they will probably be upgraded at some point in the future).

Without further ado, here is the solution (if you didn.t already click the link at the top):

Create a blank WebFormController class and a WebFormMVCUtil class which will grab the view, bind the model to it and write it to the screen. The code (located in public/Controllers/WebFormController.cs) is below:

``` c#
public class WebFormController : Controller { }
 
    public static class WebFormMVCUtil
    {
 
        public static void RenderPartial(string partialName, string controller, object model)
        {
            //get a wrapper for the legacy WebForm context
            var httpCtx = new HttpContextWrapper(HttpContext.Current);
 
            //create a mock route that points to the empty controller
            var rt = new RouteData();
            rt.Values.Add("controller", controller);
 
             //create a controller context for the route and http context
            var ctx = new ControllerContext(new RequestContext(httpCtx, rt), new WebFormController());
 
             //find the partial view using the viewengine
            var view = ViewEngines.Engines.FindPartialView(ctx, partialName).View;
 
             //create a view context and assign the model
            var vctx = new ViewContext(ctx, view,
                new ViewDataDictionary { Model = model },
                new TempDataDictionary(), HttpContext.Current.Response.Output);
 
            //render the partial view
            view.Render(vctx, HttpContext.Current.Response.Output);
        }
    }
}
```
If you don.t care how it is implemented and just want to know how to stick an MVC partial view on your WebForms page just add the line below (substituting your own controller and view names in)

```
<% WebFormMVCUtil.RenderPartial("ViewName", "ControllerName", model); %>
```

Also, depending on your view, you need to pass it the model; but, as I did here, you can place a method in there that will get the model for you.
