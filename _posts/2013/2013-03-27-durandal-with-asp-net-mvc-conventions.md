---
layout: post
title:  "Durandal with ASP.NET MVC Conventions"
date:   2013-03-27 16:07:17 +0000
categories: Development
tags: asp.net mvc durandal
---

# Background

Durandal is a SPA framework that is built on top of already popular Javascript libraries including jQuery, Knockout and RequireJS.

It provides Javascript/HTML modularity, SPA lifecycle management, navigation and screen state management plus various other features that simplify SPA development.

However out of the box Durandal seems to throw away common ASP.NET MVC conventions in favour of its own.

If you install Durandal into your project via the NuGet package you will see that it creates an `App` folder that is intended to host not only all of the Durandal modules, but also all of your views and viewmodels.

This convention is fine for small scale applications and makes optimisation using Durandal’s `optimizer.exe` straightforward. However there are some issues with this folder structure that I’m not initially fond of:

- Views are no longer in the default locations searched for by the RazorViewEngine. I still want to leverage the power of Razor views to control markup generation via the use of various HtmlHelpers.

- Views are now potentially in two separate locations (`/Views` and `/App/views`).

- Scripts are also now in two separate locations (`/Scripts` and `/App/viewmodels`).

I’m currently working on rewriting a large scale enterprise application using ASP.NET MVC. This application is broken down into a number of modules, with each module being its own SPA within its own assembly. With this in mind, it’s easy to see why the “out of the box” conventions do not work for me.

I would personally prefer to have Durandal work **around** MVC, as opposed to it being the defining aspect of the project. The rest of this post will discuss how to configure Durandal to work within existing MVC conventions, and to also use Razor to provide views for your SPA.

# Initial Structure

I'm going to create a new ASP.NET MVC 4 Web Application using the "Basic" template to keep things simple. 

First create a basic `HomeController` with two actions...

```csharp
public class HomeController : Controller
{
    //
    // GET: /Home/
    // (this is for the SPA landing page)
    public ActionResult Index()
    {
        return View();
    }
 
    //
    // GET: /Home/Shell
    // (this will serve the initial shell for the SPA)
    public ActionResult Shell()
    {
        return View();
    }
}
```

Also go ahead and create their associated views...

_Index.cshtml_

```html
@{
    ViewBag.Title = "Index";
    Layout = "~/Views/Shared/_Layout.cshtml";
}
<div id="applicationHost">
    <h2>Landing Page...</h2>
</div>
```

_Shell.cshtml_

```html
@{
    Layout = null;
}
<div>
    This is the shell for the app!
</div>
```

Next we're going to pull Durandal in via NuGet. 

As previously described Durandal creates an `App` folder that is meant to host all of the views and viewmodels for the SPA. I'm going to move the Durandal modules into the main `Scripts` folder in order to keep all of my vendor scripts where I would prefer them to be.

Here is what my structure looks like afterwards:

![Project Structure](/assets/images/posts/2013/durandal-1.png)

I have chosen to separate out the 3rd party vendor libraries (including Durandal) into a `lib` folder, leaving my application specific modules in an `app` folder.

_Note: I've also updated these paths in BundleConfig_

I've also created a `viewmodels` folder within `app` to hold all of the viewmodel modules.

As part of the changes to BundleConfig I have created a `vendor` bundle that contains jQuery and Knockout and also updated `_Layout.cshtml` to use this bundle.

_BundleConfig.cs_
```csharp
bundles.Add(new ScriptBundle("~/bundles/vendor").Include(
        "~/Scripts/lib/jquery-{version}.js",
        "~/Scripts/lib/knockout-{version}.js"));
```

__Layout.cshtml_
```csharp
@Scripts.Render("~/bundles/vendor")
```

Also notice that I have created a `main.js` file in the `app` folder and a `shell.js`.

The `main.js` file will be the entry point to the SPA and will be responsible for configuring and bootstrapping the app.

`shell.js` will be the initial module for the SPA. It is important to organise the app specific modules under the same folder structure as shared by the views. We will be configuring Durandal to use this convention next...

# Configuring Durandal

The first thing we need to do is add a reference to RequireJS in our landing page.

```html
@section Scripts {
    <script type="text/javascript" 
            src="~/Scripts/lib/durandal/amd/require.js" 
            data-main="@Url.Content("~/Scripts/app/main")"></script>    
}
```

This is the full contents of the entry point file, I will explain each part individually below...

_main.js_

```javascript
require.config({
    paths: {
        'lib': '/Scripts/lib',
        'app': '/Scripts/app',
        'durandal': '/Scripts/lib/durandal'
    }
});

define(['durandal/app', 'durandal/system', 'durandal/viewLocator', 'durandal/viewEngine'],
    function (app, system, viewLocator, viewEngine) {

        system.debug(true);

        viewLocator.useConvention('viewmodels', '../../..');
        viewEngine.viewExtension = '/';
        viewEngine.viewPlugin = 'durandal/amd/text';
        
        app.start().then(function () {
            app.setRoot('viewmodels/home/shell');
        });
    }
);
```

First we need to configure Durandal/RequireJS in `main.js` in order to make them aware of our new conventions.

RequireJS loads modules according to a baseUrl. As we used the data-main attribute on the require.js include, RequireJS uses the location of `main.js` as this baseUrl. 

I've configured RequireJS with some default paths to my `app/lib` directories, as well as to the `Durandal` directory.

```javascript
require.config({
    paths: {
        'lib': '/Scripts/lib',
        'app': '/Scripts/app',
        'durandal': '/Scripts/lib/durandal'
    }
});
```

The following line instructs Durandals viewLocator of the convention we want to use when mapping module folders to view folders. The first parameter is a string in the path that will be replaced by the second parameter.

```javascript
viewLocator.useConvention('viewmodels', '../../..');
```

So for example if retrieving the shell module, this will reference `/home/shell` as the view directory is mapped relative to the viewmodels directory.

The next line instructs the view engine to use `/` as the view extension instead of the default `.html`. We want this as we will be accessing our views through our controllers and the ASP.NET routing mechanism. 

```javascript
viewEngine.viewExtension = '/';
```

Finally, because we moved Durandal from its default location we need to update where it looks for the _text_ module.

```javascript
viewEngine.viewPlugin = 'durandal/amd/text';
```

With the config/bootstrap code in place, all that is left to do is provide the code for the shell module...

_shell.js_

```javascript
define(function () {
    var shell = {
        activate: activate
    };

    function activate() {
        alert('Shell started!');
    }

    return shell;
});
```

And there you have it, Durandal working nicely within ASP.NET MVC conventions. Although you can easily take the concepts outlined here and apply them to whatever conventions you prefer for your projects.

A sample project can be found [here](https://github.com/brettpostin/Durandal-MVC4).