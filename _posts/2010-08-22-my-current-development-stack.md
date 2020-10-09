---
author: Gary Rowe
title: My Current Development Stack
layout: post
tags:
  - Development
redirect_from: /agilestack/2010/08/22/my-current-development-stack/
---

I write web applications in Java, and have been doing so for some time now. I like to think that I’ve become pretty good at it, and I can usually get something working quickly and efficiently. This means that my choice of frameworks and tools is somewhat specialised, and others would make a different choice, perhaps [Grails][2] or [Ruby on Rails][3]. This article is not about which framework is superior to another, but rather what I’m currently using and why it’s good for me right now.

Next year, it should be different otherwise I’m not really progressing.

## IDE

[Intellij IDEA][4]. I used to be an [Eclipse][5] evangelist, but then [my friend Gareth][6] introduced me to  
Intellij and I never looked back. Well, maybe once or twice when I had to reorient myself within the new IDE, but once I got the hang of it then that was it. Everything just works really well, and the refactoring is amazing. Especially in XML and JavaScript. Oh, you thought refactoring was just for Java? I invite you to download the freebie version of Intellij and try out the goodness for yourself. If you’re a contractor like me you’ll find that the productivity increase pays for the IDE within a day or two.

## Build systems

[I’ve written about this before][7], but there is only one build system for me at the moment: [Maven][8]. I think it reduces the problem of creating portable builds (and environments) to a pretty simple format. It’s a pretty straightforward way of allowing a disparate team of developers to work on different aspects of a large project. In my opinion, a build system should be able to get a new developer productive within 30 minutes. That includes time reading the wiki to locate the appropriate project and check it out of version control. It also includes the time taken for the developer to figure out (using only the documentation) how to build and deploy it locally with a sample set of data and a demonstration page so they can see how the application is supposed to work. If your set up does not allow a developer to do this, then – seriously – look at your systems.

In Maven, the way I’ve got my stuff set up, you get a fully operational environment as described above by doing the following:

1) Check out your project from version control  
2) Type mvn clean jetty:run at the command line in the project checkout directory  
3) Navigate to localhost:8080/SomeApp in the browser (the wiki will tell you where to go)

That’s it, and it’s the same procedure for all applications at all tiers of the stack. The act of  
issuing that command will automatically ensure that the correct supporting versions of any other JARs and WARs are downloaded, included and executed within the same Jetty instance. This gives a complete set of interactivity so that a developer knows they have a representative slice of the overall system. Not bad for a single memorable command, eh?

## Web services with RESTEasy and JAXB

I used to use [SpringMVC][9] for my web front end. It remains an excellent implementation of the standard Model/View/Controller design pattern that made front end design much simpler than the frameworks that had gone before – I’m looking at Struts here. Now, I’m hooked on an even simpler framework that trivialises the whole creation of web services to a mere typing exercise: [RESTEasy][10]. Wow. What a revelation! I’m sure that in [Spring 3+][11] they’ve done something similar (and I will take a look, I promise) but right now RESTEasy is my best friend. Here’s an example of creating a simple web service to respond to a GET request:

```java
@Path("/hello")
public class Hello {

  @GET
  @Path("/{name}")
  public String example(@PathParam("name") String name) {
    return String.format("Hello, %s",name);

  }

}
```

which provides

```text
HTTP GET /ExampleApp/hello/bob

Hello, bob
```

It’s difficult to imagine a simpler way of coding that service up. Nothing is wasted, the web.xml is trivial and the same for pretty much all the web services on the back end. And it integrates with Spring too. And [JAXB][12]. And [JSON][13]. And Maven. In fact, it’s damn near perfection.

And what’s so good about JAXB I hear you ask? Well, let me tell you. Take a POJO, apply some simple annotations and hand it over to RESTEasy. All that hard effort of turning it into XML or JSON or whatever for transmission to the outside world is all taken away. Consider a simple class that provides a list of Users formatted as a paged result set:

```java
@XmlRootElement(name="Result")
public class Result {

  @XmlElementWrapper(name="Users")
  @XmlElement(name="User")
  private List<User> users=new ArrayList<User>();

  @XmlAttribute(name="firstResult")
  private int firstResult;

  @XmlAttribute(name="batchSize")
  public int getBatchSize() {

  return users.size();

  }

  public void setFirstResult(int firstResult) {

  this.firstResult=firstResult;

  }
}
```

