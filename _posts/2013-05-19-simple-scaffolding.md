---
author: Gary Rowe
title: Simple Scaffolding
layout: post
tags:
  - Tips
  - HowTo
redirect_from: /agilestack/2013/05/19/simple-scaffolding/
---
One of my biggest problems with day to day Java coding is a lack of easy code generation. Frequently I find myself working on
 a new project and having to set up all the same boilerplate code. Obviously I'll try to minimise the effort involved
 by first reaching for a [Maven archetype](http://maven.apache.org/guides/introduction/introduction-to-archetypes.html)
 or, more recently, a [git repository with a standard set of starting code](https://github.com/gary-rowe/DropwizardOpenID).
In the past I even gave [Spring Roo](http://www.springsource.org/spring-roo) a try out, and [Grails is pretty good](http://www.grails.org/)
if you're allowed to use Groovy where you work.

Unfortunately, all of the above come with drawbacks. Maven archetypes are cumbersome to create and maintain. The git repo
approach is great to get started, but doesn't go any further. Roo and Grails mandate certain design choices that
may not be compatible with your workplace.

### Feel my pain

Consider a non-trivial application where there are several developers working on it. To ensure that developers
write maintainable code it is layered and uses the [Hexagonal Architecture](http://alistair.cockburn.us/Hexagonal+architecture).
We'll allow each developer the full run of the whole application so they'll need to do the following when adding a new
entity, such as a User, into the system:

1. Create some basic User JSON to power a test fixture (usually just an ID field)
1. Create the User representation with JSON markup
1. Create a serialization unit test for the User
1. Create a CRUD Resource with POST, GET, PUT, DELETE methods with an accompanying functional test
1. Create a UserDomain object with persistence markup
1. Create read and write service interfaces (e.g. `UserReadService` and then `MongoUserReadService`) with test
1. Create repositories backed by DAOs or JMS queues with test
1. Create the front end support
1. ... and so on

Clearly there is scope to automate much of the above, and make extensive use of common support libraries and generic
implementations of base classes. No matter what, though, at some point the User-specific behaviour will need to be
catered for and that is where code generators come in handy. Again, there are snippet libraries (some are even shared)
and the endless fiddling with code generation within IDEs can go a long way to generating those starting points.

But it's still just too complicated and not easily transferable between projects and developers.

### Stating the problem

What I want is a simple way to:

1. read an existing project
1. lift its source code
1. transform the source into templates with markup for various parts (class name, package, variables etc)
1. use those templates to re-create the code for different entities and projects

### Presenting a solution: [Simple Scaffolding](https://github.com/gary-rowe/SimpleScaffolding)

Over a weekend I quickly put together a single class: `Scaffolding`. It's a general purpose project reader that creates
templates. To "install" it you just copy the `Scaffolding.java` into your project under `src/test/java`. It's open source
under MIT license so you will have no problems just adding it. It requires [Guava](https://code.google.com/p/guava-libraries/wiki/GuavaExplained?tm=6)
and [Jackson](http://wiki.fasterxml.com/JacksonHome) which are pretty standard support libraries these days so shouldn't present an issue.

All configuration is done through `scaffolding.json`:
 
```json
{
  "base_package":"uk.co.froot.example",
  "read": true,
  "entities": ["User"]
}
```

All code from `base_package` and below is recursively examined and the templates built. These are stored in a directory
structure under `src/test/resources/scaffolding` for inclusion in version control and subsequent use by other developers.
If a class including the name of one of the entities (`User`) is discovered like `MongoUserReadService.java` for example,
then it will be treated as an entity template.

Any class that does not include the name of one of the entities will be just a standard file that gets included everywhere
(like `DateUtils` if you can't have a common support JAR for some reason).

The final step is to delete any that are not useful and edit those that remain to meet your requirements. The idea is to edit them
to be as general purpose as possible (no entity-specific fields beyond the common ID for example).

### Writing from templates

Once you have your templates in place, you can use them to generate new code. This is again driven by the `scaffolding.json`
files. You switch away from `read` and provide a list of new entities that you would like created:

```json
{
  "base_package":"uk.co.froot.example",
  "read": false,
  "entities": ["Role","Customer"]
}
```

Using the above, the generic templates built from the `User` will be used to produce the equivalent for `Role` and
`User`. For example `MongoRoleReadService` and `MongoCustomerReadService`. If you have been careful with what was
included in the original `User` then the produced code will act as good launch point for the new entities.

### Conclusion

Simple Scaffolding is *not* intended to be a complete solution to the problem of code generation. It's just a simple way
to get a bunch of code representing an architectural choice easily replicated throughout your application. Over time
you'll build up a useful library of templates that fit with different types of projects which should add up to
a [considerable time saving](http://www.xkcd.com/1205/).

Why not give it a go and let me know how you get on in the comments?