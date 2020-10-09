---
author: Gary Rowe
title: Default vs Impl - A rant
layout: post
tags:
  - Principles
redirect_from: /agilestack/2010/01/06/default-vs-impl/
---

I am prolific consumer of open source Java software, and I frequently find classes that implement an interface having the suffix Impl. This shouldn’t annoy me, but it does. Why? Putting aside the horrible abbreviation “Impl”, the class is named so that it leaves no room for other implementations. This in turn implies that there is an interface which will only ever have a single implementation and therefore should not be an interface at all.

A better solution, in my opinion, is to name the first implementation with the prefix Default so that future implementations would follow a clear naming convention (such as \[Default\]\[Entity\][Pattern]). Using the word Default implies that this is the first of many implementations for this interface which allows developers to make further implementations as they deem fit. Also, to my eye it looks cleaner. For example, which looks better for the interface UserDao:

    UserDaoImpl or DefaultUserDao
    UserDaoImplJdbc or JdbcUserDao
    UserDaoImplHibernate3 or Hibernate3UserDao

I will concede that the “Impl” naming convention allows an IDE to present a complete list of classes matching on the single word User leading to less developer thinking time. But all decent IDEs provide wildcards in the class lookup selector, and an “Implementations Of” view as well so it’s just a case of different starting points. Also, the “Implementations Of” approach catches those implementing classes that don’t follow the naming convention.

So it’s settled then: Default wins.

 [1]: https://twitter.com/share