---
author: Gary Rowe
title: How to be agile when all about you are not - part 4
layout: post
tags:
  - Agile
  - Hudson
  - Testing
redirect_from: /agilestack/2010/01/09/how-to-be-agile-when-all-about-you-are-not-part-4/
---

# How to be agile when all about you are not – part 4

[In part 3 of this series][2], I explained how automated testing can be used in the build process to greatly reduce the time taken to verify that new code is correct. I also introduced the idea of refactoring to make code easier to test. In this article, I shall go on to explain how to further improve the build process with continuous integration.

The idea behind continuous integration is simple, and stems directly from automated testing. There should be an automated process that continuously checks out the latest code and attempts to build and test it, reporting any failures automatically to the person who last made a commit since they are most likely to have broken the build.

Although this sounds daunting, it’s actually trivial to set up. All that is required (and again I speak from a Java based perspective) is:  
1) the free and open source application [Hudson][3]  
2) the free and open source servlet container [Tomcat][4]  
3) the free and open source build tool [Ant][5]

You’ll doubtless have noticed a distinct bias on my behalf on free and open source software. If you’d like [more information on why this is a Good Thing ][6]then check out this article about it.

All of those applications are a snap to install and get running. Once [Hudson][3] is up and running (it’s packaged as a self-contained WAR file), it is merely a matter of creating a job and configuring it with the version control details and the location of the build script file. I would recommend, though, that the machine on which these applications are installed (I’ll call it the continuous integration server, or CI server for short) is fairly beefy since it will very quickly start to be used by everybody for everything. And therein lies a problem.

At this point it is difficult to introduce the Agile methodology by stealth, since you’re going to need a server somewhere and that means involving other people. Of course, you could just install said CI server on your own machine and let it run overnight which is perfectly acceptable up to a point. Out of the box, [Hudson][3] will perform a fairly basic set of operations: check out the code at a particular time (or interval); attempt to run the build script; note any test failures and report back with a green light if all is well.

However, Hudson has an extensible architecture that allows plugins to be created and [there is a rich tapestry of possible add-ons available from the supporting website][7]. There is a plugin for [the FindBugs application][8] which can identify code that is likely to cause problems by an examination of the byte code. Another keeps track of [code coverage ][9]so that weak tests can be readily identified. Still another identifies long term trends associated with the number of tests and number of failures.

The list is long and before long the CI server will become as vital to the build process as the version control system, and clearly it will need to be given the same level of support from the network administrators. All being well, if you adopt the nightly build approach based on your own machine, then it won’t be long before you are starting to gather very useful data about the build process. Managers love this kind of information so it should be a fairly painless transition to move [Hudson][3] from your development machine to a managed server.

So by now the gradual changes to the development process have started to show real results and others are starting to take notice. It seems that your code is getting created faster than everyone else’s and it almost always works first time. And you’re identifying weak spots in the main application that no-one has ever seen before. Not bad, but surely there is more to it.

In part 5 of this series I will explain the next major shift: automating everything.

 [1]: https://twitter.com/share
 [2]: http://gary-rowe.com/agilestack/2010/01/09/how-to-be-agile-when-all-about-you-are-not-part-3/
 [3]: http://hudson-ci.org/
 [4]: http://tomcat.apache.org/
 [5]: http://ant.apache.org/
 [6]: http://en.wikipedia.org/wiki/Free_and_open_source_software
 [7]: https://hudson.dev.java.net/servlets/ProjectDocumentList?folderID=5818&expandFolder=5818&folderID=0
 [8]: http://findbugs.sourceforge.net/
 [9]: http://wiki.hudson-ci.org/display/HUDSON/Cobertura+Plugin