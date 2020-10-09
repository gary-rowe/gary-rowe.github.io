---
author: Gary Rowe
title: How to be agile when all about you are not - part 5
layout: post
tags:
  - Agile
  - Maven
  - Tutorial
redirect_from: /agilestack/2010/01/14/how-to-be-agile-when-all-about-you-are-not-part-5/
---

# How to be agile when all about you are not – part 5

[  
In part 4 of this series][2], I explained how continuous integration is used to automate the build process. In this article, I shall go on to explain how to further apply the DRY principle with fully automatic standardised build processes that generate code for you.

One of the biggest problems with build scripts is that they are tailored to a specific system and cannot be reused. Furthermore, every developer (and IDE) has their own ideas about how a project should be laid out; where compiled code should be placed and where the dependencies are stored. In some cases there are very good reasons why code is where it is, but more often than not it is because of some kind of workaround to overcome a limitation of the build process.

For a long time, [Ant][3] was the king of the Java build process. It is present in just about every mainstream IDE that supports Java and is a solid and reliable workhorse in the development process. However, Ant has many limitations (dependency management and script standardisation being two of the main ones) which is why the developers of Ant moved on to create [Maven][4].

The idea behind Maven is to provide a unified build process that can be applied to just about any project of any size in an intuitive manner. It favours certain conventions that if followed makes your project immediately accessible to anyone else who knows Maven. Learning curves drop considerably for new developers joining the team since they will already know where to find code and how the build process works. Maven is extensible via plugins so that esoteric build requirements can quickly be catered for, and a [large number of standard plugins are available for free][5]. These include plugins for creating EAR files, automatically generating code, linking back to Ant scripts and just about every other purpose you could think of.

So how is this in line with the DRY principle? By providing a single point of reference for all project build information that is defined in a consistent format: XML. When I say all project build information, I mean it. Apart from the obvious “project name” and “version” there is a plethora of other information: who the developers are along with contact information; how to run automated tests and where to publish the results; how to generate the javadocs and where to publish them; how to manage versioned releases; how to generate the overall website associated with the project and where to publish it; and, most importantly of all, how to manage the project dependencies.

And in that closing entry lies the distillation of the DRY principle: a single point of reference for all the dependencies of the system, along with their versions so that they do not need to be stored within the project any longer. For many developers this comes as a great relief because there is no need to work out which version of, say, cglib.jar is copmatible with both the Spring 2.5.5 and Hibernate 3.3 frameworks. Instead, all Maven needs to know is what version of Spring you’d like, and which version of Hibernate and it works out the rest.

[Maven][4] also provides a very simple but powerful mechanism for avoiding repetition with the project file itself. For example, if the project has an overall dependency on Spring this is actually implemented as a small collection of dependency entries containing the precise modules of [Spring][6] that are needed (say core, transactions, web and MVC). If done without thought then there will be 4 places where the version, say 2.5.5, is referenced. However, the designers of Maven have provided a properties mechanism so that this repetition is confined to the name of a property that contains the appropriate version. Thus, to change from 2.5.5 to 2.5.6 all that is required is a simple change in the value of the property and all places where that property is referenced will be automatically updated. In a large build with many modules this saves an enormous amount of work and creates a very orderly build system.

At this point you have a very flexible and easy to maintain build system and a comprehensive set of automated tests that is providing protection against brittle code. So what next? Plenty. You have an agile build process, but you probably don’t have an Agile approach to design and implementation. For that you need to follow the trail “How to be an agile project manager” (coming soon).

 [1]: https://twitter.com/share
 [2]: http://gary-rowe.com/agilestack/2010/01/09/how-to-be-agile-when-all-about-you-are-not-part-4/
 [3]: http://ant.apache.org/
 [4]: http://maven.apache.org/
 [5]: http://maven.apache.org/plugins/
 [6]: http://www.springsource.org/about