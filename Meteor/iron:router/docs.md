
Main Concepts
=============
Server Only
----------
Old school http request. You navigate to a page and a request is sent to the server and that server sends some info back.

Client Only
-----------
Client side routes used by pageJs and Backbone, and let the client navigate throught the app without needing to refresh their shit via HTML5 fetrures like pushState of url hash fragments.

Client and Server
-------------
Iron:Router runs on both the client and the server, which gives the app some serious speed once it is loaded. Additionaly, the router is aware of all the routes on the client and the server. Which means if there is no client route defined you can send a 404 which is helps with indexing

Reativity
--------
A large marjority of routes and hooks run reactive computations. For example if you call `Meteor.user()` inside a route function, your function will rerun each time the calie of `Meteor.user()` changes.


Route Parameter
=============
Routes can have variable parameters, and if you want to specify a var param you use the `:` syntax in the url followed by the parmeter name. When a user goes to that url the actual value of the parameter will be stored as a property on `this.params`.
```javascript
// Givin a url like /post/5
Router.route('/post/:_id', function(){
    var params = this.params; // {_id: '5'}
    var id = params._id; // 5
});
```
You can also have multip route parameters!
```javascript
// given a url like '/post/5/comments/100'
Router.route('/post/:_id/comments/:commentId', function (){
    var id = this.params.id; // '5'
    var commentId = this.params.commentId; // '100'
});
```
You can also accsess `query` and `hash` properties
```javascript
// given a url like: '/post/5?q=s#hashFrag'
Router.route('/post/:_id', function(){
    var id = this.params._id; // '5'
    var query = this.params.query; //query.q -> 's'
    var hash = this.params.hash; // 'hashFrag'
})
```

Rendering Templates
===================
To render the teplate named `Post` when a user navigates to the url `/pots/1` you call the `render` method inside of your route function which takes the name of the tempalate as its first parameter.
```html
<tempalate name="Post">
    <h1>Post: {{title}}</h1>
    {{#each Post}}
        <li>{{author}}</li>
    {{/each}}
</tempalate>
```
```javascript
Router.route('/post/:_id', function(){
    this.render('Post');
})
```

In the above example `{{title}}` is not defined. You could make create a template helper, although the cleaner way to do this is to set the `data` context on the `render` call as a second paramater
```javascript
Router.route('/post/:_id', function(){
    this.render('Post', {
        data: function(){
            return Post.findOne({_id: this.params._id})
        }
    });
});

//To set the data context for {{each}} the data function must return a context
Router.route('/post/:_id', function (){
    this.render('Post', {
        data: function (){
            return {
                Post: function () {
                    return Post.find();
                }
            }
        }
    });
});
```

Layouts
=======
Layouts at the end of the day are just templates. Although, you can use the special `{{yield}}` helper. Think of `{{yield}}` as a placeholder for content. The placeholder is called a `region`. And the content will be `injected` into said region when the route is ran. This allowing you to reuse layouts on many diffrent pages, and only change the content of the `yield` regions
```html
<template name="ApplicationLayout">
    <header>
        <h1>{{title}}</h1>
    </header>

    <aside>
        {{> yield "aside"}}
    </aside>

    <article>
        {{> yield}}
    </article>

    <footer>
        {{> yield "footer"}}
    </footer>
</template>
```
You tell the route function which layout to use by calling the `layout` method
```javascript
Router.route('/post/:_id', function () {
    this.layout('ApplictionLayout')
)};
```
Furthermore, if you want to use a defualt layout template for all the routes you can specify one as such via `configure` method on the Router
```javascript
Router.configure({
    layoutTemplate: 'ApplicationLayout'
});
```

