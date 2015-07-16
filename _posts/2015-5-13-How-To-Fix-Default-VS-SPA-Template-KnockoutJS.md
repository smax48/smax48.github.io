---
layout: post
title: How to fix the default VS SPA template to work with Knockout.JS
---

If you create a new ASP.NET project with VS 2013.4 and select the default SPA template, you will get a bit uncompleted and totally undocumented set of client-side JS files.

This article will explain how to tweak this template to be able to create SPA with Knockout JS.
I must admit that I am not an expert in JavaScript development, so there might be better solutions.

Anyway, let's begin.

If you are after a SPA and want to build it with ASP.NET, probably you will want to use WebAPI to implement services that you will later call using AJAX.
The suggested (by Microsoft) way of retrieving the OAuth token (that you need to authenticate REST API calls) is a bit strange, but I will leave it without changes.

The problem here is that you must have both cookie-based login in the browser AND an OAuth token for WebAPI calls.
In this SPA template, the latter is retrieved using a redirection trick.
It is not very useful if, for instance, you want to call your WebAPI-based REST services from another client.
In this case I would suggest using IdentityServer.  

The next problem is related with how you structure and activate your views.
I couldn't find any real examples about what to do if you have more than ONE view (as you have in the default example).

After googling and thinking I came up with two solutions.

### 1. Use Knockout's 'with' binding and multiple partial cshtml views, rendered at once from Index.cshtml

```
@section SPAViews {
    @Html.Partial("_Home")
    @Html.Partial("_AnotherView")
}
```

Each view begins with <!-- ko with: bindingPropertyName -->, where bindingPropertyName is the name of JS property in the app object for your view model (app.home, app.anotherView etc). It is created automatically as a computed property when you call app.addViewModel().

To select the current view, you must call app.view(viewModelObject) inside the corresponding route function.
The problem is that the default implementation ALWAYS returns a not-null object when you call app.home() or app.anotherView(), regardless of which view is the current. Hence all of your views will be visible on the screen unless you add some if-conditions.

Fortunately, we can easily fix this by adding one line to the ko.computed function that is created for view model binding (app.viewmodel.js, inside addViewModel):

```javascript
//hack - begin
if (self.view() !== viewItem) {
	return null;
}
//hack - end
return self.Views[options.name]; });
```
Now if a view model is not current, the corresponding binding property will return null and knockout will hide this view (remember our 'with' bindings).

This works somehow, but you have unpleasant blinking when you first load the app in browser (because html is loaded before knockout is activated and you see all your views for a moment).

So I decided to test another idea - make use of knockout's components.

### 2. Use Knockout's components

The idea is to implement all our views as components and render them inside the single div element.
In this case we don't need the hack I mentioned before.
Nevertheless, we still have to modify the addViewModel function.

First of all, a component must have a name :). I decided to inject a property that holds a viewModel name inside the view model object.
The name can be anything, I add the 'View' postfix to options.name.

```javascript
var componentName = options.name + 'View';
viewItem['__name__'] = componentName;
```

Of course, we need to add a property with the current view name to our AppViewModel:

```javascript
self.currentViewName = ko.computed(function() {
    return self.view().__name__;
});
```

To facilitate the components registration process, I added another call to addViewModel:

```javascript
ko.components.register(componentName, {
    viewModel: { instance: viewItem },
    template: { element: options.name + 'ViewTemplate' }
});
```

Here we register a single instance of our view model as a knockout component.
In the ideal case, template should be loaded dynamically, eg using RequireJS.
Unfortunately, ASP.NET 4 / MVC 5 has its own technology for bundling, and it doesn't play nice with the RequireJS
(if you plan to use RequireJS in your project, you can easily load your templates using the text plugin).
I hope that in ASP.NET 5 / MVC 6 Microsoft will use something more compatible with dynamic module loading.

Now you don't need the 'with' binding to switch between views.
You can still place your html inside partial views, but you need to wrap them into script tags like this: `<script type="text/html" id="HomeViewTemplate">`
(note the template's element name that I used in component registration).

The Index.cshtml will have code like this:

```
@section SPAViews {
    @Html.Partial("_Home")
    @Html.Partial("_AnotherView")
    <h3 data-bind="visible: isLoading">Loading application, please wait...</h3>
    <div id="view-holder" data-bind="component: {name: currentViewName }">
    </div>
}
```

And yes, I forgot to mention that you also need to create the "isLoading" property for AppViewModel:

```javascript
self.isLoading = ko.computed(function () {
    return self.view() === self.Views.Loading;
});
```

This property will have the value 'false' immediately after navigating to the first view. Before that (and even before knockout is activated) you can display something like "Loading, please wait...".

That's basically all.
