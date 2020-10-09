---
author: Gary Rowe
title: Creating agile functional tests with Selenium
layout: post
tags:
  - Design
  - Selenium
  - Testing
redirect_from: /agilestack/2010/01/26/creating-agile-functional-tests-with-selenium/
---

So you’ve implemented a bunch of unit tests and you’re feeling confident that your system works as expected. Then you realise that you now have to work through all those different use cases one by one every time you make a change just to be sure that you haven’t somehow broken the front end. Surely there must be a way of automating this process? There is, and it’s called [Selenium][2].

In this article I’m going to focus on how to introduce [Selenium][2] into your development process assuming that you have developed a reasonably large web application based on Java technology. Selenium will work quite happily with a wide variety of other development languages, but since I’m not so familiar with them I’ll leave that to others to explain.

So, having lost 90% of my readers, I’ll struggle on.

I define a functional test as one that essentially mimics a user working through a particular collection of use cases. In the case of a web application, this means that a functional test needs to remotely control a browser; fill in various forms or click links; and verify that the responses from the server are as they should be. Typically in the Java world some variant of HttpUnit is used for this purpose but this only represents the behaviour of a single browser – the HttpUnit implementation.

As an aside, it should be noted that there is a strong correlation between functional tests and requirements. It is indeed possible to create a functional test matrix that allows requirements to be ticked off as their corresponding functional tests are seen to pass. This follows both the DRY principle and the Agile manifesto maxim of “Working code is the best metric”.

So what does Selenium do? Specifically, in it’s [Selenium Remote Control (SeleniumRC)][3] form it acts as a bridge between the browser and your automated build process. You write your tests in [JUnit][4] as usual ensuring that your assertions target the responses coming back from a Selenium browser object. For example:

```java
selenium().open("http://localhost:8080/mywebapp/index.html");
assertTrue(selenium().isTextPresent("Hello World");
```

The selenium() method provides access to the underlying browser. If the assertion fails then the test case fails and the usual reporting mechanism takes over.

OK, so what is going on here? Essentially, selenium() communicates with a SeleniumRC server instance, which is just a single JAR running in the background somewhere, and instructs it to fire up a browser, point it at the given URL, wait for the page to load, then check if the given text is present in the response. Not bad for 2 lines of code. Now here is where SeleniumRC gets very clever: first, you can instruct it to fire up an instance of pretty much any browser you like; second, it will inject some JavaScript into the page which will allow Selenium to perform deep selections within the browser DOM.

That abstraction of the browser, and the corresponding deep inspection, makes Selenium extremely powerful when used in a large scale environment. Since Internet Explorer does not sit well with other versions it is usually necessary to create a collection of virtual machines each with their own version of IE and SeleniumRC installed. Other virtual machines can be created that allow [Firefox][5], [Opera][6], [Chrome][7], [Safari][8] etc to be installed in a variety of combinations. However, the same suite of functional tests can be executed against them without change (so long as you use CSS selectors throughout otherwise IE runs like a dog). In a [Maven][9] environment this can be implemented as a series of executions each with a few environment parameters that can be passed to the test suite. The environment parameters indicate where the SeleniumRC server is to be found (the VM) and what browser it should target. For example (in build/plugin):

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <!-- Default configuration is to skip -->
  <configuration>
    <skip>true</skip>
  </configuration>
  <executions>
    <execution>
      <!-- Test with Firefox 3 -->
      <id>1_functionalTestWithFirefox</id>
      <phase>verify</phase>
      <goals>
        <goal>test</goal>
      </goals>
      <configuration>
        <skip>${maven.test.skip}</skip>
        <environmentVariables>
          <selenium-rc.server-host>selenium-xp-ie7ff3</selenium-rc.server-host>
          <selenium-rc.server-port>4444</selenium-rc.server-port>
          <selenium-rc.browser-type>*chrome</selenium-rc.browser-type>
        </environmentVariables>
      </configuration>
    </execution>