Rendering Templates into Regions
================================
In the route function we can specify which templates to render in each region
```html
<template name="post">
    <p>
        {{post_content}}
    </p>
</template>
<template name="PostFooter">
    Some post specific footer content
</template>
<template name="PostAside">
    Some post specific aside content
</template>
```
If you then want to inject these templates into the `ApplicationLayout` template you do so using the `to` option of the `render` method
```javascript
Router.route('/post/:_id', function(){
    this.layout('ApplicationLayout');
    //render the Post template into the "main" region via {{> yeild}}
    this.render('Post');
    //Render the PostAside template into the yeild region named "aside"
    // {{>yield "aside"}}
    this.render('PostAside', {to: 'aside'})
    //Same concept with the footer
    this.render('PostFooter', {to: 'footer'})
    })
```
You can also set the data context for regions via the `data` option to the `render` method.
```javascript
Router.route('/post/:_id', function(){
    this.layout('ApplicationLayout', {
        data: function () {return Posts.findOne({_id: this.params._id})}
    });
    this.render('Post',  {
    // we don't really need this since we set the data context for the
    // the entire layout above. But this demonstrates how you can set
    // a new data context for each specific region.
        data: function () { return Posts.findOne({_id: this.params._id})
  });
    this.render('PostAside', {
        to: 'aside',
        data: function () { return Posts.findOne({_id: this.params._id})
    });
})
```
Rendering templates into regions from our route function is useful if you are running some logic. Although, if you got no _logic_ then you can use the `contentFor` helper.
```html
<template name="Post">
    <p>{{Post_Content}}</p>
    {{#contentFor "aside"}}
        Some content for the aside
    {{/contentFor}}
    {{#contentFor "footer"}}
        Footer contnet
    {{/contentFor}}
</template>

```
Or you can use the exsiting template via
```html
<template name="Post">
    <p>{{Post_Content}}</p>
    {{> contentFor region="aside" template="PostAside"}}
    {{> contentFor region="footer" template="PostFooter"}}
</template>
```
Router.go()
==========
You use `Router.go()` to navigate to a given url or even a named route
```html
<template name="aFuckingButton">
    <button id="clickMe">Click this Button to Go to Page one</button>
</template>
```
```javascript
Template.aFuckingButton.events({
    'click #clickMe': function(){
        Router.go('/one')
    }
})
```
Using Links to Server Routes
============================
You got a file to offer up for downloads, irons got you covered.
```html
<a href="/donwload/myFileName">Download a Virus</a>
```
```javascript
Router.route('/download/:filname', function(){
    this.responce.end('some file content\n')
}, {where: 'server'})
```
Named Routes
===========
ROUtes can have names that can be used to refer to the route. No name given it iron will guess its name based on your path.
```javascript
Router.route('/posts/:_id', function(){
    this.render('Post');
}, {
    name: 'post.show'
});
```
Now we have access to the route object
```javascript
Router.routes['post.show']
```
```javascript
Router.go('post.show')
```
Path and Link Template Helpers
=============================
__pathFor__
Generates a dynamic route based on its shit.
```html
{{#with post}}
    <a href="{{pathFor route='post.show' data=getPost hash='frag'}}">Post Show</a>
{{/with}}
```
__linkTo__
The `linkTo` helper automatically generates the html for an anchor tag along with the route path for the given route
```html
{{#linkTo route="post.show"}}
    Post Show
{{/linkTO}}
```
Route Specific Options
======================
```javascript
Router.route('/post/:_id', {
    name: 'post.show',
    //Specific RouteController
    controller: 'PostRouteController',
    //Diffrent then the global template
    template: 'Not_The_Main_Layout',
    //Declarative way of providing templates for each yeild region
    yieldRegions: {
        'MyAside': {to: 'aside'},
        'MyFooter': {to: 'footer'}
    },
    //Subscriptions or other things we want to wait for
    waitOn: function () {
        return Meteor.subscribe('post', this.params._id);
    },
    //Set the data context for the layout. This function works with plugs/hooks
    //Ex. The 'dataNotFound' plugin calls this function to see if it returns
    //a null value, and if so, renders the not found template
    data: function () {
        return Posts.findOne({_id: this.params._id});
    },
    //Hook options
    onRun: function() {},
    onRerun: function() {},
    onBeforeAction: function() {},
    onAfterAction: function() {},
    onStop: function() {},

    action: function () {
        this.render()
    }
})
```



Hooks
=====
__Types__
+ `onRun`
    * Runs the first time your controller gets called and only runs once
    * Like mix-panale metric tracking - things you don't want to rerun
+ `onRerun`
    * Called only if function is rerun due to a reactive data source change
+ `onBeforeAction`
    * Gets called before the action function gets called. These hooks behave specially. IF you want to continue calling the next function you must call `this.next()`. If you don't, downstream onBeforeAction hooks and your action function will not be called
+ `onAfterAction`
    * Gets called after the action function gets called. These hooks behave normally, and you don't need to call `this.next()`
+ `onStop`
    * Gets called right before we navigate away from this route

