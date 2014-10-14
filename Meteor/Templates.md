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


