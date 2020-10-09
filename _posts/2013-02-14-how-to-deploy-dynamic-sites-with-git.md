---
author: Gary Rowe
title: How to deploy dynamic sites with git
layout: post
tags:
  - HowTo
  - Tips
  - Dropwizard
redirect_from: /agilestack/2013/02/14/how-to-deploy-dynamic-sites-with-git/
---

Automation is everything. The more you can get to occur without any effort from you the more time you have available to do other more important work (and there is less chance of an important step being missed due to manual error).

So the other day I set myself the task of building a git hook script that would automatically update a Dropwizard application simply by means of a git push. Yes, this is the logical next step from the static site deployment that [I covered in an earlier article](http://gary-rowe.com/agilestack/2012/12/14/how-to-deploy-static-sites-with-git). At this point you will probably want to refer to that article so that you have the necessary local configuration in place.

Assuming that you've followed the instructions there and you've got it all set up for static content. Here's what you need to put in the `.git/hooks/post-receive` script so that you get the same effect for, say, a Dropwizard project:

```bash
#!/bin/bash
@echo off

# This script will live in git repo as .git/hooks/post-receive
targetdir=/var/git/ExampleSite

# Target/working directory
cd $targetdir

# Stop the existing service (this may take some time so we do it before the Maven build)
echo Performing a soft kill of the existing process
pkill -f example-site-develop-SNAPSHOT

# Check out the local copy of the git repo
echo "Check out local copy"
export GIT_WORK_TREE=/var/git/ExampleSite
export GIT_DIR=/var/git/ExampleSite/.git
cd $GIT_WORK_TREE
git checkout -f

# Build with Maven
echo Maven build the new code
mvn clean package

# Start the new background process
echo Starting the new process under nohup
nohup java -jar target/example-site-develop-SNAPSHOT.jar server example.yml > /var/log/example/example-log.log 2>&1 &

echo Complete. Verify operation by visiting http://example.org/
```

The above script is a bare bones version and you'll probably want to make it more robust for a production server. However, we'll just plough ahead assuming that you're working on a test or development box that doesn't need to maintain 24/7 uptime. As an aside, you'll notice that the Maven version is "develop-SNAPSHOT" this is quite a handy way to represent the current development snapshot in the ["master-develop" branching strategy](http://nvie.com/posts/a-successful-git-branching-model/).

On your local machine make some changes that would be visible, then issue this after the commit:

```bash
$ git push production develop:master
```

This will cause the following behaviour

* The local develop branch will be pushed to the remote master branch
* The site will go offline almost immediately
* A Maven build will take place and be seen in the console
* The site will be restarted as a `nohup` process

Obviously there are a lot of improvements that could be made to the script, but this should be enough to get you started and productive.

Happy deploying!