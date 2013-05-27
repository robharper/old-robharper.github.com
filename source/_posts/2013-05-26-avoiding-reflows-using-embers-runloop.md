title: Avoiding Reflows using Ember's Run Loop
date: 2013-05-26 14:54:08
tags:
---

Most of the work I do day-to-day is targeted at mobile devices. While it's [no secret](http://www.stubbornella.org/content/2009/03/27/reflows-repaints-css-performance-making-your-javascript-slow/) that [avoiding excessive reflows](http://mir.aculo.us/2010/08/17/when-does-javascript-trigger-reflows-and-rendering/) is important for performance, it is absolutely essential on mobile, particularly on older devices. 

Simply put, document changes that affect element geometry invalidate some or all of the [page layout](http://www.phpied.com/rendering-repaint-reflowrelayout-restyle/). If an invalidating change is followed by a read from the computed style of an affected node, the browser is forced to synchronously recalculate the layout. This takes time. For example:

```javascript
for (var i = 0; i < elements.length; i++) {
  var height = elements[i].clientHeight;
  elements[i].style.height = (height % 20) + 'px';
}
```

The change to element `i`s height invalidates the layout. When the `clientHeight` of element `i+1` is requested the browser must recalculate the layout before returning a value. Depending on the number of elements and complexity of the DOM and styling, this loop can chew up hundreds of milliseconds and ruin the user experience.

While it's fairly easy to de-interleave the reads and writes to avoid forced layouts in the above example, it is slightly more complex when working with a collection of disparate Ember views. 

Take for example a `CollectionView` with many child views, each which reads and then modifies a few element style properties on `didInsertElement`. A naive approach would be to orchestrate batching reads and writes using the parent view, but this would break encapsulation. Another approach would be to allow each view to read computed styles on insertion but defer writes using `setTimeout(..., 0)`. However, this would return control to the browser before the changes are applied and could cause a flash of partially/incorrectly styled content.

A simple and effective approach uses the Ember [run loop](http://stackoverflow.com/a/14296339/1204216). First, do all necessary calculated style reads during `didInsertElement`. It executes in the *"render"* queue of the run loop. Second, defer all writes into a later queue, say the *"afterRender"* queue.

```javascript
didInsertElement: function() {
  var element = this.get('element');
  var originalHeight = element.clientHeight;
  Ember.run.scheduleOnce('afterRender', function() {
    element.style.height = (originalHeight % 20) + 'px';
  });
}
```

This is an easy way to perform all reads followed by all invalidating writes while remaining in a single browser event loop execution. It can optimize what normally would be one reflow per view down to a single reflow for all views.

## TL;DR

To avoid excessive reflows across your Ember views, perform computed style reads as needed but defer all writes to a later stage of the run loop.