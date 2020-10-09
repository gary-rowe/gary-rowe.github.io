---
author: Gary Rowe
title: How to deploy static sites with git 
layout: post
tags:
  - Tips
  - HowTo 
  - Tutorial
redirect_from: /agilestack/2012/12/14/how-to-deploy-static-sites-with-git/
---

As part of my contribution to a recent [Random Hacks of Kindness](http://www.rhok.org/solutions/growers-nation-app-development) event, I needed to put together a simple deployment mechanism for the static part of the site. The idea was to use a simple `git push` to trigger a deployment to production. There was a lot of documentation out here on the web about this, but there were many pitfalls along the way that I thought I'd document here to help others attempting to do the same.

So if you want to deploy static files to an EC2 instance using git and have a rollback archive to go back to then here is what you need to do. I'm going to assume that you've set up your EC2 instance to use Linux, and have chosen to serve files from `/var/www/html` and you will put your git repos into `/var/git`. 

I'll further assume that your local machine is running Unix as well and that your private key for your EC2 instance is called `examplekey.pem`. If you're running Windows everything should work the same, but just look a bit different.

### Prepare your server rollbacks

You just need some directories set up on your EC2 server - these just happen to be my preferences.

```bash
$ cd /var
$ mkdir git
$ mkdir www
$ mkdir /var/www/backups
$ mkdir /var/www/html
```

### Prepare your server git repos

Create a bare repo for ExampleSite as follows

```bash
$ cd /var/git
$ git clone --bare ExampleSite
```

### Prepare the post-receive git hook

The deployment magic happens using git hooks. In this case you need the `post-receive` hook, so do the following

```bash
$ cd /var/git/ExampleSite/.git/hooks
$ sudo vim post-receive
```
then populate it as follows:

```bash
#!/bin/bash

echo "Running post-receive"

# Target/working directory - use standard nginx
targetdir=/var/www/html
echo "cd $targetdir"
cd $targetdir

# Zip up what was there before, put it to an archive for emergency purposes
# /var/www/backups/example.YYYYMMMDDHHMMSSz.tgz

time_suffix=`date "+%Y-%m-%d-%H-%M-%S"`
echo "Backup timestamp $time_suffix"
tar czf ../backups/example.$time_suffix.tgz *

# Remove the existing data
echo "Remove $targetdir ..."
rm -fr $targetdir/*

# Check out the local copy of the git repo
echo "Check out local copy"
export GIT_WORK_TREE=/var/git/ExampleSite
export GIT_DIR=/var/git/ExampleSite/.git
cd $GIT_WORK_TREE
git checkout -f


# Copy everything from ExampleSite/src downwards to the above directory (so there's no src dir in the target)
echo "Copying to $targetdir"
cp -r /var/git/ExampleSite/src/* $targetdir

# No TTY interactivity so this is not possible (left in to highlight failing)
# echo "/etc/init.d/nginx restart"
# /etc/init.d/nginx restart
```

Make sure that the script is executable with the proper ownership for the SSH login.

```bash
$ sudo chmod +x post-receive
$ sudo chown ec2-user post-receive
```

and you're done.

### Prepare your local SSH setup

Get the `examplekey.pem` file for the EC2 instance, put it into `~/.ssh` and set restrictive permissions

```bash
$ mv examplekey.pem ~/.ssh
$ chmod 400 ~/.ssh/example.pem
```

Ensure the SSH agent is running on your local machine, and add the key

```bash
$ exec ssh-agent
$ ssh-add ~/.ssh/examplekey.pem
```

This will ensure that git can find it for authentication later on.

Git may get confused when deciding which key to use, so use this "belt and braces" approach by creating (or updating) an SSH config file

```bash
$ vim ~/.ssh/config

Host ExampleHost
     HostName ec2-1-2-3-4.eu-west-1.compute.amazonaws.com
     User ec2-user
     IdentityFile ~/.ssh/examplekey.pem
     IdentitiesOnly yes
```

### Add the remote repo to your local repo

Add a new remote repo called "production" to your local git repo based on the SSH path to the remote repo on the EC2 instance.

```bash
$ cd <path to local repo>/ExampleSite/.git
$ git remote add production ssh://ec2-1-2-3-4.eu-west-1.compute.amazonaws.com/var/git/ExampleSite
```

I found it best to manually edit what git has done to be sure that there is no confusion with keys

```text
[remote "production"]
        url = ExampleHost:/var/git/ExampleSite
        fetch = +refs/heads/*:refs/remotes/production/*
```

To get the above to work, you'll need to have your `examplekey.pem` registered with SSH (see earlier) or you'll get weird authentication errors. If you're really struggling you can attempt the SSH authentication manually with OpenSSL 

```bash
$ ssh -v -i ~/.ssh/examplekey.pem ec2-user@1.2.3.4
```

This will shine a light on any problems that you'll otherwise be scratching your head over. 

### Do the first push

If no-one has ever done this do an initial push of all data in the master branch (assuming that's where your production site content lives) 

```bash
$ git push production +master:refs/heads/master
```

You must have a bare remote repo (see earlier) otherwise git will starting whining at you. If you need to convert a repo back to a bare one then do the following *on the server repo*, but you'll lose any files checked out from the remote repo. 

```bash
$ cd /var/git/ExampleSite
$ mv .git .. && rm -rf *
$ mv ../.git .
$ mv .git/* .
$ rmdir .git
$ git config --bool core.bare true
```

### The usual workflow

Make your changes locally, stage and commit as usual then use 

```bash
$ git push production
```

to deploy your changes to production. You should se the script running and for typical sites it should take a nanblip to complete resulting in minimal downtime for your site.

### Check and rollback 

To check simply visit your site. If there is a total smeg up then you can SSH onto your EC2 instance, and do the following to roll back

```bash
$ cd /var/www/html
$ rm -rf *
$ tar -xzvf ../backups/example.<your timestamp>.tgz *
```

And your old site should come right back as you left it.

### Conclusion

Once this system was in place, the developers working on the static side of the project found that they were able to push their changes extremely rapidly out to the production site. This lead to very short turnaround times for minor tweaks to the site, and encouraged frequent updates. Very agile.
