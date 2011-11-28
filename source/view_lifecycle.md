## View Lifecycle

In this advanced guide, we'll discuss the nitty-gritty of how instances of SC.View are created, how they are converted from JavaScript into DOM representations, and how they are managed.

After reading this guide, you will have learned:

 * How templates in an HTML file are discovered and compiled.
 * How views progress through the rendering pipeline.
 * What happens if a view is modified in the middle of rendering.
 * The different states a view can be in.

endprologue.

### Discovering Handlebars Templates

When SproutCore loads, it registers a listener via `jQuery.ready()` that will look for any Handlebars templates inside the page's HTML. Once the DOM has finished loading, it uses jQuery to find any `<script>` tags whose `type` attribute is `text/x-handlebars`.

Templates can be either inline or named. To create a named template, give its `<script>` tag a `data-template-name` attribute. The contents of the `<script>` tag are passed to `SC.Handlebars.compile`, which returns a function that is saved in the global `SC.TEMPLATES` hash, keyed on the name you provide.

<div style='text-align: center'>
  !images/view_lifecycle/named_templates.png!
</div>

Inline templates do not have a name. Instead, a new view is created whose template is the compiled output of the text of the `<script>` tag. The view's DOM representation is scheduled to replace the `<script>` tag. (You will learn about view scheduling later in this guide.)

<div style='text-align: center'>
  !images/view_lifecycle/inline_templates.png!
</div>

### What is a SproutCore View?

Web applications are useful because they display important information to the user, then allow him to modify it. That information usually comes from a server  that transmits data to your application using JSON or some other data format.

Views are responsible for presenting information that makes sense to a computer in a way that makes sense to humans. They are also responsible for letting your program know when the user has tried to interact with that information.

To create the visual representation, properties of the SC.View instance are synthesized into a string of HTML markup. (Properties of the view that affect how it is displayed are called __display properties__.)

Views can also contain child views. When responding to user events, actions that are not handled by a child view will continue to bubble up to their parent until the top of the view hierarchy is reached.

### Render Buffers

Starting at the top of the hierarchy, views do a depth-first traversal of child views, asking each view in turn for the string representation of its HTML. Each view gets an instance of SC.RenderBuffer, which holds the strings of HTML that make up a view until it is converted into a DOM element.

Think of SC.RenderBuffer as an array of strings. As your view renders, you can call the `push()` method to push strings of HTML onto it.

Besides strings, render buffers also contain the render buffers of the view's children. This allows SproutCore to determine where the content of a child view should be rendered in a parent view.

Examine the diagram below. Each buffer has a `childBuffers` property, which is an array of either strings or other render buffers. As each child view is rendered, a new render buffer is created and inserted into the `childBuffers` array of the parent buffer.

<div style='text-align: center'>
  !images/view_lifecycle/render_buffers.png!
</div>

Once all of the views have finished writing to their render buffers, the buffers are joined into a single string of HTML. Once that string is created, it is converted into a DOM element using jQuery.

### Templates

Most views are backed by a Handlebars template. A template starts life as a string of Handlebars markup, which is compiled to a function. This function is saved to the view and invoked whenever the view needs to render (or re-render).

A template function writes a combination of HTML and JavaScript properties to a render buffer in order to create a fully-formed view. As it renders, it may also create and add new child views to the parent view.

For example, imagine Handlebars markup like this:

<html>
<b>My name is {{name}}.</b>
</html>

The compiled code would produce a function conceptually similar to this:

<javascript>
function(context, options) {
  // Get a reference to the view's render buffer
  var buffer = options.data.buffer;

  // Push some HTML
  buffer.push("<b>My name is");

  // Look up the 'name' property on the view and
  // add it to the render buffer.
  buffer.push(context.get('name'));
  buffer.push(".</b>");
}
</javascript>

The actual function produced is more complicated because of the work that happens under the hood to make values, like `{{name}}`, bindings-aware.

NOTE: The `context` parameter passed to the template function is the object from which properties should be looked up. Usually this is the view itself, but it can be changed by helpers such as {{with}}.

#### Templates & Child Views

By using the `{{view}}` helper, it is possible to specify an entire view hierarchy using a single Handlebars template. This helper creates a new instance of the specified view, then adds it as a child view of the parent.

For example, imagine the following template:

<html>
  Don't be fatuous, {{#view MyApp.UserView}}Jeffrey{{/view}}.
</html>

This is what the the template might compile to:

<javascript>
function(context, options) {
  // Get a reference to the view and
  // render buffer.
  var buffer = options.data.buffer,
      view   = options.data.view,
      newView;

  // Push HTML strings onto buffer.
  buffer.push("Don't be fatuous, ");
  
  // Since we were instructed to create a new view,
  // create an instance and set its template to the
  // provided HTML.
  newView = MyApp.UserView.create({

    // Set the template property to the compiled
    // result of the block.
    template: function(context, options) {
      var buffer = options.data.buffer;
      
      buffer.push('Jeffrey');
    }
  });
  
  // Call appendChild(), which 
  view.appendChild(newView);

  buffer.push(".");
}
</javascript>
(Again, note that the compiled version of the template is intended to be illustrative. The actual function will not look like this.)

We'll talk more about child views, and the `appendChild` method, later in this guide.