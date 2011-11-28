## Getting Started

After reading this guide, you will be able to:

 * Use SproutCore's templating support to describe the appearance of your application.
 * Delegate handling of user events with views.
 * Use bindings to update views when your models change.

You will learn all of this by building a real Todo list application from the ground up using SproutCore.

endprologue.

### Following Along

You can see the finished source code for this application on [Github](https://github.com/sproutcore/todos/). To see it in action, click [here](http://todos20.strobeapp.com).

### Installing SproutCore

Download the [SproutCore Starter Kit](https://github.com/sproutcore/starter-kit/downloads) if you haven't already. Unzip and it open the directory in your favorite text editor.

### Core Concepts

Here are the two key features that make SproutCore so powerful:

*Bindings*: Bindings create a bidirectional link between two properties. If the property of one object changes, the property on the other object updates automatically.

*Computed Properties*: Computed properties allow you to treat a function as a property. This allows you to synthesize a new property from multiple input properties, and have it update automatically any time one of those input properties changes.

This tutorial will show you how to use these two concepts to build a todo app with astonishingly little code.

### Create the Namespace

Open the JavaScript file located at `js/app.js`. Replace the default contents with the following:

<javascript filename="js/app.js">
Todos = SC.Application.create();
</javascript>

This code creates a namespace for your application (called `Todos`), which is also an instance of SC.Application.

NOTE: It is important that every SproutCore app creates an instance of SC.Application, because it is responsible for routing browser events to your views.

### Defining Your Model

In this tutorial, we want to create a list for managing todos. Users should be able to create a new todo with a specific task, then check it off once it's done.

Let's define our model as a new subclass of `SC.Object`:

<javascript filename="js/app.js">
Todos.Todo = SC.Object.extend({
  title: null,
  isDone: false
});
</javascript>

NOTE: Make sure you insert the new code after the `Todos` object is created.

We've now defined a class with two properties: `title`, a String, and `isDone`, a Boolean.

### Managing the Model Using a Controller

Now that we know what our data looks like, let's create a controller to manage it. Since we want to maintain an ordered list of todos, we'll use an instance of `SC.ArrayProxy`.

<javascript filename="js/app.js">
Todos.todosController = SC.ArrayProxy.create({
  // Initialize the array controller with an empty array.
  content: []
});
</javascript>

INFO: In MVC frameworks, like SproutCore, the controller layer bridges the model layer, which is only concerned with a pure-data representation of objects, and the view layer, which is only concerned with representing those objects.

Now we have an array controller with no content. Let's add a method to create a new todo:

<javascript filename="js/app.js">
// updating previous code

Todos.todosController = SC.ArrayProxy.create({
  // Initialize the array controller with an empty array.
  content: [],

  // Creates a new todo with the passed title, then adds it
  // to the array.
  createTodo: function(title) {
    var todo = Todos.Todo.create({ title: title });
    this.pushObject(todo);
  }
});
</javascript>

NOTE: `SC.ArrayProxy` acts as a proxy onto its `content` Array. SproutCore will propagate any modifications made to the `ArrayProxy` to the `content` Array.

### Doing It with Style

We've provided a simple stylesheet to give the application some style.  Download the [CSS file](https://raw.github.com/sproutcore/todos/master/css/todos.css), add it to your project's `css` directory, then include it in your `index.html` `<head>` tag:

<html filename="index.html">
<link rel="stylesheet" href="css/todos.css">
</html>

### Creating New Todos with a Text Field

We've got our model and controller set up, so let's move on to the fun part: creating the interface for our users. The first step is to create a text field into which the user types the name of their todo.  SproutCore uses Handlebars templates to quickly define the application's interface.  While Handlebars makes it easy to markup HTML quickly, you'll see that it also has been extended to automatically take advantage of SproutCore's bindings.

To start building our views, let's open `index.html`.

First, add an `<h1>` tag with the name of the application:
  
<html>
<h1>Todos</h1>
</html>

You'll also see that the starter kit includes a default Handlebars template. Remove it and replace it with the following Handlebars template, which emits a text field for our users to type in:

<html filename="index.html">
<script type="text/html">
{{view SC.TextField id="new-todo" placeholder="What needs to be done?"}}
</script>
</html>

NOTE: Make sure you remove the stub template in index.html that is included in the starter kit.

Here we've specified a text field, with a unique `id` attribute (so it can be styled via CSS), as well as a `placeholder` attribute that will be displayed in modern HTML5 browsers.

NOTE: For more information about using Handlebars, visit the [Handlebars website](http://www.handlebarsjs.com/). To learn more about using Handlebars with SproutCore, make sure to check out the [Using Handlebars Templates](/using_handlebars.html) guide.

Now that we've got model, view, and controller represented, it's time to open the app in our browser and see how it looks.

Double click on your `index.html` file to open it in your browser. You should see your application load. Once you've verified that the application is up and running, it's time to tell SproutCore how to handle events for your `<input>` tag. When the user types in the field and presses the return key, we will create a new Todo and have it inserted into the content of the array controller.

NOTE: In SproutCore, view objects are responsible for updating the DOM and handling events. Among other things, this allows us to buffer changes to the DOM for maximum performance and to support generic cross-platform event handling.  Whenever you want to display dynamic content or handle events, you will use a view object.

<javascript filename="js/app.js">
Todos.CreateTodoView = SC.TextField.extend({
  insertNewline: function() {
    var value = this.get('value');

    if (value) {
      Todos.todosController.createTodo(value);
      this.set('value', '');
    }
  }
});
</javascript>

Since `CreateTodoView` will handle events for a text field, we create a subclass of `SC.TextField`, which provides several conveniences for working with these input controls. For example, you can access the `value` property and respond to higher level events, such as `insertNewline`, when the user presses `enter`.

Now that we have defined our view, let's update the template to use our new view subclass.  Switch back to `index.html` and update the view helper to say `Todos.CreateTodoView` instead of `SC.TextField`.

<html filename="index.html">
<h1>Todos</h1>
<script type="text/html">
{{view Todos.CreateTodoView id="new-todo"
  placeholder="What needs to be done?"}}
</script>
</html>

Now that we have UI to create new todos, let's create the code to display them.  We'll use the Handlebars `#collection` helper to display a list of items. `#collection` will create an instance of `SC.CollectionView` that renders every item in its underlying Array using the enclosed HTML.

<html filename="index.html">
{{#collection contentBinding="Todos.todosController" tagName="ul"}}
  {{content.title}}
{{/collection}}
</html>

Notice that we've also told the collection to bind its `content` property to our `todosController`. For every item in the array controller, the collection view will create a new child view that renders the `{{content.title}}` template.

NOTE: You set up bindings by creating a property whose name ends in `Binding`. In this case, we bind `Todos.todosController` to the collection view's `content` property. When one end of a binding changes, SproutCore will automatically update the other end.

This is a good time to open `index.html` again (or reload if you still have it loaded in your browser). It should look the same as before. Type a todo into the text field and hit return.

Look at that! As soon as we create a new todo and insert it into the array controller, the view updates automatically.

You've now seen a little bit of the power of SproutCore. By using SproutCore's bindings to describe the relationship between your data and your views, you were able to change the data layer and let SproutCore do the hard work of updating the view layer for you.

INFO: This is actually a core concept in SproutCore, not just something that demos well.  SproutCore's binding system is designed with the view system in mind, which makes it easy to work directly with your data and not need to worry about manually keeping your view layer in sync. You will see this concept over and over again in the rest of this tutorial and in other guides.

### Getting Things Done

We now have the ability to add todos, but no way to mark them as done. Before the frustration of a never-ending todo list gets the better of us, let's add the ability to mark todos complete.

The first thing we need to do is add a checkbox to each todo list item. As was mentioned earlier, if we want to handle events, such as user input, we need a view. In this case, we are adding a checkbox and want to be notified whenever the value of the checkbox is changed by the user. Let's update the Handlebars template to look like the following:

<html filename="index.html">
<!-- replacing the previous code -->
<script type="text/html">
{{view Todos.CreateTodoView id="new-todo" placeholder="What needs to be done?"}}

{{#collection contentBinding="Todos.todosController" tagName="ul"}}
{{view SC.Checkbox titleBinding="content.title"
    valueBinding="content.isDone"}}
{{/collection}}
</script>
</html>

Let's take a second to talk about the bindings we just set up. For every item in its underlying array, `SC.CollectionView` will create a new item view whose `content` property contains the object the view should represent. In our case, there will be a child view for each todo. Since the checkbox is a child view of each item view, we can access the parent via the `parentView` property. When we bind to `parentView.content.title`, we're saying to bind to the `title` property of the Todo object represented by the item view.

INFO: Under the hood, SproutCore binds an event handler to the `change` event of the checkbox and updates `value` when the event occurs. This may change as needed for browser compatibility, but working with a SproutCore property insulates you from those concerns.

Before you reload the browser to see the new checkbox todo list items, the stylesheet we provided includes a CSS class meant to give completed todos a unique look. Let's bind the class of each item to the object's `isDone` property.

We'll use a property on the collection helper to set up this binding:

<javascript filename="index.html">
<!-- replacing the previous code -->
{{#collection contentBinding="Todos.todosController" tagName="ul"
itemClassBinding="content.isDone"}}
{{view SC.Checkbox titleBinding="parentView.content.title"
    valueBinding="parentView.content.isDone"}}
{{/collection}}
</javascript>

This property sets up a binding on each of the item views. Each item will get the class `is-done` if its associated content object's `isDone` property is `true`. SproutCore will automatically dasherize property names into class names.

NOTE: All views have a number of properties, including `id`, `class`, and `classBinding`. The `collection` helper allows you to prefix any of those properties with `item`, which will then apply to each of the child item views. For instance, if you use the `itemClass` property on a collection, each item will get that class name.

Now reload your application in the browser and try it out. As soon as you click a todo's checkbox, the text will become crossed out. Keep in mind you didn't need to update any views when marking the `Todo` as done; bindings did it for you. If a different part of the app changes the `Todo` item's `isDone` property, your list item would automatically update without any more work on your part.

### The More You Know

We can now create todos and mark them as being complete. While 76% of all statistics are made up, let's see if we can display more accurate information from the data we have.  At the top of our list, let's display the number of todos remaining. Open `index.html` and insert this new view:

<html filename="index.html">
<!-- Insert this after the CreateTodoView and before the collection. -->
{{#view Todos.StatsView id="stats"}}
  {{remainingString}} remaining
{{/view}}
</html>

Handlebars expressions, like `{{remainingString}}`, allows us to automatically update the DOM when a property on our view changes. In this case, we're saying that `Todos.StatsView` should always display the most up-to-date value to the view's `remainingString` property.  As with the other bindings, this will be updated for us automatically whenever the value changes.

Let's go ahead and implement that view in `app.js` now:

<javascript filename="js/app.js">
Todos.StatsView = SC.View.extend({
  remainingBinding: 'Todos.todosController.remaining',

  remainingString: function() {
    var remaining = this.get('remaining');
    return remaining + (remaining === 1 ? " item" : " items");
  }.property('remaining')
});
</javascript>

`remainingString` contains a pluralized string, based on the number of remaining todos.  Here is another core piece of SproutCore in action, this one called a *computed property*. Computed properties are properties whose value is determined by running a function. For example, if `remaining` equals `1`, `remainingString` will contain the string `"1 item"`.

We say that `remainingString` *depends on* `remaining` because it requires another value to generate its own value. We list these *dependent keys* inside the `property()` declaration.

We've also bound the view's `remaining` property to the `todosController`'s `remaining` property. This means that, because of the dependency chain we've described, if `Todos.todosController.remaining` changes, `remainingString` will automatically update.

When we have information that views need but is based on aggregate information about our models, the array controller is a good place to put it. Let's add a new computed property to `todosController` in `todos.js`:

<javascript filename="js/app.js">
// updating previous code

Todos.todosController = SC.ArrayProxy.create({

  // ...

  remaining: function() {
    return this.filterProperty('isDone', false).get('length');
  }.property('@each.isDone')
});
</javascript>

Here, we specify our dependent key using `@each`. This allows us to depend on properties of the array's items. In this case, we want to update the `remaining` property any time `isDone` changes on a `Todo`.

NOTE: The `@each` property also updates when an item is added or removed.

It's important to declare dependent keys because SproutCore uses this information to know when to update bindings. In our case, our `StatsView` updates any time `todosController`'s `remaining` property changes.

Here's how it all fits together:

<div style='text-align: center'>
  !images/html_based/bindings.png!
</div>

INFO: As we build up our application, we create *declarative* links between objects. These links describe how application state flows from the model layer to the HTML in the presentation layer.

### Clearing Completed Todos

As we populate our list with todos, we may want to periodically clear out those we've completed. As you have learned, we will want to make that change to the `todosController`, and let SproutCore automatically propagate those changes to the DOM.

Let's add a new `clearCompletedTodos` method to the controller.

<javascript filename="js/app.js">
// updating existing code

Todos.todosController = SC.ArrayProxy.create({

  // ...

  clearCompletedTodos: function() {
    this.filterProperty('isDone', true).forEach(this.removeObject, this);
  }
});
</javascript>

Next, let's add a button to our template. Open `index.html` and add a button inside the HTML for `StatsView` as shown here:

<html filename="index.html">
<!-- Insert into existing StatsView -->

{{#view Todos.StatsView id="stats"}}
  {{#view SC.Button classBinding="isActive"
    target="Todos.todosController"
    action="clearCompletedTodos"}}
    Clear Completed Todos
  {{/view}}
  {{remainingString}} remaining.
{{/view}}
</html>

We've defined an instance of `SC.Button`, which calls a method on an object when it is clicked. In this case we've told the button to call the `clearCompletedTodos` method (its action) on the `Todos.todosController` object (its target).

We've also told it to add an `is-active` class to the view when it is clicked or tapped. Every `SC.Button` has an `isActive` property that will be true when it is in the process of being clicked. This allows us to display a visual cue to the user that they have hit the right target.

Go back to your browser and try it out. Add some todos, then mark them done and clear them.  Because we previously bound the visual list to the `todosController`, making a change through new means has the expected effect.

### Marking All as Done

Let's say you've decided to actually get all of your work done. Wouldn't it be nice to have a way to easily mark every todo as complete?

It turns out that, due to our application's declarative nature, all the hard work is already done. We just need a way to mark every `Todo` complete.

Let's first create a new computed property on our controller that describes whether or not every todo is done. It might look something like this:

<javascript filename="js/app.js">
// updating existing code

Todos.todosController = SC.ArrayProxy.create({

  // ...

  allAreDone: function() {
    return this.get('length') && this.everyProperty('isDone', true);
  }.property('@each.isDone')
});
</javascript>

INFO: SproutCore has many enumerable helpers. `everyProperty('isDone', true)` returns true if every item in the array has an `isDone` property that evaluates to `true`. You can find out more in the [Enumerables guide](enumerables.html).

Next, we'll create a checkbox view to mark all items complete and bind its value to the controller's property:

<html filename="index.html">
<!-- directly below the Todos.StatsView -->

{{view SC.Checkbox class="mark-all-done"
  title="Mark All as Done"
  valueBinding="Todos.todosController.allAreDone"}}
</html>

If you were to reload your browser and use the app now, you'd notice that clicking all of the checkboxes of your todos will cause the "Mark All as Done" checkbox to become checked. However, it doesn't work in the other direction: clicking "Mark All as Done" has no effect.

So far, our computed properties have described how to calculate a value from dependent properties. However, in this case, we want to accept a new value, and update dependent properties to reflect it.  Lets update our `allAreDone` computed property to also accept a value.

<javascript filename="js/app.js">
// updating existing code

Todos.todosController = SC.ArrayProxy.create({

  // ...

  allAreDone: function(key, value) {
    if (value !== undefined) {
      this.setEach('isDone', value);

      return value;
    } else {
      return this.get('length') && this.everyProperty('isDone', true);
    }
  }.property('@each.isDone')
});
</javascript>

When you set a computed property, your computed property function is called with the property key as the first parameter and the new value as the second. To determine if SproutCore is asking for the value or trying to set it, we examine the `value` parameter. If it is defined, we iterate through each `Todo` and set its `isDone` property to the new value.

Because bindings are bi-directional, SproutCore will set `allAreDone` to `true` when the user clicks the "Mark All as Done" checkbox. Conversely, unchecking it will make SproutCore set `allAreDone` to `false`, unchecking all todos.

Reload the app and add some todos. Click "Mark All as Done". Wow! Each of the individual check boxes gets checked off. If you uncheck one of them, the "Mark All as Done" checkbox un-checks itself.

INFO: When you use SproutCore, as you scale your UI, you never need to wonder whether a new feature will work consistently with parts of the UI you already implemented. Since you build your view layer to simply reflect the state of your models, you can make changes however you want and see them update automatically.

### Resources

Now that you have finished the first part of the Getting Started tutorial, you will want to acquaint yourself with the resources available to the SproutCore community.

 * Sign up for the [SproutCore Mailing List](http://groups.google.com/group/sproutcore)
 * Follow [@sproutcore](http://twitter.com/sproutcore) on Twitter
 * Visit the [#sproutcore](irc://irc.freenode.net/sproutcore) IRC channel to get real-time help
 * Bookmark the [SproutCore API Documentation](http://docs.sproutcore20.com) and the [SproutCore Guides](http://guides.sproutcore20.com)

### Changelog

 * March 1, 2011: initial version by [Tom Dale](credits.html#tomdale) and [Yehuda Katz](credits.html#wycats)
 * March 2, 2011: fixed formmating and added paths to filenames by [Topher Fangio](credits.html#topherfangio) and [Peter Wagenet](credits.html#pwagenet)
 * March 22, 2011: cleaned up demo based on new features by [Yehuda Katz](credits.html#wycats)
 * April 11, 2011: consistently use view classes and extend, update to reflect better Handlebars integration by [Yehuda Katz](credits.html#wycats) and [Tom Dale](credits.html#tomdale)
 * May 6, 2011: clarifications, minor inconsistency fixes, updated CSS for older browsers, plus new mobile section by [Tyler Keating](credits.html#publickeating)
 * May 9, 2011: update for recent changes in SproutCore 1.6 by [Tom Dale](credits.html#tomdale) and [Yehuda Katz](credits.html#wycats)
 * May 25, 2011: Modified for SproutCore 2.0 by [Tom Dale](credits.html#tomdale) and Majd Taby*
 * Aug 4, 2011: fixed wrong link to todos example app, reported by Chad Lung, fixed by Clemens Müller