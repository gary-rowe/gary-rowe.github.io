---
author: Gary Rowe
title: How to deploy a Dropwizard project to Heroku
layout: post
tags:
  - Dropwizard
  - Heroku
  - HowTo
  - Tutorial
redirect_from: /agilestack/2012/10/09/how-to-deploy-a-dropwizard-project-to-heroku/
---

The other day I needed to deploy [MultiBit Merchant][2] (a [Dropwizard][3] project) on to [Heroku][4]. This is all part of [my continuous deployment strategy][5] and I thought it would be helpful to others if I posted how I managed to get it is all to work.

### Core concepts

For those not aware of Heroku, here is a quick summary of what it is all about from a Java perspective. 

With Heroku you don’t think of deployment in terms of a target cluster, instead you just treat it as a fully automatic deployment script triggered from a git push. If you want to scale your application up then you add more [Dynos][6] which are either Web Dynos for application serving the public, or Data Dynos for handling your backend end processes like batch jobs and queue slurping. You can think of a Dyno as a container for a single command with 512Mb of memory available to it by default. For more information you should refer to the excellent [Heroku developer support pages][7] which provide a lot of detail on each process.

So, if you can design your application to run from the command line you’re all good. Fortunately, Dropwizard provides exactly this kind of behaviour out of the box.

### Getting started for the first time

Most of the time, you will be driving Heroku from the command line. This is done by their [toolkit application `heroku`][8] which provides a lot of handy functionality – it’s a lot quicker than clicking around a website. 

In the sections that follow I’m assuming that you’ve got a working Maven build for your Java project (it could be a reactor build) and that you’re working from the command line. Since I work with Unix, I’ll prefix all the commands with the usual $ symbol and assume that you’ve got a working git repository with a Maven build of Dropwizard that works locally.

#### 1) Add your Heroku SSH keys

Since this is probably the first time you’ve used Heroku on the local machine, ensure the SSH public key is added to the heroku application:

```bash
$ heroku keys:add ~/.ssh/id_rsa.pub`
```

#### 2) Create Heroku git repo

This provides the link between Heroku and the local app:

```bash
$ heroku create`
```

#### 3) Rename to something sensible

At this stage, you’ll want to rename the app. You can do this through the command line, or through the website. If done remotely, you need to resync the local heroku with the remote repo:  

```bash
$ git remote rm heroku
$ heroku git:remote -a <newname>
```

See [this article about renaming apps][9].

#### 4) Add Procfile with $PORT and $JAVA_OPTS

Ensure the project has a [Procfile entry][10] that looks like this for a Dropwizard project:  

```text
web    java $JAVA_OPTS -Ddw.http.port=$PORT -Ddw.http.adminPort=$PORT -jar path/to/dw/module/target/example-develop-SNAPSHOT.jar server path/to/dw/module/config-heroku.yml
```

The Procfile tells Heroku what to do with the project after it has been built (Heroku will assume a Java build if it finds a pom.xml in the project root). You’ll notice that the configuration indicates a single web Dyno and a command line to execute. 

Heroku will provide various parameters, such as the (single) server port **and various JVM parameters** through environment variables. This last setting is critical since if not included Heroku will configure the JVM to use the maximum 8Gb heap space. This conflicts with the hard limit of 1.5Gb set by Heroku’s internal watchdog so you end up with the situation that your Java process does not see the need to garbage collect, but gets killed for exceeding its memory allocation. Hat tip to Michael Fairley for pointing this out. 

One feature of Dropwizard is that during startup the configuration is read in from a YAML file. Normally this works very well since a different configuration file can be specified for each environment on the command line. However, in the case of Heroku [all parameters are provided as environment variables][11] which require specialised command line handling.

Dropwizard has a feature that [allows command line parameters to override the YAML configuration][12] so long as the property paths are prefixed with dw. This comes in handy when it is used in combination with the `$VARIABLE` notation for Unix environment variables.

#### 5) Set any custom environment variables

If you need to add any custom environment variables of your own, the command line is:  

```text
heroku config:add FOO=bar
```

### Standard workflow

Now that you’re all configured and prepared. This is the normal workflow that you will follow to deploy your application on to Heroku. 

#### 1) Login and update

```bash
$ heroku login
$ heroku update
```

#### 2) Verify the Maven build

Ensure that the correct branch is selected, the project will pass verification and nothing is waiting to be committed (staged). If you’re not [using the “master-develop” branching strategy][13], you should ask yourself why not.  

```bash
$ git -b develop
$ git status
$ mvn clean verify
```

#### 3) Push to Heroku

Deployment is achieved by pushing the release candidate branch to the Heroku repo master branch. This triggers the Java detection process and if you are using master as your deployment branch then the command would be:  

```bash
$ git push heroku master
```

When staging (testing a release candidate), you might choose to deploy direct from your local “develop” branch in which case you should use this git command to push your local branch to Heroku’s master:  

```bash
$ git push heroku develop:master
```

If all goes well, you’ll see your usual Maven build occurring – this is continuous deployment in action. A simple git push and your app is potentially out there.

#### 4) Scale appropriately

At this stage you are ready to go, but you have not allocated any Dyno resources to actually run your app. The command line for this is:  

```bash
$ heroku ps:scale web=1
```

#### 5) Check the processes

Always check your processes are doing what you expect them to be doing:  

```bash
$ heroku ps
```

#### 6) Check the logs

Even though your application is running, it is always worth checking the logs. For a quick dump of the most recent logs use:  

```bash
$ heroku logs
```

Or to monitor continuously with tail -f behaviour use:  

```bash
$ heroku logs --tail
```

#### 7) Open a browser onto the application

Finally, you can view your masterpiece. Issuing  

```bash
$ heroku open
```

will launch your default browser at the appropriate website for your application.

### Conclusion

The world of continuous deployment is now just a git push away thanks to the great infrastructure put in place by the Heroku team. Gone are the days of faffing about with endless configuration settings and arcane XML – now it’s just as it should be – simple and scriptable.

 [1]: https://twitter.com/share
 [2]: https://github.com/gary-rowe/MultiBitMerchant
 [3]: http://dropwizard.codahale.com/
 [4]: http://www.heroku.com/
 [5]: http://gary-rowe.com/agilestack/2012/06/06/multibit-merchant-deployment-driven-design/ "MultiBit Merchant: Deployment Driven Design?"
 [6]: https://devcenter.heroku.com/articles/dynos
 [7]: https://devcenter.heroku.com/articles/java
 [8]: https://devcenter.heroku.com/articles/quickstart
 [9]: https://devcenter.heroku.com/articles/renaming-apps
 [10]: https://devcenter.heroku.com/articles/procfile
 [11]: https://devcenter.heroku.com/articles/config-vars
 [12]: http://dropwizard.codahale.com/manual/core/#id5
 [13]: http://nvie.com/posts/a-successful-git-branching-model/