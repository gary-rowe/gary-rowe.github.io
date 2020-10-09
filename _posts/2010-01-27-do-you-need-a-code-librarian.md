---
author: Gary Rowe
title: Do you need a Code Librarian?
layout: post
tags:
  - Design
  - Management
redirect_from: /agilestack/2010/01/27/do-you-need-a-code-librarian/
---

Nope. Well that was easy. Except you probably do.

According to the [Don’t Repeat Yourself (DRY)][2] principle, all code and reference material should have a single point of reference that is both unambiguous and easy to find.

Most major development efforts will be spread over several large projects. As a natural result of this there is going to be some duplication between, at least, the fundamental utility and data access classes. One approach in combating this duplication is to create a project that could absorb a lot of the commonality found in several diverse applications into a single module for inclusion in each.

As most developers know, the [Apache Foundation][3] have provided [a vast array of common extensions][4] to the Java language – some of which have been included in later versions of the language itself because they were so useful. However, as a taster, there are file utilities, specialised collections, string manipulation methods, date handlers, logging frameworks all of which provide a standardised approach to solving commonly found problems. Rather than roll your own library, why not take the best of breed from Apache and ask new developers coming to the company if they have familiarity with these libraries? The learning curve drops away the more you depend on freely available standard software.

Even after refactoring the existing code to use the various Apache Commons libraries, the size of the common project will increase. It is important to keep it focused so that it does not swell to be an all-encompassing uber-module that all projects must depend on. As the packages contained within the distributable JAR get too numerous, split them out into sub-modules that are highly targeted. A JAR for utilities, another for DAO interfaces, another for web utilities. Different projects can then use these highly specific, and loosely coupled, collections of foundation classes to meet their exact needs without needing a load of extraneous dependencies to also be included.

If you are using a build process like [Maven][5] then the issue of dependency management becomes trivial. Developers can make updates to the common modules and release new versions of the JAR into the company repository (managed by [Artifactory][6] for example). Other developers are unaffected until they decide to update their project file to include this higher release version. This allows them to continue without the new features saving the upgrade to a later time that suits them.

As these modules begin to grow in size it becomes difficult to keep track of all that useful code that your developers have spent a lot of time creating. It is at  this point you should consider introducing the role of a Code Librarian. This role, ancillary to normal development duties, is responsible for the maintenance of code that is deemed to be useful in more than one project. They can also assist with refactoring the original design of the code so that it fits with a wider audience. Typically the code librarian would sit in on code reviews, or be called over if doubt has arisen. They can then point out where the developer has written code themselves that should have been brought in from a common library, or to find ways of incorporating new code into the library.

The creation of a well-maintained centrally-managed code repository drastically reduces the amount of time and effort it takes to get a new project off the ground and out into production. It also greatly reduces the amount of developer time spent debugging. If the supporting documentation, such as a Wiki, contains good examples and references to other similar classes with slightly different applications then developers will quickly find themselves avoiding the re-invention of the wheel. For an example of this, [take a look at the YUI framework documentation][7].

 [1]: https://twitter.com/share
 [2]: http://gary-rowe.com/agilestack/2010/01/09/how-to-be-agile-when-all-about-you-are-not-part-1/
 [3]: http://www.apache.org
 [4]: http://commons.apache.org/
 [5]: http://maven.apache.org/
 [6]: http://www.jfrog.org/features.php
 [7]: http://developer.yahoo.com/yui/