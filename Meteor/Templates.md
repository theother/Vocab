Templates
========

Intro
-----
Tempaltes are the builfing blokes for your UI and are basically sinppets of HTML that that contain placeholers which can be filled by contentn processed by the application logic.
+ They can be reused to ensure the same look and feel on all sties and all elements.
+ Seperation of concers: is the only representation of `data` and gets rendered as `HTML` to the `DOM`
+ Increase readablity - css/html/js

```html
{{#each person}}
  <h5>{{NAME}}</h5>
  <p>Phone: {{PHONE}}</p>
{{/each}}
````

```html
<ul>
   <li>
      <h5>Christina</h5>
      <p>Phone: 1234-567</p>
     </li>
     <li>
      <h5>Stephan</h5>
      <p>Phone: 666-999-321</p>
     </li>
     <li>
       <h5>Manuel</h5>
       <p>Phone: 987-654-321</p>
   </li>
</ul>
```

Working With Blaze
------------------
Blaze is what Meteor uses a its UI and it makes a lot of magic happen. It has two major compnets:
+ Runtime API
  * Rendering of the elements/DOM
  * Observes changes
  * Udates elements
+ Build Time Compiler
  * HTML -> HTMLJS

Runtime API: if your data source changes, it will be reflected/updated on the DOM. If a phone number changes in the BD the Runtime API will update what the user current view since the placeholder `{{}}` depends on the actual value stored in the DB

Build Time Compiler: Compiles to HTML -> HTMLJS

> A side note: Both components work seperately, so it is entirely possible to just write js that targets the API


Organizing Template Files
------------------------
There are four elements to take into consideration
1. The actual template stored in an `.html` file
2. The JS code in the `.js` file that runs inside the client contenxt and provides functionality to the templates
3. Your styles in the `.css` file
4. Static resorces like pictures and such in the `\public` folder

Meteor will find your code for you so you could put your code where ever the fuck you please, although , I recomend you use a folder orginzation solution.
```html
<template name>.js
<template name>.html
```

Creating dynamic HTML templates
-------------------------------
Meteor Blaze uses a a templating language called Spacebars which is very similar to Handlebars and Mustache. There are four major types of template tags
1. Double-braced tags `{{...}}`
2. Triple-brace tags `{{{...}}}`
3. Inclusion tags `{{> ... }}`
4. Block tages `{{#directive}} .. {{/directive}}`

__Double and Triple__
Template tags are used to generate dynamic content which are also called expressions. They are data dependent on some kind of source that returns a value or some logic.
```html
<template name="profile">
  <li>{{name}}</li>
  <li>{{address}}</li>
</template>
```
Shit you need to know:
1. Tempalte opening and closing tags are mandatory
2. Make sure the template name is unique from other template names
3. The attribute name `{{attribute_name}}` has to be unique as well

__Double__
These are use to insert strings into your HTML, and no matter what will render a string regardless of the data type.
```html
<template name="profile">
  {{name}}
</template>
```
```javascript
Template.profile.helper({
  name: function(){
    return "<em> Willy </em>"
  }
})
```
The following code will render the following:
```html
<em>Willy</em>
```

__Triple-Braced Tags__
This template tag renders its contents exactly as you pass it into the template tag. So if you took the above example the content would be rended with a emphasis.
>__Word to the wise__: If triple braces are used to display user generated content and the data has not been check/sanitized for malicious content you will be left open to Cross-Site-Scripting attacks.

Inclusion Tags (Partials)
-------------------------
Templates can be inported into other templates via inclusion tags and these subtemplates are also known as `partials`
```html
{{> anotherTemplate}}
```
This alows for a better break-up of information and orginization by representing only one specific item/template/task.

Dynamic templates
-----------------
It is possible to include dynamic templates based on the return value of a corresponing helper. Allowing you to reactively switch templates without having to maintain complex if/else structures inside of templates.
```html
<template name="dynamicPartials">
  <div class="left">
    {{> Template.dynamic template=templateNameLeft }}
  </div>
  <div class="right">
    {{> Template.dynamic template=templateNameRight }}
  </div>
</template>

<template name="templateOne">
  <h1>Template One</h1>
</template>
<template name="templateTwo">
  <h1>Template Two</h1>
</template>
```
```javascript
Template.dynamicPartials.helpers({
  templateNameLeft: function () {
    return "templateOne";
  },
  templateNameRight: function () {
    return "templateTwop";
  }
});

```
In the above example `templateOne` and `templateTwo` are injected in the left and right name template respectivley.

Block Tags
----------
Block tags change the behavior of the enclosed block of HTML.
```html
<template name="main">
  {{#name arguments}}
    <p>Content</p>
  {{/name}}
</template>
```
You can either define your own block tags or use Spacebars which is as follows.
+ `#if` - executes a content block if a condition is true or the else block if not
+ `#unless` - executes a block if a condition is false or the else block if not
+ `#with` - sets the data context of a block
+ `#each` - loops through mulitiple elements
```html
<div>
  {{#if image}}
    <img src="{{image}}" />
  {{else}}
    <p>Sorry, no image available.</p>
  {{/if}}
</div>
```
>There is no {{elseif}} rather you will have to use nested if else statments

In the bellow example `#if` will evaluate the expression. The content of the `skills` obj is passed to the `hasMoreSkills` function and evaluated. In this case it is true and the link will apear in the tempalte.
```html
<div>
  {{#if hasMoreSkills skills}}
    <a href="/skills">See more</a>
  {{#if}}
</div>
```
```javascript
Template.skills.helpers({
  skills: ['Meteor', 'Sailing', 'Cooking'],
  //if more then one skill return true
  hasMoreSkills: function (skills) {
    return skills && skills.length > 1;
  }
});
```


##### Each/With TAg
The `{{#each}}` tag will iterate over multiple vales in an array, cursors, or falsey values. And in this example `this` defines the `data-context` of the block. `{{#each}}` __must have__ a data context.
```html
<template name="eachJob">
  <ul>
    {{#each skills}}
      <li>{{this}}</li>
    {{/each}}
  </ul>
</template>
```
```javascript
Template.eachJob.helper({
  skills: function () {
    return ['Mow Grass', 'Burn Shit', 'Relax']
  }
});
```

__With__
`{{#with}}` allows you to define a `data-context` which is the association between a template and any data. Setting the data context with `{{#with}}` requires a single attribute that will become the data context for the following block.
```html
<template name="withBlock">
  <ul>
    {{#with profile}}
      <p>{{name}}</p>
      {{#each skills}}
        <li>{{this}}</li>
      {{/each}}
    {{/with}}
  </ul>
</template>
```
```javascript
Template.withBlock.helpers({
  profile: function () {
    var jim = {
      name: 'Jim',
      skills: ['Doing it biG', 'Chilling', 'drinking']
    };
    return jim;
  }
});
```


Local Template Helpers
---------------------
Local Tp Helpers is used to only extend one specific template and cannot be shared with other Tp's.

Gloabl Helpers
--------------
Like the name indicates this helper is global and can be used more than once. For example you want a helper that returns `true` if an `array` is larger then `x`. You would put this helper in a `globalHelper` file. And you would use the following helper syntax `Template.registerHelper`. The following example skills will be shows, and friends will false and use the else block
```html
<template name="globalHelper">
  {{#if gtGlobal skills 1}}
    <a href="/skills" title="">See more skills</a>
  {{/if}}
  {{#if gtGlobal friends 3}}
    <a href="/moreImg" title="">See More Friends</a>
    {{else}}
    <h2>No more friends to show</h2>
  {{/if}}
</template>
```
```javascript
Template.registerHelper('gtGlobal', function (array, n) {
  console.log(array); // ['test', 'one', 'two'] // ['Tim', 'Ry']
  console.log(n); // 1 // 3
  return array && array.length > n;
});

Template.globalHelper.helpers({
  skills: function () {
    return ['test', 'one', 'two'];
  },
  friends: function  () {
    return ['Tim', 'Ry'];
  }
});

```

Custome Block Helpers
--------------------
Custome blocks are also globally avialible and can be very useful. They allow you to build resuable UI components or widgets. Additionaly, they can be used without js. In the example bellow, the template `customHelperBlock` is our defined helper block which the `someValue` template or anyother template template can use. In this case the custome helper block wraps it content div with the `box custom-helper-block` class
```html
<body>
  {{> someValue}}
</body>

<template name="someValue">
  {{#customHelperBlock}}
    Surrounded by a box.
  {{/customHelperBlock}}
</template>

<template name="customHelperBlock">
  <div class="box custom-helper-block">
    {{> Template.contentBlock}}
  </div>
</template>
```
Here is a silly font size example
```html
<template name="content">
  {{#fontSize "big"}}
    Big Time h1 babay
  {{/fontSize}}

  {{#fontSize "medium"}}
    Kicking around with h3
  {{/fontSize}}

  {{#fontSize "small"}}
    Poor little small me
  {{/fontSize}}
</template>

<template name="bigFont">
  <h1>
    {{> Template.contentBlock}}
  </h1>
</template>
<template name="mediumFont">
  <h3>
    {{> Template.contentBlock}}
  </h3>
</template>
<template name="smallFont">
  <p>
    {{> Template.contentBlock}}
  </p>
</template>

```
```javascript
Template.registerHelper('fontSize', function () {
  var size = this.valueOf();
  console.log(size); // big, medium, small
  if (size==='big') {
    return Template.bigFont;
  }
  else if (size==='medium'){
    return Template.mediumFont;
  }else{
    return Template.smallFont;
  }
});

```
You can also you `Template.contentBlock`'s opposite brother whom is know as `Template.elseBlock` which will import the `else` content
```html
<template name="genderTemplate">
  {{#ifFemale gender}}
    MRS.
  {{else}}
    MR.
  {{/ifFemale}}
</template>
<template name="isFemale">
  {{#if eq this 'f'}}
      {{> Template.contentBlock}}
    {{else}}
      {{> Template.elseBlock}}
  {{/if}}
</template>
```
```javascript
Template.genderTemplate.helpers({
  gender: function () {
    return 'm' // you would use some data source
  }
});

Template.isFemale.helpers({
  eq: function (a, b) {
    return a === b;
  }
})
```
__However,__ using the above syntax you are just asking for some shit and a headache. You want to keep your logic out of the html cuz this is not angular. So the following example is a better way to do said logic.
```html
<template name="genderTmpl">
  {{findGender gender}}
</template>
```
```javascript
Template.genderTmpl.helpers({
  gender: 'm',
  findGender: function (gender) {
    if (gender==='m') {
      return "Mr.";
    }else{
      return "Mrs.";
    }
  }
});
```

Handling Events
--------------
Meteor uses event maps to both define events and their actions. DOM events are used in conjunction with css selctors to specify what elements to watch.
#### Event Propagation
Be careful when is comes to event propagation, since it will prbly bite you in the ass if you are not careful. Thus it I recommend you use `event.stopImmediatePropagation();`. In the example bellow, with out the propagation stop the screen will only turn red and not green.
```html
<template name="layout">
  <button>Turn red</button>
  {{> green }}
</template>
<template name="green">
  <button id="green">Turn green</button>
</template>
```
```javascript
Template.layout.events({
  'click button': function (event, template) {
    event.stopImmediatePropagation()
    $('body').css('background-color', 'red');
  }
});
Template.green.events({
  'click button': function(event, template) {
    event.stopImmediatePropagation()
   $('body').css('background-color', 'green');
  }
});
```
Here is a list of of event propagations you can use:
+ `stopImmediatePropagation()` - to your template’s event map to prevent events from bubbling up the DOM.
+ `evt.stopPropagation()` - This does not prevent other event handlers from being executed
+ `evt.isPropagationStopped()` - to check whether `stopPropagation()` was called somewhere in the event chain
>Not really worth saying but of course you have event.preventDefault();

Template Lifecycle
-----------------
__The Three Life Stages__
1. Created
  + tmpl instance accessible
  + tmpl is not visible
2. Rendered
  + tmpl instance accessible
  + tmpl is visible
3. Destroyed
  + tmpl instance is no longer accessible
  + tmpl is not visible

__Created__
You have the ablity to set a callback function to set the properties of a tmpl before it is rendered, and everything you specify in that call back is avalible throught the life span of the tmpl. You can even use them in your helpers and event handlers, but in order to do so you can only access a tmpl instance from within a helper or event handler using `Template.instance()`
>`this` is the tmpl instance

__Rendered__
This callback is typically used to initiate obj that are inside the DOM already. Typically jGay plugin shit. They require a rendered DOM element, so they are initiated in the `renjdered` callback.
```javascript
Template.formTmpl.rendered = function () {
  $('.dateinput').datepicker({
    //ect
  });
};
```
__Destroyed__
This is used to clean up anything that you set up during the lifetime of a templaet.
```html
<template name="profile">
  <p>{{placeholder}}</p>
  <button>Button</button>
</template>
```
```javascript
Template.profile.created = function () {
  this.lastCallback = 'created';
  console.log('profile.created', this);
};
Template.profile.rendered = function () {
  this.lastCallback = 'renderd';
  console.log('profile.renderd', this)
};
Template.profile.destroyed = function () {
  this.lastCallback = 'dystroyed';
  console.log('profile.dystroyed', this)
};


Template.profile.helpers({
  placeholder: function () {
    console.log('profile.placeholder', this);
    console.log('profile.tplInstance', Template.instance().lastCallback);
    return 'This is the {{placeholder}} hlper'
  }
});
Template.profile.events({
  'click button': function () {
    Template.instance().lastCallback = 'renderd clicked';
    console.log('profile.clicked', this);
    console.log('profile.clicked.tplInstance', Template);
  }
});

```