---
author: Gary Rowe
title: Dropwizard with Ember Data demo
layout: post
comments: true
tags:
  - HowTo
  - Dropwizard
redirect_from: /agilestack/2013/05/11/dropwizard-with-ember-data/
---
In this tutorial I'm going to create a reasonably complex demonstrator of [Ember Data](https://github.com/emberjs/data) against a [Dropwizard](http://dropwizard.codahale.com) RESTful API. It is intended to demonstrate the workflows associated with creating a self-contained Ember.js front-end served by a Dropwizard back end. Thus you will see how to use [Grunt.js](http://gruntjs.com/) as a kind of Maven for the front-end, and the role of Dropwizard in both serving the JavaScript application and handling the REST queries that subsequently come from the user interaction.

It will demonstrate creating a simple display of some blog posts presented in a paginated list with a link to see the full post content. The posts are held in a simple in-memory cache. All data is served through a simple REST API that adheres to the requirements of the Ember Data RESTAdapter.

If you're serious about learning Ember Data and using it within Dropwizard then you'll need to set aside a fair amount of time to learn all that goes with it. This tutorial will get you started down a very long, and rewarding, road.

Some readers do not have admin rights to their machines. For you there is an alternative version of the application that just uses [a single HTML file referencing some external libraries](https://github.com/gary-rowe/DropwizardEmberData/tree/master/src/main/ember/no_grunt). Nothing to install.

Dropwizard is rapidly becoming accepted as the best way to develop low complexity RESTful APIs in Java with speed and efficiency. You can get your RESTful API out to your customers faster and have it compliant with best practices much faster with Dropwizard than with any other Java approach. It glues together Jetty, Jersey, Guava, Metrics, FEST and more with a simple YAML configuration file that is production ready out of the box.

Ember.js (just Ember from now on) provides a comprehensive Model-View-Controller framework that makes building complex JavaScript applications much easier. When linked with Ember Data it creates a powerful way to visualise the
output of your REST services.

Essentially Ember provides an efficient way to create a client-side state machine with your API acting as a data layer.

### I'm in a hurry where's the code?

I've put together a [simple starter kit for Ember Data](https://github.com/gary-rowe/ember-data-starter-kit) on GitHub. You won't need anything else other than a web server to make it run. It doesn't do much but serves as a gentle introduction to Ember.

This article deals with the more complex application [Dropwizard Ember Data Demo (DEDD)](https://github.com/gary-rowe/DropwizardEmberData). It is intended to provide a useful starting point for Ember projects so I'll build out other useful features (like OpenID and User account support) as I get time (and feedback).

### JavaScript inheritance for Java developers

Many of the readers of this blog are from a Java background rather than JavaScript. Consequently I'll put in a bit of background to cover the basics of how objects are handled in JavaScript since they are used extensively in the Ember framework. I'm going to assume that you've done a bit of basic JavaScript programming, perhaps a few functions with jQuery, but haven't progressed as a far as implementing a single page web application using a client-side web MVC framework. For more detail please see this [informative tutorial by Toby Ho](http://tobyho.com/2011/11/11/js-object-inheritance/) and then perhaps go on to read the [Mozilla Introduction](https://developer.mozilla.org/en-US/docs/JavaScript/Introduction_to_Object-Oriented_JavaScript) once the core concepts are in place. What follows is a condensed version that may be sufficient if you just need a quick refresher.

### Under the covers with `__proto__`

First some technical background. JavaScript is not an object-oriented programming language but it does support first-class functions. This allows it to emulate some object-oriented behaviour such as a special form of inheritance known as Prototype Inheritance. This is different to the classical inheritance found in Java and can be thought of as a variant of composition. The rules are:

1. If `B` "extends" `A` then `A` is `B`'s "prototype" (different semantics to a superclass so requires a different term)
2. If `A` has a property `x` then `B.x == A.x`
3. If `B.x` is set then it overrides `A.x`

To see this working you will need a JavaScript console. Fire up Firebug or Chrome Developer Tools (this won't work in IE for reasons that will become apparent later) and select the console option. Then just copy paste the following lines into the input field and click Run:

```javascript
var john = {firstName: 'John', lastName: 'Smith'} // We're creating a hashmap
var jane = {firstName: 'Jane'} // and another one
jane.__proto__ = john // then linking them together
jane.lastName // and giving some output
```
So we can now say that `john` is `jane`'s prototype, and `jane` inherits `john`'s properties. 

Try to work out what will be the output of this additional sequence of commands:

```javascript
jane.lastName = 'Doe' // jane changes her last name
delete jane.lastName // then changes her mind
jane.lastName // what happens?
```

Answer: `"Smith"` because `jane` will revert back to using `john.lastName`.

So now we know that JavaScript doesn't have classes, but does support prototypes.

Unfortunately, `__proto__` is unlikely to remain supported in modern browsers so we have to construct prototypes using a different technique based on functions.

### Using Object.create()

From JavaScript 1.8.5 (ECMAScript 5th Edition) onwards, the `create()` method is supported. This greatly simplifies the syntax of working with prototypes. Try this in your console:

```javascript
function Shape() {
  this.x = 0;
  this.y = 0;
}
```

This creates a function with some properties dynamically declared and initialised. Every function comes with an implicit prototype which we reference through `.prototype`. Thus to add a `move(x,y)` method to the `Shape` prototype we express it like this:

```javascript
// Add the move() method to the prototype making it available to inheritors
Shape.prototype.move = function(x, y) { 
    this.x += x;
    this.y += y;
    console.info("Shape moved.");
};
```
A rectangle is a kind of shape, so we express this by creating a function that simply calls the `Shape` constructor passing in the reference to the `Rectangle`:

```javascript  
function Rectangle() {
  Shape.call(this); // call super constructor.
}
Rectangle.prototype = Object.create(Shape.prototype); 
```
The reason for the final line is so that `rect instanceof Rectangle` will give the correct answer (see later). Without it `Rectangle.prototype` will assume that it is the less specific `Shape`. Let's make some instances of `Shape` and `Rectangle` to try it out.

```javascript
var shape = new Shape();
var rect = new Rectangle();
 
console.info("Is shape an instance of Shape? " + (shape instanceof Shape));
console.info("Is shape an instance of Rectangle? " + (shape instanceof Rectangle));

console.info("Is rect an instance of Rectangle? " + (rect instanceof Rectangle));
console.info("Is rect also an instance of Shape? " + (rect instanceof Shape));
 
rect.move(); 
```
The final call to the `move()` method shows that `Rectangle` has inherited from `Shape` as we require.

### OK, I get JavaScript objects, tell me about Ember

Ember provides a Model-View-Controller framework with an additional routing structure that is used to maintain application state client-side (so your REST API remains stateless like it should be). I'll quickly summarise the role of each component so you can quickly recognise code that belongs in each category.

Don't try to analyse the example code given here beyond just reading the comments. I'll go into much more detail later on. For now a quick skim and a nod is good enough.

### Model

The Model provides the raw data obtained from the REST API (via the `Store`) and some computed properties. A typical
example would be:

```javascript
App.Post = Ember.Post.create({
  // A standard string
  title: DS.attr('string'); 
  // Default format is HTTP format (Sat, 01 Jan 2000 00:00:00 GMT) but can be changed
  dateCreated: DS.attr('date'); 
});
```

Typically Models don't have a lot of extra properties on them since they act as Data Transfer Objects (DTOs). Never put any state related to a View logic in them - that goes in the Controller.

Models are usually placed in `app/models/*.js`.

### View

This is where the HTML gets wrapped around your data. Ember uses a templating language called Handlebars to generate HTML elements using data provided to the View. 

{% raw %}
```html
<div> 
  <!-- Iterate over controller.content (default if not explicitly referenced)
       referencing each item as "post" 
   -->	
  {{#each post in controller}}
    <div>
      <!-- Build a link for the posts.selectedPost route
           Include a post Model so that path parameters can be filled in
       -->	 
      {{#linkTo posts.selectedPost post}}
        <!-- Output the post title --> 
        {{post.title}}
      {{/linkTo}}
    </div>
  {{/each}}
</div>
```
{% endraw %}
A View will not directly alter any state. Instead it will provide links that permit those actions to take place. For complex views a simple Handlebars template may not be sufficient and so you will need to fall back to JavaScript:

```javascript
App.PostSummaryView = Ember.View.extend({
  tagName: 'post',
  attributeBindings: ['src'],
  classNames: ['summaryItem'],
  classNameBindings: 'isSelected',

  ... more properties ...

});

```
The above is only a taste of the sophistication that can be applied to view rendering. For more details the user is directed to the [Ember documentation](http://emberjs.com/api/modules/ember-views.html).

Views are usually placed in `app/views/*.js` or in `app/templates/*.hbs`.

### Controller

This is where you put you put the meat of your application. Each Controller is responsible for providing access to Model data (and logic on top of it) so that Views can render the output.

```javascript
App.PostsController = Ember.ArrayController.extend({

 findSelectedItemIndex: function() {
     ... some work with models ...
 }

});
```

Typically methods on Controller classes are named in line with their purpose rather than a simple property accessor.

Controllers are usually placed in `app/controllers/*.js`.

### Mixins

Mixins are an advanced topic within Ember. Typically you'd use a Mixin to provide additional functionality that isn't present within the base classes of Ember itself. 

At the time of writing pagination support isn't directly available so I've opted to provide it to the `PostsController` via a Mixin.

```javascript
App.PostsController = Ember.ArrayController.extend(Ember.PaginationMixin, {
  itemsPerPage:3,
  pagesPerControl:5
});
```

Mixins are usually placed in `app/mixins/*.js`.

### Routes

I mentioned at the start of this article that Ember provides an easy way to create a client-side state machine. Think of a Route as the state transition mechanism. It performs the same function as a URI except that the URI is resolved locally within the page via a fragment identifier (the bit at the end of a URI beginning with `#`). Since URI fragments are not intended to be sent to the server they are an excellent choice for this kind of client-side state management. 

You'll commonly see Ember URIs that look similar to `http://example.org/index.html#/posts/page/1`. This provides a clean separation between the server state (`/index.html`) and the client state (`/posts/page/1`).

```javascript
App.Router.map(function (match) {
  
  // IndexController -> "/" via a generated "App.IndexRoute"
  this.route("index", {path: "/"}); 

  // PostsController -> "/posts" via "PostsRoute"
  this.resource("posts", { path:"/posts" }, function () {  
    
    // PostsSelectedPageRoute/Controller -> "/posts/page/{id}" 
    this.route("selectedPage", { path:"/page/:page_id" }); 

    // PostsSelectedPostRoute/Controller -> "/posts/post/{id}"
    this.route("selectedPost", { path:"/post/:post_id" }); 
  });
});


// Override the generated App.IndexRoute with this one
App.IndexRoute = Ember.Route.extend({
    redirect: function() {
        // Make this route redirect to the named "posts" resource
        this.transitionTo("posts"); 
    }
});

```

Route code contains a bunch of maps between a path template and a named Route. Since most of the time this code is just going to be boilerplate, Ember will generate it for you following naming conventions. A `resource()` is used to group several `route()` entries under a single common root. This gives rise to lengthy names for Routes (and subsequent Controllers) as you can see from the comments in the code snippet above.

On occasion you may want the route to do something a bit different - perhaps performing a redirect to another Controller (e.g. make "/" redirect to "/posts"). At this point you should be able to reason about this as follows:

> I see that path "/" maps to "index". This implies the existence of a generated IndexRoute which will look up IndexController. I want to change that behaviour so I will implement IndexRoute and make it transition to "posts" which will make it target the PostsRoute instead.

The Router map is usually placed in `app/routes/router.js`, and each Route has its own `.js` file. Often Routes cluster on a named Model (e.g. `app/routes/post.js` contains all the `Post*Route` classes).

### Application

This is where everything is bound together. In a larger scale application you will find the top-level `require()` statements here so that all the appropriate libraries are referenced. Normally the `Ember.Store` instance is configured here as well. An (incomplete) snippet could look like this:

```javascript
require('dependencies/jquery');
require('dependencies/handlebars');
require('dependencies/ember');
require('dependencies/ember-data');

require('dependencies/compiled/templates');

// Create the application
App = Ember.Application.create({
  LOG_TRANSITIONS: true,
  rootElement: '#myapp'
});

App.store  = DS.Store.create({
  adapter:  "DS.RESTAdapter",
  revision: 12
});

```

#### The flexible `RESTAdapter`

In the above example the Ember Data `RESTAdapter` has been added. The `RESTAdapter` provides a lot of useful additional functionality to make interfacing with RESTful APIs a lot easier. Here are a couple of examples that may solve common problems:

```javascript
// Map all requests to /api/{usual Ember path structure}.json
App.Adapter = DS.RESTAdapter.extend({
  url: '/api',

  buildURL: function(record, suffix) {
    return this._super(record,suffix) + '.json'
  }
});

// and attach it to the main App
App.store = DS.Store.extend({
  revision: 12,
  adapter: App.Adapter.create({})
});

```

In Dropwizard with Views you frequently want to serve static assets from `/` through the AssetBundle. To do this you must first configure `http.rootPath = /api` in the YAML file so that your Resources don't conflict with your assets.

You may also want to customise the default date format (mentioned earlier in the Model section) away from HTTP and into ISO 8601. This is done as follows:

```javascript
DS.RESTAdapter.registerTransform("isodate", {
  deserialize: function(serialized) {
    return serialized;
  },

  serialize: function(deserialized) {
    return deserialized;
  }
});
```

### Where do I put all this in a Dropwizard project?

There is no convention, but I have found that placing all Ember code under `src/main/ember` allows me to reason more clearly about the role of Ember in regard to the larger role of the REST API within a Maven build environment. 

You might decide to put it all under `src/main/resources/ember`. Doing this will make Maven copy the entire collection of source files (`package.json` and all) into the `target/classes` directory. Obviously some filtering can be applied to Maven resources plugin to avoid this. However, when you start developing non-trivial applications you will find that you will start [compiling LESS into CSS](http://lesscss.org/) and combining all your JavaScript files into a single minified download. 

This is the work of Grunt rather than Maven.

To enforce this strong separation between the source code and the build output `src/main/ember` is a better fit. 

In summary an Ember app looks like this:

* `/` - Various files supporting Grunt 
* `/app` - The app.js file that contains the overall application
* `/app/models` 
* `/app/views` 
* `/app/templates`
* `/app/mixins`
* `/app/controllers`
* `/app/routes`

### What's my workflow?

There are two distinct workfows when working with both Dropwizard and Ember on the same project: server- side and client-side. Ember will only see the API exposed through the Resources you have created so we won't concern ourselves with the deeper code backing them (business processes, DAOs etc).

### Server-side

For the Dropwizard side of things you'll probably have a Resource workflow like this:

1. Invent some JSON that represents the output of the Resource you want to write and store it under `src/test/resources/fixtures`
1. Write a `ResourceTest` for your Resource endpoint referencing the fixture
1. Write the corresponding method on the Resource (I tend to work in order of `POST`, `GET`, `PUT`, `DELETE`)
1. Rinse and repeat until the fixture and Resource are correct and all tests are green

You would then fire up the app, fixing any problems and verify the endpoint with some REST plugin for your browser. I like the [Advanced REST Client](https://chrome.google.com/webstore/detail/hgmloofddffdnphfgcellkdfbfbjeloo)
plugin for Chrome for my day to day development.

### Client-side

At this point the traditional approach is to have Dropwizard use Views to serve templated HTML via Freemarker for the client to render. These are usually stored under `src/main/resources/views/ftl`. You would then likely perform the following:

1. Fire up Dropwizard and leave it running
1. Make a change to something under `src/main/resources` (manual save or autosave on focus change)
1. Use CTRL+F9 (in Intellij) to trigger a Build resulting in a resource copy so that `AssetBundle` can update
1. Refresh the browser (CTRL+R) to see the result

This grates on me since it could easily benefit from some optimisation. Consider what happens once Grunt is introduced. The `Gruntfile.js` is adjusted so that the Neuter task outputs to `target/classes/app` and the workflow becomes:

1. Fire up Dropwizard and leave it running
1. Fire up Grunt and leave it running
1. Make a change to something under `src/main/ember/app` (with save triggered on focus loss)
1. Refresh the browser

This works because Grunt is continually watching your source files and automatically rebuilding the `app/application.js` file and assets with the changes behind the scenes. The resulting output is then copied to 'target/classes/app' where it can be immediately served by the `AssetBundle`. 

You can have Dropwizard serve up the static assets like images and so on as well. I tend to use a structure like this under `src/main/ember`:

```text
/app
/less - LESS files to generate CSS
/images - Images for the app
```

### So how do I "Think in Ember"?

When building up an Ember application it is best to approach it iteratively. Start with the smallest workable piece of code and build out from there. I find that the following approach works best for me:

1. Code up the server side so that the REST API is solid for one entity (e.g. a `Post` and a `PostList`)
2. Add the Node, Grunt and Ember boilerplate code for rapid iteration
3. Code up a single Ember Model for the `Post` (the `PostList` will be automatic)
4. Code up the Routes so that you can reason about the different state transitions for the view (how to build links)
5. Code up the Controllers since they immediately follow from the Routes
6. Finally you can introduce the View which is where most of the fiddly work takes place

### How to implement a paginated master-detail collection in Ember

To illustrate the above process we'll use it in the context of creating a simple paginated list of Posts with clickable titles and a summary. The act of clicking the title will cause the selected Post to be displayed next to the list. This is a classic example of a paginated master-detail list.

### Code up the server side

Since this is a tutorial about Ember rather than Dropwizard, I'll shortcut much of the server-side work and just point out some common gotchas. First we'll implement the simplest possible Post (title, summary and body). In Dropwizard this is our representation class and would be present under `org.example.dedd.api` in the Java package structure. It is nothing more than a value object with JSON annotations. 

It should be noted that in a more complex application there would normally be a similarly named domain object under `org.example.dedd.core.domain` which provides persistence annotations and offers much more domain-specific processing logic. Think of the API `Post` as being a public representation (simplification) of the private domain.

```java
package org.example.dedd.api;

@JsonSnakeCase
@JsonRootName("post")
public class Post {

  @JsonProperty
  private Long id;

  @JsonProperty
  private String title;

  @JsonProperty
  private String summary;

  @JsonProperty
  private String body;

  ... getters/setters elided ...

}
```

Note the use of `@JsonSnakeCase` and `@JsonRootName`. These are necessary to ensure that Dropwizard configures Jackson to de/serialize the JSON in accordance with the requirements of Ember.

It is also necessary to include a wrapper around a `List<Post>` that I'll call `PostList` so that the JSON representation is suitable for Ember Data. The Resource would look like this for a `find()` and `find(1)` from Ember Data:

```java

  /**
   * Retrieve all posts (typically a paginated subset)
   *
   * @return A PostList wrapper
   */
  @GET
  @Timed
  public PostList findAll() {
    return readService.all();
  }

  /**
   * Retrieve a single post by ID
   *
   * @return The single post
   */
  @GET
  @Timed
  @Path("/{id}")
  public Post find(
    @PathParam("id") Integer id) {

    Optional<Post> optional = readService.find(id);

    if (optional.isPresent()) {
      return optional.get();
    }

    throw notFound();

  }

```

We can now address the client-side work.

### Add the Node, Grunt and Ember boilerplate

This is the point where you build out your `src/main/ember`. When preparing for a large scale application you would just copy everything from the example except `src/main/ember/app` and run `npm install`. 

Earlier in the article I mentioned a simplified structure consisting of a single HTML file referencing some external JavaScript files. I'll use that structure here so that it makes it easier to follow along. You can find this condensed version in the `src/main/ember/no_grunt` directory or you can build it up from the snippets in the following sections.

The boilerplate covers creating an Ember Application and configuring the Store to use the REST adapter when locating Models:

```javascript
// Application
App = Ember.Application.create({
  LOG_TRANSITIONS:true,
  rootElement:'#dedd'
});

// Store
App.Adapter = DS.RESTAdapter.extend({
  namespace:'api'
});
App.Store = DS.Store.extend({
  revision:12,
  adapter:App.Adapter.create({})
});
```
Normally placed in `src/main/ember/app/models/app.js`. 

### Code up a single Ember model

```javascript
App.Post = DS.Model.extend({
    title: DS.attr('string'),
    summary: DS.attr('string'),
    body: DS.attr('string')
});
``` 
Normally placed in `src/main/ember/app/models/post.js`. 

### Code up the Routes

In Ember the Router is the entry point to implement this. Recall that a Router will associate a URL with one or more route handlers. These route handlers are responsible for:

1. mapping the URL to a particular Model (or collection of Models)
2. configuring a Controller with that data (setting its `content` property which is the default if nothing is specified in the View)
3. triggering the rendering of a View bound to the Controller

To begin with we'll need a default view so that the user can immediately begin interacting with it. The global index route (`/`) should transition to a list of Posts (`/posts`). 

Each Post should be identified using a "dynamic segment" which is essentially a URI template that describes where dynamic content will be placed. In JAX-RS we'd express this as `/posts/post/{id}` in Ember this becomes `/posts/post/:post_id`.

(The `/post` segment is redundant but serves to illustrate a naming point later on).

In the case of a master-detail view the user will see a paginated list, so we will need some kind of paging mechanism for the Post. We'll introduce an additional path segment to the template `/posts/page/:page_id`. 

Expressing the above in Ember we get:

```javascript
// Routes
App.Router.map(function (match) {
  // IndexRoute/Controller -> "/" 
  this.route("index", {path: "/"}); 

  // A resource() groups many related route() entries
  // PostsRoute/Controller -> "/posts" 
  this.resource("posts", { path:"/posts" }, function () {  
    
    // PostsSelectedPageRoute/Controller -> "/posts/page/{id}" 
    this.route("selectedPage", { path:"/page/:page_id" }); 
    
    // PostsSelectedPostRoute/Controller -> "/posts/post/{id}"
    this.route("selectedPost", { path:"/post/:post_id" }); 
  });
});

```

Since we don't want the default generated behaviour for the above Routes we need to create them for ourselves. Note the naming convention includes the base Route name (`PostsRoute`) and then expands it (`PostsSelectedPostRoute`).

```javascript
// Override the generated App.IndexRoute with this one
App.IndexRoute = Ember.Route.extend({
    redirect: function() {
        // Make this route redirect to the named "posts" route
        this.transitionTo("posts"); 
    }
});

// Handles /posts
App.PostsRoute = Ember.Route.extend({
  model:function (params) {
    // Set a default page number 
    this.controllerFor('posts').set('selectedPage', 1);
    // Get the posts from the Store
    return App.Post.find();
  }
});

// Handles /posts/page/:page_id
App.PostsSelectedPageRoute = Ember.Route.extend({
  model:function (params) {
    // Create a model on the fly containing just the page ID
    return Ember.Object.create({id:params.page_id});
  },
  setupController:function (controller, model) {
    // Find the PostsController and set its selected page to the page ID
    // This is used by the PaginationMixin
    this.controllerFor('posts').set('selectedPage', model.get('id'));
  }
});

// Handles /posts/post/:post_id
App.PostsSelectedPostRoute = Ember.Route.extend({
  model: function(params) {
    // Find a Post with the given ID
    return App.Post.find(params.post_id); 
  }
});
```

Normally placed in `src/main/ember/app/routes/post.js`. 

### Code up the Controllers (and provide a Mixin)

The `PostsRoute` infers the existence of a `PostsController`. This should have a `selectedPage` property set to whatever was provided in the URI (`posts/page/:page_id`). This Controller is responsible for providing various properties that the View will need in order to display a paginated list. The Controller will need to provide:

* `currentPage`, `prevPage` and `nextPage` so that links can be made
* `availablePages` so that the maximum page count can be obtained
* `pages` so that a range of page numbers can be shown
* `paginatedContent` so that only a subset of the overall Posts will be available to the View
* `disablePrev` and `disableNext` to ensure we don't go outside the bounds of the range

All of the above is not unique to the behaviour of the `PostsController` so it makes more sense to implement it as a common base class. However we still need the standard Ember Controller base class. This is a classic use case for Protoype Inheritance which allows for multiple inheritance. 

```javascript

// Extends the PaginationMixin which provides standard methods for
// page navigation for use by the views
App.PostsController = Ember.ArrayController.extend(Ember.PaginationMixin, {
  itemsPerPage:3,
  pagesPerControl:5
});
```

Normally placed in `src/main/ember/app/controllers/post.js`. 

The code for the `PaginationMixin` was inspired by [this StackOverflow answer by Toran Billips](http://stackoverflow.com/a/13033425/396747) and is as follows:

```javascript
var get = Ember.get, set = Ember.set;

Ember.PaginationMixin = Ember.Mixin.create({

  pages:function () {

    var availablePages = this.get('availablePages');
    var currentPage = this.get('currentPage');
    var pagesPerControl = this.get('pagesPerControl');
    var pages = [];
    var page;
    var start = 0;
    var end = availablePages;

    var offset = Math.ceil(pagesPerControl / 2);

    // Scroll? (Start at page 2, 3 etc)
    if (currentPage - offset > 0) {
      start = currentPage - offset;
    }
    // End point
    if (start + pagesPerControl < availablePages) {
      end = start + pagesPerControl;
    }
    // Reached end?
    if (start + pagesPerControl >= availablePages) {
      start = availablePages - pagesPerControl;
    }

    for (var i = start; i < end; i++) {
      page = i + 1;
      pages.push({ page_id:page.toString() });
    }

    return pages;

  }.property('currentPage', 'availablePages'),

  currentPage:function () {

    return parseInt(this.get('selectedPage'), 10) || 1;

  }.property('selectedPage'),

  nextPage:function () {

    var nextPage = this.get('currentPage') + 1;
    var availablePages = this.get('availablePages');

    if (nextPage <= availablePages) {
      return Ember.Object.create({id:nextPage});
    } else {
      return Ember.Object.create({id:this.get('currentPage')});
    }

  }.property('currentPage', 'availablePages'),

  prevPage:function () {

    var prevPage = this.get('currentPage') - 1;

    if (prevPage > 0) {
      return Ember.Object.create({id:prevPage});
    } else {
      return Ember.Object.create({id:this.get('currentPage')});
    }

  }.property('currentPage'),

  firstPage:function () {

    return "1";

  }.property(),

  availablePages:function () {

    return Math.ceil((this.get('content.length') / this.get('itemsPerPage')) || 1);

  }.property('content.length','itemsPerPage'),

  paginatedContent:function () {

    var selectedPage = this.get('selectedPage') || 1;
    var upperBound = (selectedPage * this.get('itemsPerPage'));
    var lowerBound = (selectedPage * this.get('itemsPerPage')) - this.get('itemsPerPage');
    var models = this.get('content');

    return models.slice(lowerBound, upperBound);

  }.property('selectedPage', 'content.@each'),

  /**
   * Computed property to determine if the previous page link should be disabled or not.
   * @return {Boolean}
   */
  disablePrev:function () {
    return this.get('currentPage') == 1;
  }.property('currentPage'),

  /**
   * Computed property to determine if the next page link should be disabled or not.
   * @return {Boolean}
   */
  disableNext:function () {
    return this.get('currentPage') == this.get('availablePages');
  }.property('currentPage', 'availablePages')

});

```

Much of the above should now be starting to make sense. Outside of the standard JavaScript class definition you will notice the use of "computed properties". These are part of the Ember infrastructure that enables the View to update in response to changes in the underlying Model. The mechanism that enables this is appending `.property()` to the function definition and including the names of all external properties that are referenced within the function. This enables Ember to trigger updates at the appropriate times.

Normally placed in `src/main/ember/app/mixins/pagination.js`. 

### Code up the Views

All that is left is to create suitable templates to render the data provided by the Controllers. In a large scale application these would be provided by Handlebars templates provided under `src/main/ember/app/views`, but for this example we're going to use the alternative approach which is to simply include the view templates directly within a `script` element in a HTML page.

#### Begin with `index.html`

I like [Twitter Bootstrap](http://twitter.github.io/bootstrap/) since it provides an attrative user interface out of the box. 

To that end here is the `index.html` that will provide all the boilerplate we need to present our app, without any of the view templates. You could use this as a generic starting point for any application:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Dropwizard Ember Data Blog</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="">
  <meta name="author" content="">

  <!-- CSS -->
  <link href="/bootstrap/css/bootstrap.css" rel="stylesheet">
  <style>
    body {
      padding-top: 60px; /* 60px to make the container go all the way to the bottom of the topbar */
    }
  </style>
  <link href="/bootstrap/css/bootstrap-responsive.css" rel="stylesheet">

  <!--[if lt IE 9]>
  <script type="text/javascript" src="//html5shiv.googlecode.com/svn/trunk/html5.js"></script>
  <![endif]-->

  <link rel="shortcut icon" href="/favicon.ico">
</head>

<body>

<div class="navbar navbar-inverse navbar-fixed-top">
  <div class="navbar-inner">
    <div class="container">
      <button type="button" class="btn btn-navbar" data-toggle="collapse" data-target=".nav-collapse">
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
      </button>
      <a class="brand" href="#">Dropwizard Ember Data Blog</a>

      <div class="nav-collapse collapse">
        <ul class="nav nav-pills">
          <li class="active"><a href="#">Home</a></li>
          <li><a href="#write">Write</a></li>
          <li><a href="#about">About</a></li>
          <li class="divider-vertical"></li>
          <li class="dropdown">
            <a class="dropdown-toggle" data-toggle="dropdown" href="#">API<b class="caret"></b></a>
            <ul class="dropdown-menu">
              <li><a href="/api/posts">All Posts</a></li>
              <li><a href="/api/posts/1">Post 1</a></li>
            </ul>
          </li>
        </ul>

      </div>
      <!--/.nav-collapse -->
    </div>
  </div>
</div>

<div class="container">

  <!-- Ember demo code starts -->

  <section id="dedd">

     <!-- TODO Add some Ember templates here -->

  </section>

  <!-- Ember demo code ends -->

</div>
<!-- /container -->

<!-- Placed at the end of the document so the pages load faster -->

<!-- The order is important here -->
<script type="text/javascript" src="/jquery/jquery.js"></script>
<script type="text/javascript" src="/bootstrap/js/bootstrap.js"></script>
<script type="text/javascript" src="/ember/handlebars.js"></script>
<script type="text/javascript" src="/ember/ember.js"></script>
<script type="text/javascript" src="/ember/ember-data.js"></script>
<script type="text/javascript" src="/ember/pagination.js"></script>
<script type="text/javascript" src="/app.js"></script>

</body>
</html>

```
The above assumes that you've been placing the code snippets into a file `app.js` and it is being served by Dropwizard in some manner.

Normally placed in `src/main/ember/app/assets/index.html`. 

#### Add the application output

All the output from the application will be presented inside the `section` element so our first step is to provide Ember with a top level view that simply renders any output.

```html
<!-- The application outlet -->
<script type="text/x-handlebars" data-template-name="application">
  {{outlet}}
</script>

```
Starting from the beginning we notice that the script type is unusual in that it specifies a `text/x-handlebars`. It also uses a custom HTML5 attribute `data-template-name` which Ember uses to map it to a View. 

The `{{outlet}}` directive is where all output for this template will go. Any views within the application will now be displayed.

Normally placed in `src/main/ember/app/templates/application.hbs`. 

#### Preparing the posts template

The next step is to present the paginated list of Posts.

```html
<script type="text/x-handlebars" data-template-name="posts">

  <div class="row">
    <div class="span12">
      <div class="row">
        <div class="span6">

          <!-- TODO Add the post list -->

          <!-- TODO Add the pagination controls -->

        </div>
        <div class="span6">
          <!-- Present the output from the post selection -->
          {{outlet}}
        </div>
      </div>
    </div>
  </div>

</script>

```
This template will be used to render the output of the "posts" View. We'll address each TODO in turn.

Normally placed in `src/main/ember/app/templates/posts.hbs`. 

#### Adding post list

We require an iterator over all the paginated Models and links to individual Posts for more detailed rendering. Consider the following:

{% raw %}
```html
<div>
  {{#each post in controller.paginatedContent}}
  <h3>{{#linkTo 'posts.selectedPost' post}}{{post.title}}{{/linkTo}}</h3>

  <p>{{post.summary}}</p>
  {{/each}}
</div>
``` 
The `{{#each }}` directive handles iterating over a collection. In this case each item is referenced by `post` and is supplied from the Controller for the View. This is inferred from the name of the View template (`data-template-name="posts"`) and is therefore the `PostsController.paginatedContent` property.

The `{{#linkTo }}` directive references a Route name and passes in a model as a parameter. This builds up the link matching up the path and any dynamic segments. Thus we get `posts/post/1`, `posts/post/2` and so on.

We now have a simple list of Posts displayed but we need the pagination to allow us to navigate past the default page 1. 
{% endraw %}

#### Adding the pagination controls

We're looking for something like:

> << 1 |2| 3 4 5 >>

which is represented in Bootstrap as an unsigned list with particular `class` attribute values to indicate "active" (the current selection) and "disabled" (greyed out). We'll dive right in with this:

{% raw %}
```html
<div class="pagination pagination-centered">
    <!-- contentBinding maps to App.PostController.prevPage in PaginationMixin -->
    <ul>
      <li {{bindAttr class="controller.disablePrev:disabled"}}>{{#linkTo 'posts.selectedPage' prevPage target="controller"}}&laquo;{{/linkTo}}</li>
      {{#each pages}}
    
      {{view App.PaginationView contentBinding="this" selectedPageBinding="controller.currentPage"}}
      
      {{/each}}
      <li {{bindAttr class="controller.disableNext:disabled"}}>{{#linkTo 'posts.selectedPage' nextPage target="controller"}}&raquo;{{/linkTo}}</li>
    </ul>
</div>
```
There is a lot going on there so let's break it down. The `{{bindAttr}}` directive is used to set the value of a named HTML attribute. In this case the setting will come from `PostsController.disablePrev` and will be `disabled` if the response is `true`. 

Next up is the familiar `{{#linkTo}}` directive that references the `posts.selectedPage` Route with the ID value provided by `PostsContoller.prevPage`.

Skipping over the `{{#each}}` section we have a closing section that does the same for the `nextPage`.

Returning to the `{{#each}}` directive we see that there is some complex presentation work required. To get the page numbers correct it is necessary to build out some more `<li>` tags and populate them with page IDs, but we must also ensure that the "active" class attribute is set. 

For the sake of demonstrating alternatives we'll do this with a dedicated View template backed by a View class to handle the complex behaviour. First the View class:

```javascript
App.PaginationView = Ember.View.extend({
  // This View backs the pagination template
  templateName:'pagination',
  // It builds <li> tags
  tagName:'li',
  // It will set class="active" or nothing depending on the activeCurrent property
  classNameBindings: ['activeCurrent:active:'],

  /*
   * Computed property
   * Returns a simple model containing the page ID
   */
  page:function () {
    // The View has a default "content" from its Controller
    return Ember.Object.create({id:this.get('content.page_id')});
  }.property('page_id'),

  /*
   * Computed property
   * Returns true if the selected page ID is the same as the current page ID so
   * that the UI can show highlighting
   */
  activeCurrent:function () {
    var currentPage = parseInt(this.get('content.page_id'));
    var selectedPage = this.get('selectedPage');
    return selectedPage == currentPage;
  }.property('content.page_id','selectedPage')

});

```
We then hook the View class to its template like so:

```html
<script type="text/x-handlebars" data-template-name="pagination">

  {{#with view}}
  {{#linkTo 'posts.selectedPage' page}}
  {{content.page_id}}
  {{/linkTo}}
  {{/with}}

</script>
```
The `{{#with view}}` directive changes the context of template away from the default `PaginationController` to the `PaginationView` instead. In the `{{#linkTo}}` directive the `page` model is therefore `PaginationView.page`. The `{{content.page_id}}` in turn references `PaginationView.content.page_id`.

There are no HTML tags present in the teplate since this View will handle that automatically.

Normally placed in `src/main/ember/app/templates/pagination.hbs`. 
{% endraw %}
#### Adding the detail view

There is one last View template required: the Post detail. In comparison to the earlier one this is a piece of cake:

```html
<script type="text/x-handlebars" data-template-name="posts/selectedPost">
  <div id="selectedPost">
    <h1>{{title}}</h1>

    <p>{{summary}}</p>

    <p>{{body}}</p>
  </div>
</script>
```
There is one new thing here: the `data-template-name` demonstrates the nesting of the view names. 

Remember that the detail section will render its output into the `{{outlet}}` directive of the calling template. In this case it is the "posts" template so will be placed in the second `<div>`.

Normally placed in `src/main/ember/app/views/templates/posts/selected_post.hbs`. 

### Conclusion

So there you have it. A working paginated master-detail list which you can use as the basis for your own projects. It's been a long journey to get here and I hope that I have provided sufficient detail for you to be able to tackle your own Ember projects with more confidence than when you started.

Let me know how you get on in the comments!