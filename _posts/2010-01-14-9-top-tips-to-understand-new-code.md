---
author: Gary Rowe
title: 9 top tips to understand new code
layout: post
tags:
  - Management
  - Testing
  - Tips
redirect_from: /agilestack/2010/01/14/9-top-tips-to-understand-new-code/
---

# 9 top tips to understand new code

You’ve just started at a new company, or have been introduced to a new system,  and you need to get up to speed quickly. Here are some proven tips and techniques to get you started.

### 1) Show no fear

Most developers are nervous when they first encounter a new system, but just keep in mind that *it’s only code*. There is a structure involved, however convoluted and apparently incomprehensible it may appear at the start. Once you start exploring this new realm of code you’ll be able to gain greater understanding, and if you’re the only one who knows it then your position within the company becomes much more valuable.

### 2) Identify the principal technologies involved

Have a quick scan over a few modules in the system. If you recognise various patterns that indicate classic design approaches (e.g. Spring, EJB, OSGI, Hibernate) then you’re in luck – you can make rapid progress. If not, then don’t despair. It just means you have to do some reading. And if you don’t do the background reading then you’ll be floundering in no time trying to wrap your head around a seemingly incomprehensible approach to coding. However, investing that time in yourself will mean that you’ll recognise patterns and will be able to start making progress.

### 3) Leave breadcrumbs for yourself

As you step into the code and start exploring the dependencies among the various objects it is important to build up a trail of breadcrumbs to help you find your way back. Just keep a list of class names with a short note of what they apparently do, along with some notes on possible future refactorings . Perhaps some simple diagrams here and there. They don’t have to be a full-on UML representation, just a quick scribble in your notebook is sufficient to understand data flow and immediate class dependencies.

### 4) Add comments

As you go through the code add comments so that you leave the system in better shape than you found it. The next developer to follow in your footsteps will definitely thank you for it – so long as the comments are accurate and to the point. Don’t comment the obvious: “Setting km to 111″ is redundant – any developer can see that. Instead, reserve comments for the less obvious: “Using an approximate value for kilometres in a degree of latitude – REFACTOR with method?”. This identifies the overall context of the value, and raises the possibility that a method could be employed to return differing values depending on the longitude.

### 5) Explore the database

Nearly all applications attempt to model their database in some manner, and the database is usually the cleanest part of a system. After all, the database is likely to be reasonably relational in nature and there are a lot of tools available to allow exploration of a schema visually. By exploring the database the principal entities involved in the system can be identified and this in turn provides some insight to starting points within the code. Of course, if the database is a mess, imagine how awful the code that relies on it is likely to be.

### 6) How does login work?

Almost every system has some kind of login process and since this does not vary an awful lot conceptually it provides a good starting point. In understanding the login process you will have a better idea of how data is taken from the user interface and the database; compared in some kind of business logic and how the result is output to the user interface.

### 7) Write a test

Once you feel confident navigating around the code base, the next step is to try to add some code. The safest way to do this is to write some code that will never get released to live, but serves a useful purpose: a unit test. Pick a class that has few dependencies and then write the unit test. Do not alter the class under test. Your objective here is not to rewrite the existing code, just to understand how to introduce the unit test framework, or use the existing one.

### 8) Implement a small change

Pick an innocent looking change from the task list. Ideally, this should be something straightforward like a user interface correction or a small extension to existing code. Then implement it in very small increments, with a unit test covering your efforts as you go. Do not make big sweeping code changes, or refactorings that affect a large number of dependencies. Keep it small. Keep it simple.

### 9) Break dependencies

Once you’ve got some confidence with the code and you understand (somewhat vaguely) how it all hangs together, then it’s time to identify and break dependencies. This means finding a way to break tightly coupled code into smaller fragments that provide better encapsulation. Usually this means following a dependency injection (or Inversion of Control) design pattern. There are several available refactorings that help with this: Encapsulate Field; Extract Method; Extract Interface; Extract Subclass; Extract Superclass and so on.

 [1]: https://twitter.com/share