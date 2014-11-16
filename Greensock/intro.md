

Terms
====
+ `to()`
    * Method will tween to your set properties to be tweened
+ `from()`
    * Method does the revers of `to()`
    * Animate from `x` to whatever `x` currently is
+ `fromTo()`
    * Method allows you to define start value and end
+ `set()`
    * Sets default

##### Controls
+ `pause()`
+ `resume()`
+ `reverse()`
+ `restart()`

##### Special Terms
+ `onComplete`
+ `ease` 
+ `overwrite`
+ `paused` 
+ `useFrames`
+ `immediateRender`
+ `onStart` 
+ `onUpdate` 
+ `onCompleteParams`
+ `onCompleteParams`
    * Array - An Array of parameters to pass the onComplete function. For example,TweenLite.to(element, 1, {left:"100px", onComplete:myFunction, onCompleteParams:[element, "param2"]}); To self-reference the tween instance itself in one of the parameters, use "{self}", like:onCompleteParams:["{self}", "param2"]

##### Edge
+ autoAlpha
    * the same thing as "opacity" except that when the value hits "0", the "visibility" property will be set to "hidden" in order to improve browser rendering performance and prevent clicks/interactivity on the target. When the value is anything other than 0, "visibility" will be set to "visible".
    * `TweenLite.to(element, 2, {autoAlpha:0});`
+ className
    * Change the name to `x`
        - `TweenLite.to(element, 2, {autoAlpha:0});`
    * Add class
        - `TweenLite.to(myElement, 1, {className:"+=class2"});`


##### TimeLine
+ `tl.seek('lable')`
    * WIll Only tween from that label
+ `tl.timeScale(3)`
    * Modify the time

Basic
=====
GSAP can tween any numeric property of any JS obj.

```javascript
var obj = {prop: 0};
TweenLite.to(obj, 2, {prop:100});
``` 
__Paramaters__
1. Traget
2. Duration
3. Properties to be tweened

##### To
```javascript
TweenLite.to(".myClass", 2, {boxShadow:"0px 0px 20px red", color:"#FC0"});
``` 
##### From
```javascript
//animate from 0 to whatever the scaleX/scaleY is now
TweenLite.from(photo, 1.5, {scaleX:0, scaleY:0});
``` 

##### fromTo
```javascript
//tweens from width 0 to 100 and height 0 to 200
TweenLite.fromTo(photo, 1.5, {width:0, height:0}, {width:100, height:200});
``` 

OBJ Oriented Approch
=====================
All tweens play immediately, but you can pause them initially by passing `paused:true` in the vars parameter or by calling `pause()` on the instance.
```javascript
var myTween = new TweenLite(photo, 1.5, {width:100, height:200});

//pause the tween initially
var myTween = TweenLite.to(photo, 1.5, {width:100, paused:true});

//then later, resume
myTween.resume();
``` 

Special Properties
==================
A special property is a reserved keyword that TweenLite recognizes and handles differently than it would a normal property. One example is `delay` which allows you to delay a tween's start time by a certain number of seconds.
```javascript
//waits 2 seconds before beginning ("delay" is a special property TweenLite recognizes)
TweenLite.to(photo, 1.5, {width:100, delay:2});
``` 
__Special Terms___
Most used `onCompleted` and `ease`
```javascript
//notice there's no "()" after the onComplete function because it's just a reference to the function itself (you don't want to call it yet)
TweenLite.to(photo, 1.5, {width:100, delay:0.5, onComplete:myFunction});
function myFunction() {
    console.log("tween finished");
}
``` 
__Easing__
Controls the ease properties
```javascript
TweenLite.to(photo, 1, {width:100, ease:Power2.easeOut});
TweenLite.to(photo, 1, {height:200, ease:Elastic.easeOut});
``` 

Plugins
=======
Think of plugins like special properties that are dynamically added to TweenLite, giving it extra abilities. This keeps the core engine small and efficient, yet allows for virtually unlimited capabilities to be added dynamically. 

> The ScrollToPlugin will intercept the "scrollTo" value, etc.:

```javascript
//ScrollToPlugin will intercept the "scrollTo" value (if it's loaded)...
TweenLite.to(window, 2, {scrollTo:{y:300}, ease:Bounce.easeOut});
``` 

Tweening CSS Properties
=====================
Things that can be tweened

+ width
+ height
+ margins
+ padding
+ top left, ect 
+ colors
+ backgroundPosition
+ opacity
+ fontSize
+ backgroundColor
+ position
+ borderStyle

###### 2d
+ transforms 
+ rotation
+ scaleX
+ scaleY
+ skewX
+ skewY
+ x
+ y
+ xPercent
+ yPercent
+ rotationX
+ rotationY

```javascript
//much simpler
TweenLite.to(element, 2, {rotation:30, scaleX:0.8});
//use "deg" or "rad"
TweenLite.to(element, 2, {rotation:"1.25rad", skewX:"30deg"});
``` 

##### 3d
+ rotationX
+ rotationY
+ rotationZ (identical to regular "rotation"), 
+ z
+ perspective
+ transformPerspective

(Some 3d examples)[http://greensock.com/css3/]


Controlling Tweens
=================
```javascript
//using the static to() method...
var tween = TweenLite.to(element, 1, {width:"50%"});

//or use the object-oriented syntax...
var tween = new TweenLite(element, 1, {width:"50%"});
``` 
Then you can use these methods
```javascript
//pause
tween.pause();

//resume (honors direction - reversed or not)
tween.resume();

//reverse (always goes back towards the beginning)
tween.reverse();

//jump to exactly 0.5 seconds into the tween
tween.seek(0.5);

//make the tween go half-speed
tween.timeScale(0.5);

//make the tween go double-speed
tween.timeScale(2);

//immediately kill the tween and make it eligible for garbage collection
tween.kill();
``` 
You can also kill all tweens of a particular element/target
```javascript
TweenLite.killTweensOf(myElement);
``` 

Sequencing
```javascript
var t1 = new TimelineMax();
t1.to('.box', 0.5, {width: 200})
  .to('.box', 0.5, {height: 200})
  .to('.box h1', 1, {fontSize: 10})

//add another sequenced tween (by default, tweens are added to the end of the timeline which makes sequencing simple)
tl.to(element, 1, {height:"300px", ease:Elastic.easeOut});

//offset the next tween by 0.75 seconds so there's a gap between the end of the previous tween and this new one
tl.to(element, 1, {opacity:0.5}, "+=0.75");

//overlap the next tween with the previous one by 0.5 seconds (notice the negative offset at the end)
tl.to(element, 1, {backgroundColor:"#FF0000"}, "-=0.5");

//animate 3 elements (e1, e2, and e3) to a rotation of 60 degrees, and stagger their start times by 0.2 seconds
tl.staggerTo([e1, e2, e3], 1, {rotation:60}, 0.2);

//then call myFunction()
tl.call(myFunction);

//now we can control the entire sequence with the standard methods like these:
tl.pause();
tl.resume();
tl.restart();
tl.reverse();
tl.play();

//jump to exactly 2.5 seconds into the animation
tl.seek(2.5);

//slow down playback to 10% of the normal speed
tl.timeScale(0.1);

//add a label named "myLabel" at exactly 3 seconds:
tl.add("myLabel", 3);

//add a tween that starts at "myLabel"
tl.add( TweenLite.to(element, 1, {scale:0.5}), "myLabel");

//jump to "myLabel" and play from there:
tl.play("myLabel");
``` 

