---
layout: post
title: Different ways to execute code on startup of WCF application
---

1. Add a Global.asax file in VS, then use Application_Start method in the code-behind class (inhereted from HttpApplication)
2. Create an App_Code folder, then create a class (with any name) with a `public static void AppInitialize()`.
This approach can be a bit tricky, because the class will be compiled during startup and you cannot use any internal classes,
e.g. Settings.
3. Starting from the .NET Framework 4.5 you can define a `public static void Configure(ServiceConfiguration config)` for a service class.
It will override any existing  app.config/web.config configuration. Details: [MSDN Article](https://msdn.microsoft.com/en-us/library/hh205277(v=vs.110).aspx)
4. [Implement `ServiceHostFactory`](http://blogs.msdn.com/b/carlosfigueira/archive/2011/06/14/wcf-extensibility-servicehostfactory.aspx)
