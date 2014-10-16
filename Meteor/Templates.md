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
````

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
Template tags are used to generate dynamic content which are also called expressions. They are data dependent on some kind of source trhat returns a value or some logic.
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
    return `<em> name </em'
  }
})


