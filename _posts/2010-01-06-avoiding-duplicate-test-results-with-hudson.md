---
author: Gary Rowe
title: Avoiding duplicate test results with Hudson
layout: post
tags:
  - Tips
  - Hudson
  - Maven
  - Testing
  - Workaround
redirect_from: /agilestack/2010/01/06/avoiding-duplicate-test-results-with-hudson/
---

I’m a big fan of automated testing, in fact automated everything since I quickly lose interest in repetitive tasks. As a result of this I’m drawn to build systems like [Maven][2] which provide a lot of good conventions and automation right out of the box.

There are a million articles out there about Maven so I’m not going to repeat what they say here, but I would like to comment about a particularly tricky problem that I encountered a little while ago which ended up having a very neat solution.

[Hudson][3] duplicates the results of multiple executions of a test suite through Maven so that a failure in, say, the first run is overwritten by a success in the second. Hudson signals the failure in the build report and the test report contains multiple entries with the same class and method name, one marked as a failure the other as a success and no reliable stack trace highlighting which of the executions was responsible for the failure. How can Hudson be made to provide separate test reports for each test suite execution?

Firstly, it should be noted that Hudson runs Maven in a special way which allows it to trigger operations when plugins complete. In the case of automated testing Hudson is able to slip in between plugin executions and extract the test results before another plugin is able to execute. This rules out a plugin based solution (although I did have some fun writing one only for it to fail to solve the problem).

The way to solve the problem is to change the JUnit4ClassRunner that is being used by [JUnit][4] to one of your own making which is able to modify the returned name of the test method that has been executed. This allows a test method to have, for example, a different suffix depending on an environment variable that can be passed in by Maven. This feature is easy to implement if you are using JUnit4 (which is backwards compatible with JUnit3 so there’s no excuse not to upgrade).

In Maven the [surefire plugin][5] provides access to JUnit tests and thanks to the 
`executions` configuration element the plugin can be repeated with different settings. For example,

```xml
<executions>
  <execution>
    <id>1_FF3</id>
    <phase>integration-test</phase>
    <goals>
      <goal>test</goal>
    </goals>
    <configuration>
      <environmentVariables>
        **<browser>ff3</browser>**
      </environmentVariables>
    </configuration>
  </execution>
  <execution>
    <id>2_IE6</id>
    <phase>integration-test</phase>
    <goals>
      <goal>test</goal>
    </goals>
    <configuration>
      <environmentVariables>
        **<browser>ie6/browser>**
      </environmentVariables>
    </configuration>
  </execution>
</executions>
```

The above, if placed within a surefire plugin declaration, will run the test suite twice, once passing in “ff3″ as an environment variable (“browser”) and the second “ie6″.

Typically, most JUnit test cases inherit from some common base class that provides common methods, so I’ll assume that you’ve got something like AbstractWebTestCase below:

```java
@RunWith(JUnit4ClassRunner.class)
public abstract class AbstractWebTestCase {}
```

The next step is to write your own replacement for JUnit4ClassRunner so that the Description object is constructed with slightly altered values. This is actually pretty trivial since you only need to override a single method:

```java
public class SuffixEnabledJUnit4ClassRunner extends JUnit4ClassRunner {
    public SuffixEnabledJUnit4ClassRunner(Class clazz) throws InitializationError {
        super(clazz);
    }
    @Override
    protected Description methodDescription(Method method) {
        String suffix = System.getenv("browser");
        return Description.createTestDescription(
          getTestClass().getJavaClass(),
          testName(method)+suffix,
          testAnnotations(method));
    }  
}
```

Using the above example, you should be able to construct your own JUnit4ClassRunner which will modify your method name descriptions in the manner of your choosing. Hudson will merrily use whatever name is given out by your ClassRunner so feel free to experiment.</div> </div>

 [1]: https://twitter.com/share
 [2]: http://maven.apache.org/index.html "Maven build system"
 [3]: https://hudson.dev.java.net/ "Hudson continuous integration"
 [4]: http://www.junit.org/ "JUnit home page"
 [5]: http://maven.apache.org/plugins/maven-surefire-plugin/ "Surefire plugin home"