> With anly of these hooks they can contian an array of function that you wish to call


The following example renders the template and regions for a route only if the user is logged in.
```javascript
Router.onBeforeAction(function() {
    // All properties are available in the route function
    // are also avialable here such as this.params
    if(Meteor.logginIn()) { //If the user is loggin in, just wait
        //Just wait
        return;
    }else if(!Meteor.user) { //If user is not loged in
        // Redirct them to the login page
        this.redirect('Login')
    }else{
        // otherwise run the rest of the hooks
        this.next()
    }
});
```
Thus if you have a route like so
```javascript
Router.route('/users', function (){
    this.render('Userpage')
});
```
if the user is not logged in the route function will never get called and the `Userpage` will never be rendered. Addionaly, hooks will rerun if they have a reactive data source like `Meteor.user()` and if that varibale changes the whole function will rerun.

__Apply Hooks Specificaly__
You can pass `except` or `only` option to each respective hook
```javascript
Router.onBeforeAction(myUserHookFunction, {
    only: ['user']
    // or exept: ['routeOne', 'routeTwo']
})
```
The `myUserHookFunction` will only get applied to the route named `user`


__Global Hooks__
You can make gloabal hooks like a global login hook using the specific hooks as such.
```javascript
Router.onBeforeAction(function() {
    if(Meteor.logginIn()) {
        return;
    }else if(!Meteor.user) {
        this.redirect('Login')
    }else{
        this.next()
    }
}, {only: ['user.area']}); // will only run if user if clicks that route
```

__Plugins__
Two params are passed to the `router instance` and a set of `options`
```javascript
Iron.Router.plugins.authorize = function(specificRouter, specificOptions){
    specificRouter.onBeforeAction(function() {
    if(Meteor.logginIn()) {
        return;
    }else if(!Meteor.user) {
        this.redirect('Login')
    }else{
        this.next()
    }
}, options);
}

//To use you apps the plugin and your options

//I want to use the `authorize` plug in only to `article.new`
Router.plugin('authorize', {only: ['article.new']});
```
Furthermore, you can specify options within your route as such
```javascript
Iron.Router.plugins.authorize = function(specificRouter, specificOptions){
    specificRouter.onBeforeAction(function() {
    if(Meteor.logginIn()) {
        return;
    }else if(!Meteor.user) {
        //This is our new option, that we then would pass a var to
        this.redirect(this.lookupOptions('notAuthorizedRoute'))
    }else{
        this.next()
    }
}, options);
}

//Passing in new option
Router.plugin('authorize', {
    only: ['article.new'],
    notAuthorizedRoute: 'home'
});
```


Server Router
============
__Creating Routes__
You can create server routes with full access to the NodeJs request and response objects. To create a server route you provide the `where: 'server'` option to the route.
```javascript
Router.route('/download/:file', function () {
    // Nodejs Request object
    var request = this.request;

    // Nodejs response
    var response = this.response;

    this.response.end('file downloaded')
}, {where: 'server'})
```

__Restful Routes__
You can create server-side restful routes which correspond to an http verb. This is useful if you are setting up a webhook for another server to post data to.
```javascript
Router.route('/webhooks/stripe', { where: 'server' })
  .get(function () {
    // GET /webhooks/stripe
  })
  .post(function () {
    // POST /webhooks/stripe
  })
  .put(function () {
    // PUT /webhooks/stripe
})
```

__404s and Client vs Server Routes__
When you initially navigate to your Meteor application's url, the server router will see if there are any routes defined for that url, either on the server or on the client. If no routes are found, the server will send a 404 http status code to indicate no resource was found for the given url.

__Server Middleware and Connect__
YOu can attach middleware to the router on the server using the `use` method of the router. And Connect middleware just works out-of-the-box. This is becuase the `req, res, next` arguments are passed to the router handler functions like just in the Connect middleware statck. But typically we will access those properties using `this.request`, `this.response`, and `this.next` instead.
```javascript
if (Meteor.isServer) {
  // assuming we've loaded a package with access to connect
  var connect = Npm.require('connect');
  Router.use(connect.queryParser(), {where: 'server'});
}

//You could also create your own server-side middlewate, for example, you might want to log all http requests

Router.use(function logHttpRequests () {
  var method = this.method;
  var url = this.url;
  console.log(method + ' ' + url);

  // go on to the next handler now
  this.next();
}, {where: 'server'});
```