Assume that User has been created and marked up in a similar fashion. Put this into a RESTEasy web service, like so

```java
@Path("/search")
public class SearchService {

  UserDao userDao = new UserDao();

  @GET
  @Path("/{query}")
  public Result example(@PathParam("query") String query) {

  return userDao.search(query);

  }
}
```

You’ll see the following

```text
HTTP GET /ExampleApp/search/bob

<Result firstResult="1" batchSize="1">
<Users>
  <User>
    <Name>Bob</Name>
  </User>
  </Users>
</Result>

```
Neat and very obvious how it all works so maintenance issues drop off.

## Database access with JPA

Now I jumped on the Hibernate wagon with everyone else. It solved a huge problem in the ORM world in a neat and elegant manner. It also educated mere mortals into the proper way to think about the way our applications dealt with database issues. I’m thinking of sessions, lazy loading, avoiding the dredge anti-pattern, entity to table mappings and on and on.

In fact Hibernate was so good that it sparked a complete revisit of the awful EJB approaches so that EJB3 is actually palatable – if you’re starved of any alternative that is. I’m still not convinced that EJB is compatible with an Agile approach to development. I’ve not yet seen an EJB work efficiently in a Jetty environment that neatly supports JUnit integration testing against an in-memory HSQL database. Spring on the other hand manages that feat with ease, particularly in JUnit4.

One good thing about EJB3 is the creation of the JPA and JTA specifications, which formalised what had been going on in Hibernate for ages before. All those lovely annotations were ratified and the persistence layer took a major leap forward. As a developer this puts me in a good position where I can be ambivalent about the implementation technology behind the annotations. Want me to code up your DAOs using JPA and JPQL, that’s fine. Want the same done with Hibernate and HSQL, just gimme a mo to tweak some queries and you’re good to go.

## Automated functional testing

[I’ve discussed this elsewhere too][14], but I thought I’d include a mention here for completeness. No development stack is complete without catering for automated testing. Everyone reading this naturally has a unit test for everything – don’t you? You don’t?! You’d better get started then, ‘cos everyone else has. However, once you’ve got your unit tests working nicely with both dependency injection and mocking approaches. Which reminds me, I’d recommend JMockit over JMock these days due to it’s better handling of static methods which are sometimes forced on us by legacy code. Then it’s time to consider the functional testing. For some reason everyone seems to adopt the line of “Oooh, I’d love to do that but I just don’t have the time.”, which in my opinion doesn’t add up. Every developer should strive to make their days easier, and having the safety net of a  comprehensive set of functional tests is essential. Take a look at my detailed article on the most excellent SeleniumRC for more information.

## Performance

With the above technologies, I can usually get a full-blown, integration tested RESTful web service off the ground and operational against a database within 2 hours. Most of that time is spent typing and configuring for the particular problem I’m facing. I know other frameworks exist that could speed that up (Appfuse and Spring Roo, er, spring to mind – sorry). And, yes,  
I do have templates to solve standard problems. However, within the constraints imposed by my clients who want to use, or be guided into using, a particular technology, this is a pretty good delivery time.

 [1]: https://twitter.com/share
 [2]: http://www.grails.org/
 [3]: http://rubyonrails.org/
 [4]: http://www.jetbrains.com/idea/
 [5]: http://www.eclipse.org/
 [6]: http://careers.stackoverflow.com/garethdavis
 [7]: http://gary-rowe.com/agilestack/2010/01/14/how-to-be-agile-when-all-about-you-are-not-part-5/
 [8]: http://maven.apache.org/
 [9]: http://static.springsource.org/spring/docs/2.0.x/reference/mvc.html
 [10]: http://jboss.org/resteasy
 [11]: http://www.springsource.org/
 [12]: http://en.wikipedia.org/wiki/Java_Architecture_for_XML_Binding
 [13]: http://www.json.org/
 [14]: http://gary-rowe.com/agilestack/2010/01/26/creating-agile-functional-tests-with-selenium/