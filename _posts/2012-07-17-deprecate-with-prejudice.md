---
author: Gary Rowe
title: Deprecate with Prejudice
layout: post
tags:
  - Principles
redirect_from: /agilestack/2012/07/17/deprecate-with-prejudice/
---

When developing an application or library it is important to recognise the importance of deprecation. Without it, dead code hangs around building up [technical debt][2]. The problem is that developers are often reluctant to remove this dead code because they fear that it will break downstream users of the code. The idea is that by preserving the public API they will keep their users happier. 

Unfortunately, that path leads to more technical debt.

When a developer spots that the design of an API is wrong, then they are not alone. Most likely the users of the API have spotted that there is a problem as well. By deprecating a method on an API the developer is clearly informing the users that this shortfall has been noticed and is being actively dealt with. 

Also, if you are continuously changing your public API you are highlighting to yourself that you have not spent sufficient time designing it for flexibility. See m[y article on building APIs that last for decades][3] for more information on that topic.

### Step 1 – Mark deprecated code in two places

Consider the following example, where `java.util.Date` is being deprecated in favour of Joda DateTime (a common occurrence these days):

```java
    /**
     * @deprecated Part of the announced migration to Joda DateTime. 
     * See <a href="http://joda-time.sourceforge.net/">Joda DateTime</a>
     * @since 11.22.33
     */
    @Deprecated
    public Date nowUtc() { ... }
```

The `@Deprecated` annotation is placed on the dead code to ensure that the compiler will inform the user that they are now on unstable code. 

The `@deprecated` annotation is placed in the Javadoc to ensure that the users are given a clear message about what they need to do to fix the problem and get back to safety.

#### Step 2 – Perform a Major or Minor release

Many developers use the Maven versioning system which consists of 3 parts: major, minor and revision. As a general rule the version numbers tend to be increased based on the severity of the change taking place. This gives rise to a useful set of rules that can be applied:

1. Major – use when you have breaking public API changes **and** a lot of new public functionality  
2. Minor – use when there are additional public API calls (without breaking the earlier public API) and significant bug fixes or refactoring in the private code  
3. Revision – you have no changes to the public API and a few minor bug fixes or private code refactoring

One problem that is more common than you might think is “decimal rollover” where a field in a version can reach 9 and then it must rollover to the next level. Avoid this by allowing each of the fields to act independently of each other so that 11.22.33 is a valid release version number. At the time of writing Google’s Chrome browser has a version labelled as 22.0.1201. That’s a lot of revisions.

### Step 3 – Deprecate with Prejudice

In the next minor release (or major if that is next) you follow through with your deprecation. No ifs or buts – just do it. You have given fair warning and by deprecating you have committed to your action and paid down some technical debt.

 [1]: https://twitter.com/share
 [2]: http://martinfowler.com/bliki/TechnicalDebt.html
 [3]: http://gary-rowe.com/agilestack/2012/06/08/multibit-merchant-open-the-pod-bay-doors-hal/ "MultiBit Merchant: “Open the pod bay doors, HAL”"