</executions>
</plugin>
```

But wait, there’s more.

Since a functional test is considered as a collection of use cases it makes sense to structure the testing environment to follow this pattern. Essentially a series of standard unit tests are created with a FunctionalTest suffix to make their purpose clearly visible. These are usually packaged in line with the various system modules that they test. The functional test simply delegates to the use cases they wrap (passing in a reference to themselves to allow the use case to access test methods). Since there is a lot of common functionality between functional tests (such as the initial creation of the Selenium browser reference) it makes sense to introduce an abstract base class that provides these references. For example:

```java
package org.example.web.functest.security;
@RunWith(BlockJUnit4ClassRunner.class)
public class LoginAdminFunctionalTest() extends AbstractWebTestCase {
  ...
  @Test
  public void loginAdmin() {

    LoginAdminUseCase login = new LoginAdminUseCase(this);
    login.execute();
  }
}
```

By contrast, the use cases are simply [implementations of the Command pattern][10] which do all the work. They have a UseCase suffix to allow for quick identification, and typically extend an abstract base class to provide useful common methods. Extending org.junit.Assert will provide direct access to assertion methods. For example:

```java
package org.example.web.functest.security.usecase;
public class LoginAdminUseCase() extends AbstractUseCase {
  public LoginAdminUseCase(AbstractWebTestCase tester) {
    // The abstract super class provides various selenium methods via tester
    super(tester);
  }

  public void execute() {

    selenium().windowMaximize();
    selenium().deleteAllVisibleCookies();

    // Obviously the URL here would be passed
    // as a parameter by the build process
    selenium().open("http://localhost:8080/mywebapp/login.html");

    // Assumes that AbstractUseCase extends Assert
    assertEquals("My Login Page", selenium().getTitle());

    selenium().type("username", "admin");
    selenium().type("password", "admin");
    selenium().click("submit");

    // Convenience method to introduce standard short delay
    waitForPageToLoad();

    assertEquals("Administration console", selenium().getTitle());

  }
}
```

Any number of functional tests could re-use the LoginAdminUseCase in any combination. It would be trivial to include a LogoutUseCase that clicked a common logout link. In fact, at the end of every test as part of the tearDown() sequence the LogoutUseCase could be invoked.

I have glossed over several important implementation details in the interests of brevity. For example, the issue of what acts as the container for the web application, and how this can be invoked before testing starts. My favoured approach to this is twofold: one for developers the other for continuous integration servers.

The developer version uses [Jetty][11] to act as a lightweight servlet container that can be stopped and started very quickly; is responsive to changes in the web application (e.g. a JSP or CSS adjustment) while running; and allows for programmatic configuration within the functional test module. This allows a developer to work in a much smaller environment with perhaps a single functional test driving their work so there is very little repetitive typing or page navigation going on. Since the environment responds immediately to changes very rapid progress can be made.

By contrast, the continuous integration version relies on Maven to start and stop some larger application container that is representative of the final deployment environment. In the case of[ Glassfish this is the asadmin plugin][12], but others exist for different containers (notably [the general purpose deployment plugin for Cargo][13]).

In addition to the container, the database must also be regulated so that provides predictable results. Usually, after the application container has been stopped a script is executed against the database to drop all objects and rebuild. Clearly the test database is a minimal subset of the actual production schema. It must be large enough to allow a wide range of use cases to be explored, but small enough so that it doesn’t dominate the build process.

Once the application container is restarted, then the build process can begin the functional test suite with each of a variety of browsers being targeted at the system under test. The individual functional test results are collected and presented as part of the final build result.

 [1]: https://twitter.com/share
 [2]: http://seleniumhq.org/
 [3]: http://seleniumhq.org/projects/remote-control/
 [4]: http://www.junit.org/
 [5]: http://www.mozilla.com/en-US/firefox/upgrade.html
 [6]: http://www.opera.com/browser/
 [7]: http://www.google.co.uk/chrome
 [8]: http://www.apple.com/safari/download/
 [9]: http://maven.apache.org/
 [10]: http://en.wikipedia.org/wiki/Command_pattern
 [11]: http://jetty.codehaus.org/jetty/
 [12]: http://eskatos.wordpress.com/2008/03/24/a-simple-maven-plugin-for-glassfish-asadmin-maven-plugin/
 [13]: http://cargo.codehaus.org/Maven2+plugin