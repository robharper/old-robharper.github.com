---
layout: post
title: "Ember's {{outlet}}"
date: 2013-03-30 13:43
comments: true
categories: ember
---

{% raw %}
While Ember's convention over configuration often provides the desired result out of the box, I often find myself needing to dig into the Ember source to find out what's really happening behind the wizard's curtain.

The handlebars `{{outlet}}` helper is typically used to specify the location at which the views of child routes will render into a parent route's template:

``` html
<header>
  Some content
</header>
{{outlet}}
<footer>
  Some other content
</footer>
```

Behind the scenes, the `{{outlet}}` helper does the following:

 1. Finds the top-level view associated with the route - the `outletSource`.
 
    For example, if the outlet is somewhere within the content rendered in the `application` template, the helper must traverse the view hierarchy to find the instance of the `ApplicationView`.
 
 2. Inserts a new [`ContainerView`](http://emberjs.com/api/classes/Ember.ContainerView.html) at the location of the `{{outlet}}`.

     In reality, Ember uses a [`Ember._Metamorph`](http://emberjs.com/api/classes/Ember._Metamorph.html) extension of a ContainerView. The metamorph view is *virtual*, preventing the view from appearing in the `childViews` / `parentView` hierarchy, and it doesn't create a DOM element (it uses a [metamorph](https://github.com/tomhuda/metamorph.js/)), so any template rendered into the outlet appears precisely in place of `{{outlet}}` helper without a wrapping element.  Here's a fiddle that shows [how mixing in _Metamorph affects how a ContainerView works](http://jsfiddle.net/rharper/bzga8/4/)
 
 3. Binds the `currentView` property of this ContainerView to the view under a named key in the `outletSource`'s `_outlets` hash.

     Unless the outlet is explicitly given a name it gets the name `main`. In this case, the currentView is bound to `outletSource._outlets.main`. When the view the _outlets hash is changed, Ember automatically updates the view in the outlet through the magic of bindings.

{% endraw %}