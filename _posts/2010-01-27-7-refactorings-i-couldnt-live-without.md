---
author: Gary Rowe
title: 7 refactorings I couldn't live without
layout: post
tags:
  - Tips
  - Design
  - Refactoring
redirect_from: /agilestack/2010/01/27/7-refactorings-i-couldnt-live-without/
---

As a full time developer I spend a lot of time writing code. And I almost never get it right first time. I did once, a long time ago, and since then I’ve not had such luck so I’ve had to rely on a myriad of safety nets, such as unit tests and a keep it simple approach to design. Even then my code tends to grow somewhat organically leading to various code smells that a healthy wave of refactoring wafts away.

I thought I’d take this opportunity to share the ones I use the most often – in fact I couldn’t really live without them these days.

### 1) Rename and Change Method Signature

My personal favourites. Rename does just what it says on the tin. Initiate the refactoring, tell the IDE to apply it in all the right places and it just does it. When applied to fields in Java beans, the IDE will ask if sir would like those getters/setters renamed as well, and throughout the application, including JSPs. Yes, please, do that for me and save me a whole bunch of time.

Change Method Signature is even better because it includes Rename as a happy side effect. Want to change the order of those parameters and their types but don’t want to screw up all the myriad of calls into those methods? Why, yes, yes I do, Mr IDE. Just click in this box until you’re happy with the new order and let me do all the hard work. Don’t mind if I do (cracks open beer).

### 2) Extract Method

Especially useful when [exploring a new code base][2]. If you want to simplify a method that has clearly gotten out of hand, just select the area you feel should be a method in it’s own right and hit that refactor button. Suddenly all that complex code is whisked away and in it’s place is a simple method call. And [if your IDE is really spiffy][3] then you’ll find that all the other copy/pasted versions of that code have also been refactored into the method call. Marvellous.

### 3) Pull Up and Push Down

Have you suddenly discovered that a method or field is going to be duplicated in a bunch of subclasses? Use Pull Up to move the duplicated method or field into a super class. Done correctly, the IDE will automatically apply the refactoring to all subclasses with the duplication and substitute the super class reference in it’s place.

Push Down, as it’s name suggests, does the opposite. Your wonderful base class is now preventing subclasses from specialising as they should so you push the field or method down so that the subclasses can implement the method or not as they see fit.

### 4) Extract Base Class

Want to do a Pull Up but don’t have a super class in place? No problem, just use this and select the methods/fields that you want pulled up. Want that base class abstract? Not a problem, just tick the box, sir.

### 5) Introduce Parameter Object

If there’s one thing I really hate it’s a method with a long parameter list. And by long I mean 5 parameters maximum. Especially if they’re all named string1, string2, stringX and so on because a) it’s mental, 2) it’s confusing and iii) nobody will ever get the parameters in the correct order.

The solution to this debacle is, of course, to introduce a parameter object. A simple Value Object (read Java Bean) that abstracts all the parameters away behind a single reference. Want to add a new parameter but don’t want to have to go and change all the method signatures on your publicly available API? Just make sure they take a parameter object and the API will continue to work as you change the properties on the parameter object. If you were feeling really ambitious you could put validation code into the parameter object to provide a defensive layer, but that’s just silly, isn’t it? Isn’t it?

### 6) Introduce Constant

We’ve all done it: hard coded a string or number somewhere. Then used it again. And again. Then realised that, really, it should be defined properly somewhere up in the root of the class or ideally injected. And that is exactly what the Introduce Constant refactoring does. With a swift click of the mouse all the duplications are pulled up into a single, well named constant field destined to remain the single point of reference for that datum for all time. Very DRY.

### 7) Introduce Field

Method signatures getting a bit repetitive with that same reference getting passed around all over the place? It’s about time that parameter was converted into a fully fledged field.

The only problem with refactoring is that after you’ve done it a few times you kind of get addicted to it. Before long you’ll be spotting duplications and excessive parameter lists all over the place and applying refactorings to tidy them all up and what are you left with? Just well designed, clean code that is much easier to maintain.

 [1]: https://twitter.com/share
 [2]: http://gary-rowe.com/agilestack/2010/01/14/9-top-tips-to-understand-new-code/
 [3]: http://www.jetbrains.com/idea/features/index.html