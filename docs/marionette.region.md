# Regions

Regions provide consistent methods to manage, show and destroy
views in your applications and layouts. You can use a jQuery selector to
identify where your region must be displayed.

See the documentation for [laying out views](./marionette.view.md#laying-out-views---regions) for an introduction in
managing regions throughout your application.

A Region is a [`Marionette.Object`](./marionette.object.md) and has all of its functionality.

Regions maintain the [View's lifecycle](./viewlifecycle.md#regions-and-the-view-lifecycle) while showing or emptying a view.

## Documentation Index

* [Defining the Application Region](#defining-the-application-region)
* [Defining Regions](#defining-regions)
  * [String Selector](#string-selector)
  * [Additional Options](#additional-options)
  * [Specifying regions as a Function](#specifying-regions-as-a-function)
  * [Using a RegionClass](#using-a-regionclass)
  * [Referencing UI in `regions`](#referencing-ui-in-regions)
* [Adding Regions](#adding-regions)
* [Removing Regions](#removing-regions)
* [Using Regions on a view](#using-regions-on-a-view)
* [Showing a View](#showing-a-view)
  * [Checking whether a region is showing a view](#checking-whether-a-region-is-showing-a-view)
* [Emptying a Region](#emptying-a-region)
  * [Preserving Existing Views](#preserving-existing-views)
  * [Detaching Existing Views](#detaching-existing-views)
* [`reset` A Region](#reset-a-region)
* [Set How View's `el` Is Attached](#set-how-views-el-is-attached)

## Defining the Application Region

The Application defines a single region `el` using the `region` attribute. This
can be accessed through `getRegion()` or have a view displayed directly with
`showView()`. Below is a short example:

```javascript
var Mn = require('backbone.marionette');
var SomeView = require('./view');

var App = Mn.Application.extend({
  region: '#main-content',

  onStart: function() {
    var main = this.getRegion();  // Has all the properties of a `Region`
    main.show(new SomeView());
  }
});
```

For more information, see the
[Application docs](./marionette.application.md#root-layout).

## Defining Regions

Marionette supports multiple ways to define regions on your `Application` or
`View`. This section will document the different types as applied to `View`,
although they will work for `Application` as well - just replace `regions` with
`region` in your definition.

### String Selector

You can use a jQuery string selector to define regions.

```javascript
var Mn = require('backbone.marionette');

var MyView = Mn.View.extend({
  regions: {
    mainRegion: '#main'
  }
});
```

### Additional Options

You can define regions with an object literal. Object literal definitions expect
an `el` property - the jQuery selector string to hook the region into. The
object literal is the most common way to define whether showing the region
overwrites the `el` or just overwrites the content (the default behavior).

To overwrite the parent `el` of the region with the rendered contents of the
inner View, use `replaceElement` as so:

```javascript
var Mn = require('backbone.marionette');

var OverWriteView = Mn.View.extend({
  className: '.new-class'
});

var MyView = Mn.View.extend({
  regions: {
    main: {
      el: '.overwrite-me',
      replaceElement: true
    }
  }
});
var view = new MyView();
view.render();

console.log(view.$('.overwrite-me').length); // 1
console.log(view.$('.new-class').length); // 0

view.showChildView('main', new OverWriteView());

console.log(view.$('.overwrite-me').length); // 0
console.log(view.$('.new-class').length); // 1
```

When the instance of `MyView` is rendered, the `.overwrite-me` element will be
removed from the DOM and replaced with an element of `.new-class` - this lets
us do things like rendering views inside `table` or `select` more easily -
these elements are usually very strict on what content they will allow.

**_DEPRECATED: The `selector` option of a region is deprecated in favor of using `el`_**

```js
var MyView = Mn.View.extend({
  regions: {
    deprecatedRegionDefinition: {
      selector: '.foo',
      replaceElement: true
    },
    regionDefinition: {
      el: '.bar',
      replaceElement: true
    }
  }
});
```

### Specifying regions as a Function

The `regions` attribute on a view can be a
[function returning an object](./basics.md#functions-returning-values):

```javascript
var Mn = require('backbone.marionette');

var MyView = Mn.View.extend({
  regions: function(){
    return {
      firstRegion: '#first-region'
    };
  }
});
```

### Using a RegionClass

If you've created a custom region class, you can use it to define your region.

```javascript
var Mn = require('backbone.marionette');

var MyRegion = Mn.Region.extend({
  onShow: function(){
    // Scroll to the middle
    this.$el.scrollTop(this.currentView.$el.height() / 2 - this.$el.height() / 2);
  }
});

var MyView = Mn.View.extend({
  regions: {
    firstRegion: {
      el: '#first-region',
      regionClass: MyRegion
    }
  }
});

```

[Live example](https://jsfiddle.net/marionettejs/oLLrzx8g/)

### Referencing UI in `regions`

The UI attribute can be useful when setting region selectors - simply use
the `@ui.` prefix:

```javascript
var Mn = require('backbone.marionette');

var MyView = Mn.View.extend({
  ui: {
    region: '#first-region'
  },
  regions: {
    firstRegion: '@ui.region'
  }
});

```

[Live example](https://jsfiddle.net/marionettejs/ey1od1g8/)

## Adding Regions

To add regions to a view after it has been instantiated, simply use the
`addRegion` method:

```javascript
var MyView = require('./myview');

myView = new MyView();
myView.addRegion('thirdRegion', '#third-region');
```

Now we can access `thirdRegion` as we would the others.

You can also add multiple regions using `addRegions`.

```javascript
var MyView = require('./myview');

myView = new MyView();
myView.addRegions({
  main: {
    el: '.overwrite-me',
    replaceElement: true
  },
  sidebar: '.sidebar'
});
```

## Removing Regions

You can remove all of the regions from a view by calling `removeRegions` or you can remove a region by name using `removeRegion`. When a region is removed the region will be destroyed.

```javascript
var Mn = require('backbone.marionette');

var MyView = Mn.View.extend({
  regions: {
    main: '.main',
    sidebar: '.sidebar',
    header: '.header'
  }
});

var myView = new MyView();

// remove only the main region
var mainRegion = myView.removeRegion('main');

mainRegion.isDestroyed(); // -> true

// remove all regions
myView.removeRegions();
```

## Using Regions on a view

In addition to adding and removing regions there are a few
methods to help utilize regions.

- `getRegion(name)` - Request a region from a view by name.
- `getRegions()` - Returns an object literal of all regions on the view organized by name.
- `hasRegion(name)` - Check if a view has a region.
- `emptyRegions()` - Empty all of the regions on a view.

## Showing a View

Once a region is defined, you can call its `show` method to display the view:

```javascript
var myView = new MyView();
var childView = new MyChildView();
var mainRegion = myView.getRegion('main');

// render and display the view
mainRegion.show(childView);
```

This is equivalent to a view's `showChildView` which can be used as:

```javascript
var myView = new MyView();
var childView = new MyChildView();

// render and display the view
myView.showChildView('main', childView);
```

Both forms take an `options` object that will be passed to the
[events fired during `show`](./viewlifecycle.md#show-view-events).

For more information on `showChildView` and `getChildView`, see the
[Documentation for Views](./marionette.view.md#managing-sub-views)

### Checking whether a region is showing a view

If you wish to check whether a region has a view, you can use the `hasView`
function. This will return a boolean value depending whether or not the region
is showing a view.

```javascript
var myView = new MyView();
var mainRegion = myView.getRegion('main');

mainRegion.hasView() // false
mainRegion.show(new OtherView());
mainRegion.hasView() // true
```

If you show a view in a region with an existing view, Marionette will
[remove the existing View](#emptying-a-region) before showing the new one.

## Emptying a Region

You can remove a view from a region (effectively "unshowing" it) with
`region.empty()` on a region:

```javascript
var myView = new MyView();

myView.showChildView('main', new OtherView());
var mainRegion = myView.getRegion('main');
mainRegion.empty();
```

This will destroy the view, clean up any event handlers and remove it from
the DOM. When a region is emptied [empty events are triggered](./viewlifecycle.md#empty-region-events).

**Note**: If the region does _not_ currently contain a View it will detach
any HTML inside the region when emptying. If the region _does_ contain a
View [any HTML that doesn't belong to the View will remain](./upgrade.md#changes-to-regionshow).

### Preserving Existing Views

**_DEPRECATED: `preventDestroy` is deprecated. See
[Detaching Existing Views](#detaching-existing-views)
for how to preserve a shown view_**

If you replace the current view with a new view by calling `show`,
by default it will automatically destroy the previous view.
You can prevent this behavior by passing `{preventDestroy: true}` in the options
parameter. Several events will also be triggered on the views.

```javascript
// Show the first view.
var myView = new MyView();
var childView = new MyChildView();

var mainRegion = myView.getRegion('main');

mainRegion.show(childView);

// Replace the view with another. The
// `destroy` method is called for you
var anotherView = new AnotherView();
mainRegion.show(anotherView);

// Replace the view with another.
// Prevent `destroy` from being called
var anotherView2 = new AnotherView();
mainRegion.show(anotherView2, {preventDestroy: true});
mainRegion.empty({preventDestroy: true});
```

**NOTE** When using `preventDestroy: true` you must be careful to cleanup your
old views manually to prevent memory leaks.

### Detaching Existing Views

If you want to detach an existing view from a region, use `detachView`.

```javascript
var myView = new MyView();

var myOtherView = new MyView();

var childView = new MyChildView();

// render and display the view
myView.showChildView('main', childView);

// ... somewhere down the line
myOtherView.showChildView('main', myView.getRegion('main').detachView());
```

## `reset` A Region

A region can be `reset` at any time. This destroys any existing view
being displayed, and deletes the cached `el`. The next time the
region shows a view, the region's `el` is queried from
the DOM.

```javascript
var myView = new MyView();
myView.showChildView('main', new OtherView());
var mainRegion = myView.getRegion('main');
myRegion.reset();
```

This can be useful in unit testing your views.

## Set How View's `el` Is Attached

Override the region's `attachHtml` method to change how the view is attached
to the DOM. This method receives one parameter - the view to show.

The default implementation of `attachHtml` is essentially:

```javascript
var Mn = require('backbone.marionette');

Mn.Region.prototype.attachHtml = function(view){
  this.el.appendChild(view.el);
}
```

It is also possible to define a custom attach method for a single region by
extending from the Region class and including a custom `attachHtml` method.

This example will make a view slide down from the top of the screen instead of just
appearing in place:

```javascript
var Mn = require('backbone.marionette');

var ModalRegion = Mn.Region.extend({
  attachHtml: function(view){
    // Some effect to show the view:
    this.$el.empty().append(view.el);
    this.$el.hide().slideDown('fast');
  }
});

var MyView = Mn.View.extend({
  regions: {
    mainRegion: '#main-region',
    modalRegion: {
      regionClass: ModalRegion,
      el: '#modal-region'
    }
  }
});
```
