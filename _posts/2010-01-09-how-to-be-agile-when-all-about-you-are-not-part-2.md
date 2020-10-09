---
author: Gary Rowe
title: How to be agile when all about you are not - part 2
layout: post
tags:
  - Agile
  - Design
redirect_from: /agilestack/2010/01/09/how-to-be-agile-when-all-about-you-are-not-part-2/
---

# How to be agile when all about you are not – part 2

It’s best to start with something familiar and slowly modify it to become more Agile in nature. So let’s start with a typical medium scale project – an imaginary airline booking system – and a typical developer who is part of a team, who we’ll call Bob. He’s been given the task of introducing a new credit card into the overall payment acceptance system, and he’s keen to introduce an Agile approach.

Now, this airline booking system, like thousands of other systems all over the world has been built in a more or less ad hoc fashion. A wide range of developers, some highly skilled, others less so, have stamped their individual style on the code base leaving it a patchwork of different approaches. There is little or no documentation available outside of the code apart from reams of meaningless specification documents that have been more or less ignored since they were created. The code contains a fair amount of comments, and developers try to keep them in step with the code but it is very difficult to gain a big picture perspective on how it all fits together.

Does any of this sound familiar? I imagine that it does because the overwhelming majority of systems that I have encountered all follow that pattern. It is the classic legacy system. It is not legacy because it uses old technology, it is legacy because it is impenetrable, untestable in any meaningful way and extremely brittle meaning that any form of change is going to cause problems in random places.

So I’ll gloss over the details of how Bob actually implements his update, but suffice to say that he follows this process:  
1) Fire up a local copy of the system with a debugger attached  
2) Work through the user interface making a flight booking until he reaches the payment page  
3) Enter test data and note any problems  
4) Implement the fixes in the code and rebuild  
5) Repeat from step 1 until the code works as expected  
6) Check it into version control

Anyone who has any familiarity with Agile practices will look at the above list and rightly shudder, but it is all too common because it is the most obvious way of meeting the objective.

How could Bob apply the DRY principle to the above?

The first step is to isolate any repetitive manual tasks that are involved. In this case it’s the endless building and stepping through the application that is causing all the time and enjoyment to be sucked away. Fortunately, there is a well known solution to this problem: automated testing.

[In part 3][2] I will demonstrate how to organise your project and build system to support automated testing.

 [1]: https://twitter.com/share
 [2]: http://gary-rowe.com/agilestack/2010/01/09/how-to-be-agile-when-all-about-you-are-not-part-3/