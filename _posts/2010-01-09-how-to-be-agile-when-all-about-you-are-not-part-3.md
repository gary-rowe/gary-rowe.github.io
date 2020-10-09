---
author: Gary Rowe
title: How to be agile when all about you are not - part 3
layout: post
tags:
  - Agile
  - Refactoring
  - Testing
redirect_from: /agilestack/2010/01/09/how-to-be-agile-when-all-about-you-are-not-part-3/
---

# How to be agile when all about you are not – part 3

[In part 2 of this series][2], I explained how the DRY principle can be used in the build process and how the need for automated testing naturally arises from this. In this article, I go on to explain how to implement automated testing in Java using Ant and JUnit.

The subject of automated testing is vast, with many different philosophies and naming conventions springing up around it and a full explanation would take ages. [Include links to other sites]. However, it is well proven that the introduction of automated testing greatly improves the quality of a code base because of what is necessary to support it. There are 3 pre-requisites to successful automated testing:

1) an automatic build system  
2) some kind of automated testing framework  
3) an open design allowing small parts of the system to be tested in isolation

For the vast majority of projects, some kind of automated build system is already in place within the IDE. In the case of Java this is [Ant][3], and typically the build script is automatically generated as a result of the various options that are selected within the IDE for the project.

Although it’s obvious, it still needs to be said that test code should not be mixed with production code, and that includes the supporting libraries. There are several ways to avoid this, but the best way is to introduce a new source tree that only contains test related classes and resources.

If you are a Java developer, the [Maven][4] convention is to define a sub-directory under the main source tree as follows: src/test/java, with production code going under src/main/java. I would strongly urge you to follow this convention as [Maven][4] is growing steadily as a replacement for Ant. Test classes then have the same package as the class under test.

The most widely used automated test framework in the Java world is [JUnit][5] (NUnit is used in the .Net world). At the time of writing version 4 is well-established and I would strongly urge you to use it. All that is required is the inclusion of a simple dependency into your project and you’re done. If you want to find out more about [JUnit][5] here is a good starting point.

The final step is modifying the existing code to support automated testing. This can be easy or hard depending on how well designed the legacy code is. If good programming practices have been followed, then automated testing is straightforward. There will be strong separation between classes, with each class performing a few well defined tasks that contribute to an overall cluster of functionality. Navigation around the cluster of objects will be straightforward since they will have clear naming conventions and follow some kind of established design pattern.

**Time to refactor**  
For the rest of us, it’s time for some hard graft in the form of refactoring. This is the process by which code is rewritten to conform to a higher standard of design principles without changing it’s functionality. If done correctly the outside world is unaware of any change being made, while internally the code is much cleaner, succinct and therefore easier to maintain.

There are many references on the subject of refactoring (here is a good starting point), and your IDE will definitely support a wide variety of refactoring operations. Essentially your goal in refactoring is to break the code apart into smaller testable units that can be instantiated without reference to any of their surrounding classes. Or, if a reference must exist for a unit to work, that the reference is supplied from outside the class, i.e. the test class. This is known as [Dependency Injection or Inversion of Control (IOC)][6]. A quick way to identify if you’re using IOC is to see if any of the dependency objects are created with the new operator, or are static references to utility classes. Either of these will mean some refactoring to introduce getters and setters so that external classes can pass in the instances for these dependencies.

So, instead of

```java
private Worker worker = new Worker();
```

you have

```java
private Worker worker = null;

public void setWorker(Worker worker) {
  this.worker=worker;
}
```

and now the test class can provide an alternative implementation of the Worker class which behaves in a predictable manner suitable for testing. The above can be achieved by means of the [Encapsulate Fields refactoring][7] (which is really for converting public fields into private ones with a setter, but never mind you can see what I’m trying to get across here).

The best approach to refactoring is to target a single change at a time. Your goal is to isolate each class as much as possible with the IOC pattern through a series of field conversions – pushing the external dependencies to the outer class that control the one you’re working with. At this point, the class is ready for automated testing.

Let’s get back to our fictional developer, Bob, and assume that he’s made these refactorings to his code to introduce the new credit card support ([see part 2 for a fuller description of this][2]). Instead of the inefficient and laborious process that he previously had to go through, the new approach is something like this:  
1) Find the credit card payment system in the code and it’s associated automated tests  
2) Modify the tests to include the new credit card and a wide range of test data that should induce failures  
3) Run the tests and note any failures (the first time will definitely fail since there is no implementation code)  
4) Fix the failures and repeat from step 3) until it’s all good  
5) Perform an update from version control and re-run the tests  
6) If it’s all OK, check it in

As you can see the cycle time between making a test and seeing the result of it is greatly reduced to a matter of seconds rather than minutes. Also, once the automated test has been written then every time the system is built then it will perform the same checks, over and over again forever. So now, if someone makes a change to that part of the system guarded by the test which causes the test to fail then there is a clear path to what went wrong and how to to fix it. The failure is caught early on in the development cycle and that is the cheapest point to fix it. Also, if the developer introducing the failure is regularly running a build that includes the test then they will remember what change they made that caused the failure in the first place and be quicker to debug and fix it.

In part 4 of this series I will explain what happens next to further improve the build process by introducing the idea of continuous integration.

 [1]: https://twitter.com/share
 [2]: http://gary-rowe.com/agilestack/2010/01/09/how-to-be-agile-when-all-about-you-are-not-part-2/
 [3]: http://ant.apache.org/
 [4]: http://maven.apache.org/
 [5]: http://www.junit.org/
 [6]: http://en.wikipedia.org/wiki/Inversion_of_control
 [7]: http://www.refactoring.com/catalog/encapsulateField.html