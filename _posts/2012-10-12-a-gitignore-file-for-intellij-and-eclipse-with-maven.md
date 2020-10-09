---
author: Gary Rowe
title: A .gitignore file for Intellij and Eclipse with Maven
layout: post
tags:
  - Tips
  - HowTo
redirect_from: /agilestack/2012/10/12/a-gitignore-file-for-intellij-and-eclipse-with-maven/
---

I often find myself having to fiddle about with ignore settings in various IDEs, so I thought I’d put together a simple general purpose solution that will ignore all the usual suspects for Intellij and Eclipse within a Maven reactor build (even works on a Mac).

Here it is:  

```text
# Eclipse
.classpath
.project
.settings/

# Intellij
.idea/
*.iml
*.iws

# Mac
.DS_Store

# Maven
log/
target/
```

If you place this in `.gitignore` the root directory of your project, the ignore settings will be applied to all sub-directories automatically.

Finally, no more accidental commits of `/target` into the repo.

You may also want know how to purge the commit history to get rid of those old bloating commits, or perhaps an accidental commit of a password. If so the GitHub people [have written an excellent tutorial][2].

 [1]: https://twitter.com/share
 [2]: https://help.github.com/articles/remove-sensitive-data