---
layout:     post
title:      "Understand javascript clousres."
subtitle:   "It always seems impossible until it's done."
date:       2016-03-18 20:25:00
author:     "Nickolas"
header-img: "img/hot-air-balloons-1253229.jpg"
---

> A closure is the local variables for a function — kept alive after the function has returned, or a closure is a stack-frame which is not deallocated when the function returns (as if a 'stack-frame' were malloc'ed instead of being on the stack!).

## An example of Closure

``` 
function sayHello2(name) {
    var text = 'Hello ' + name; // Local variable
    var say = function() { console.log(text); }
    return say;
}
var say2 = sayHello2('Bob');
say2(); // logs "Hello Bob"
```

The above code has a closure because the anonymous `function function() { console.log(text); }` is declared inside another function, `sayHello2()` in this example. In JavaScript, if you use the function keyword inside another function, you are creating a closure.

In JavaScript, if you declare a function within another function, then the local variables can remain accessible after returning from the function you called.

## Another example 

This example shows that the local variables are not copied — they are kept by reference. It is kind of like keeping a stack-frame in memory when the outer function exits!

```
function say667() {
    // Local variable that ends up within closure
    var num = 666;
    var say = function() { console.log(num); }
    num++;
    return say;
}
var sayNumber = say667();
sayNumber(); // logs 667
``` 

## Example 3

All three global functions have a common reference to the same closure because they are all declared within a single call to `setupSomeGlobals()`.


```
var gLogNumber, gIncreaseNumber, gSetNumber;
function setupSomeGlobals() {
    // Local variable that ends up within closure
    var num = 666;
    // Store some references to functions as global variables
    gLogNumber = function() { console.log(num); }
    gIncreaseNumber = function() { num++; }
    gSetNumber = function(x) { num = x; }
}

setupSomeGlobals();
gIncreaseNumber();
gLogNumber(); // 667
gSetNumber(5);
gLogNumber(); // 5

var oldLog = gLogNumber;

setupSomeGlobals();
gLogNumber(); // 666

oldLog() // 5
```

Note that in the above example, if you call `setupSomeGlobals()` again, then a new closure (stack-frame!) is created. The old gAlertNumber, gIncreaseNumber, gSetNumber variables are overwritten with new functions that have the new closure. (**In JavaScript, whenever you declare a function inside another function, the inside function(s) is/are recreated again each time the outside function is called.**)

## Example 4

This one is a real gotcha for many people, so you need to understand it. Be very careful if you are defining a function within a loop: the local variables from the closure do not act as you might first think.

```
function buildList(list) {
    var result = [];
    for (var i = 0; i < list.length; i++) {
        var item = 'item' + i;
        result.push( function() {console.log(item + ' ' + list[i])} );
    }
    return result;
}

function testList() {
    var fnlist = buildList([1,2,3]);
    // Using j only to help prevent confusion -- could use i.
    for (var j = 0; j < fnlist.length; j++) {
        fnlist[j]();
    }
}
```

Note that when you run the example, "item2 undefined" is alerted three times! This is because just like previous examples, there is only one closure for the local variables for buildList. When the anonymous functions are called on the line `fnlist[j]();` they all use the same single closure, and they use the current value for i and item within that one closure (where i has a value of 3 because the loop had completed, and item has a value of 'item2'). Note we are indexing from 0 hence item has a value of item2. And the i++ will increment i to the value 3.

## Example 5

This example shows that the closure contains any local variables that were declared inside the outer function before it exited. Note that the variable `alice` is actually declared after the anonymous function. The anonymous function is declared first; and when that function is called it can access the alice variable because alice is in the same scope (JavaScript does variable hoisting). Also `sayAlice()()` just directly calls the function reference returned from `sayAlice()` — it is exactly the same as what was done previously, but without the temporary variable.

```
function sayAlice() {
    var say = function() { console.log(alice); }
    // Local variable that ends up within closure
    var alice = 'Hello Alice';
    return say;
}
sayAlice()();
```

## Final points
* Whenever you use function inside another function, a closure is used.
* Whenever you use `eval()` inside a function, a closure is used. The text you `eval` can reference local variables of the function, and within eval you can even create new local variables by using eval('var foo = …')

* When you use new Function(…) (the Function constructor) inside a function, it does not create a closure. (The new function cannot reference the local variables of the outer function.)
* A closure in JavaScript is like keeping a copy of all the local variables, just as they were when a function exited.
* It is probably best to think that a closure is always created just on entry to a function, and the local variables are added to that closure.
* A new set of local variables is kept every time a function with a closure is called (given that the function contains a function declaration inside it, and a reference to that inside function is either returned or an external reference is kept for it in some way).
* It is possible to get function declarations within function declarations within functions — and you can get closures at more than one level.
* I think normally a closure is the term for both the function along with the variables that are captured. Note that I do not use that definition in this article!


Notes:  
The original article is from [stackoverflow.com](http://stackoverflow.com/questions/111102/how-do-javascript-closures-work), written by Morris and community (cc-wiki-license). I have taken some good parts. I will add more information about how clousres works later. 

The picture is from pixabay.com with CC0.












