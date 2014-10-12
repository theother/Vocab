
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