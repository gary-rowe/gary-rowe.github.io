---
author: Gary Rowe
title: MultiBit Merchant - Genesis
layout: post
tags:
  - MBM
redirect_from: /agilestack/2012/06/06/multibit-merchant-genesis/
---

Back in October 2011, I decided to write an online shop. Several months on (at the time of writing) I’m still at it.

I’m not doing it because I want to enter the market that already exists. Just looking at Shopify, BigCommerce, and almost every hosting site there is, tells me that it’s ridiculously overcrowded already. I’m doing it because I have a particular itch to scratch.

I want to see how well I can put together a complex application when I have the following restrictions:

* it must adhere to RESTful principles
* it must be open source under the MIT license
* it must **not force the use of any commercial service**
* the first released version must be written in Java (with optional JavaScript)
* the first released version must be used as part of a workable business

Each of those restrictions introduce some tricky, and therefore interesting, design considerations.

### RESTful principles

Well that sounded easy enough – anyone who’s anyone is writing a RESTful API these days. What’s so hard about that? Well, let me tell you that writing a proper RESTful API is nowhere near as easy as you first think. I’ll discuss how I approached that [in a later article][2].

### MIT license

In short, the MIT license means that anyone can take your code and do whatever they want with it, subject to keeping a copyright notice for original author attribution. That essentially means that it would be unrealistic for me to sell this code because anyone can just make their own copy and compile it. On the other hand, it does mean that I can use it as a showcase for my own work that may interest future clients. Also, it means that anyone would be able to use this software to make their own online shop.

Since it’s all going to be out in the open, I can feel free to blog about my progress and can showcase good agile development techniques as well. This means making use of the best of breed tools for coding ([Intellij IDEA][3]), collaborative version control (git on [GitHub][4]), continuous integration ([Jenkins][5]), scalable deployment (yet to be determined).

When it comes to source control, I believe that nothing beats the power and simplicity of git.  As a result I made the choice to host the [MultiBit Merchant project repository][6] on GitHub. This will give it a wide audience and provide me with good tools to actually work with users and collaborators.

### No commercial service

At first, this appears to be the most difficult restriction of all. Selling stuff through an online shop means taking payments online, and that means accepting credit cards. To do that means signing up to a payment gateway. Which leads to all kinds of additional compliance measures: Know Your Customer, PCI regulations, storage of personal data, restrictions on age and location of customers, restrictions on sales items, risk of chargebacks, transaction processing fees (how can a large transaction cost more to process than a small one?). The list goes on.

How can it be possible to meet this restriction? Enter [Bitcoin][7].

For those that are unaware of it, Bitcoin provides an additional payment gateway that has the following features:

*   You don’t have to pay any fees to receive bitcoins
*   The fees to send them are tiny (usually less than a penny worldwide)
*   You don’t have to register with anyone to send or receive them
*   You can send a wide range of amounts cost-effectively (from under a penny to 1,000s pounds internationally)
*   You can exchange bitcoins for most currencies (certainly GBP, USD and EUR) any day of the week 24/7
*   You can receive payments offline (i.e. you can print the address on a business card if required)
*   You can receive payments from anyone, anywhere in the world (Bitcoin is global)
*   You are immune from problems with payment processors
*   It is cryptographically secure
*   All the software required to operate them is free and open source
*   The software runs on Windows, Mac, Linux, iPhone and Android
*   Bitcoin itself promotes a fair and just economy by offering people an alternative currency not under anyone’s control (no government, no corporation, just a collaborative global effort)
*   Lots of other businesses are using it
*   It has been running continuously since 2009

That’s good enough for me.

### Must be Java

Java seems to suffer under the image of being a stuffy “enterprise” language these days. I could have gone for something else like Clojure, or Scala, but this would have introduced a fairly serious language learning curve. Learning a new language typically means following the [“11-step algorithm”][8] and the first few steps are usually quick to accomplish. However, learning the idioms takes a lot longer. I know that I can write a better application in Java than I can in Clojure right now, and that’s important for this project.

Still, Java is on the back foot when it comes to hosted services. In comparison to the number of hosting companies offering support for PHP, Ruby on Rails and Python the ones providing a Tomcat servlet container are few and far between. Is Java forever doomed to remain in the data centers of the large and powerful corporations, or are there alternatives? More about that [in the next article][9] which will discuss deployment issues and how they shape the design.

### Must be part of a workable business

In short, I must [“eat my own dog food”][10]. I’m always on the lookout for new and interesting business opportunities, but I always seem to stop short of getting the business off the ground. Having a quick and easy way of getting a shop up and running to meet a particular need, without all the credit card hassle, would solve a particular problem that I have, and perhaps this would benefit everyone.

Experience has told me that it is essential that any software operating online must be secure by design. This is particularly important when applied to a Bitcoin shop since bitcoins are a frequent target for hackers in just the same way that credit card numbers and PayPal credentials are. Also an online business must be easy to look after since it won’t bring in much money to begin with and the owner must continue with their day job as they build it up. The more automation it can offer the easier it is for that person to build it up into something self-sustaining, and ultimately free them from their day job.

### Where next?

So now that I’ve set out my store, so to speak, I am going to keep a blog of my progress. Hopefully, this will help other developers following a similar path for their own projects and maybe I can provide (and receive) some insight that helps make MultiBit Merchant useful.

You can read more about my progress by following the articles labelled with the “MBM” tag. Or [jump straight to the next article now][9].

 [1]: https://twitter.com/share
 [2]: http://gary-rowe.com/agilestack/2012/06/08/multibit-merchant-open-the-pod-bay-doors-hal/
 [3]: http://www.jetbrains.com/idea/download/
 [4]: https://github.com/
 [5]: http://jenkins-ci.org/
 [6]: <a href=
 [7]: http://bitcoin.org
 [8]: http://programmers.stackexchange.com/a/23411/7167
 [9]: http://gary-rowe.com/agilestack/2012/06/06/multibit-merchant-deployment-driven-design/
 [10]: http://en.wikipedia.org/wiki/Eating_your_own_dog_food