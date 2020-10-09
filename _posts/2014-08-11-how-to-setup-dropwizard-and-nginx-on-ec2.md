---
author: Gary Rowe
title: How to set up Dropwizard and nginx on EC2
layout: post
tags:
  - HowTo 
  - Dropwizard
redirect_from: 
---
It's been a while since I've added any articles to this blog. This has been due to working flat out on [MultiBit HD - a Bitcoin startup project](https://multibit.org).

As part of that work I needed to add a Dropwizard project to an EC2 box fronted by nginx. As usual I wanted full automation so the whole build and deploy process was
to take place on the server (it's only a small build) in response to a git push. [I reviewed my earlier work](http://gary-rowe.com/agilestack/2013/02/14/how-to-deploy-dynamic-sites-with-git)
but found that it didn't give a complete set of instructions for setting up on a bare bones server, so I thought I'd add some.

I'll assume that you have an AWS account in place so this is just a case of commissioning a box. Naturally this process could be scripted but sometimes just having the
 manual steps available can be useful.

## Use AWS to set up your bare bones EC2 server

Create a bare bones Ubuntu instance on the Free tier (micro should be fine for development projects).

Do the following with the `examplekey.pem` file for the EC2 instance from Security Key Pair:

```bash
$ mv examplekey.pem ~/.ssh
$ chmod 400 ~/.ssh/example.pem
```

Ensure the SSH agent is running on your local machine, and add the key

```bash
$ exec ssh-agent
$ ssh-add ~/.ssh/examplekey.pem
```

## Configure for HTTP, SSH, HTTPS as usual

Do the usual AWS security settings to enable HTTP, SSH and HTTPS

## Login with the `ubuntu` user

```bash
$ ssh -i .ssh/examplekey.pem ubuntu@1.2.3.4
```

## Install the infrastructure

```bash
$ sudo -i
$ apt-get upgrade
$ apt-get update
```

## Install JDK7, SSL, curl and nginx in one hit

```bash
$ apt-get install openjdk-7-jdk openssl curl nginx
```

Key files for later:

```bash
/usr/share/nginx/www
/etc/nginx/nginx.conf
/etc/nginx/sites-available/<host name>
```

## Verify nginx

Visit `http://1.2.3.4`

You should see the standard nginx message

## Install Maven

```bash
$ apt-get install maven
```

Verify with:

```bash
$ mvn --version
```

Key files for later:

```text
/usr/bin/mvn
/usr/share/maven
/etc/maven
```

## Edit Maven global settings to use a shared repository location for all users

```bash
$ vi /etc/maven/settings.xml
```
And then paste in the following:
```xml
<settings>
  ...
  <!-- The local repository -->
  <localRepository>/var/maven/repository</localRepository>
  ...
</settings>
```

Set up the repository directory structure

```bash
$ mkdir /var/maven/repository
$ chown -R ubuntu:ubuntu /var/maven/repository
```

## Install git

```bash
$ apt-get install git
```

## Verify that the project can be checked out and built by a local user

```bash
$ exit
$ cd ~
$ git clone https://github.com/example/example-dropwizard.git
$ cd example-dropwizard
$ git checkout example-branch
$ mvn clean install
```

## Verify project startup

```bash
$ java -jar target/example-dropwizard-version.jar server example.yml
```

Expect to see a clean startup with banner

CTRL+C to interrupt

## Prepare your server rollbacks

As root, set up the following directory structure

```bash
$ sudo -i
$ cd /var
$ mkdir git
$ mkdir www
$ mkdir /var/www/backups
$ mkdir /var/www/html
$ chown -R ubuntu:ubuntu /var/git
$ chown -R ubuntu:ubuntu /var/www
$ exit
```

## Prepare a bare git repo

As ubuntu, create a bare repo that will receive client pushes. It must be bare otherwise the remote refs will get conflicted and git will complain.

```bash
$ cd /var/git
$ git clone --bare https://github.com/example/example-dropwizard.git
```

## Prepare the post-receive git hook

The deployment magic happens using git hooks. In this case you need the post-receive hook, so do the following

```bash
$ cd /var/git/example-dropwizard/.git/hooks
$ vi post-receive
```

Populate it as follows:

```bash
#!/bin/bash
@echo off

# This script will live in git repo as .git/hooks/post-receive
targetdir=/var/git/example-dropwizard

# Target/working directory
cd $targetdir

# Stop the existing service (this may take some time so we do it before the Maven build)
echo Performing a soft kill of the existing process

pkill -f example-dropwizard-develop-SNAPSHOT

# Check out the local copy of the git repo
echo "Check out local copy"

export GIT_WORK_TREE=/var/git/example-dropwizard
export GIT_DIR=/var/git/example-dropwizard

cd $GIT_WORK_TREE

# The push will go from client 'develop' branch to server 'master' for convenience
git checkout -f

# Build with Maven
echo Maven build the new code
mvn clean package

# Start the new background process
echo Starting the new process under nohup

nohup java -jar target/example-dropwizard-develop-SNAPSHOT.jar server example.yml > /var/log/example/example-log.log 2>&1 &

echo Complete. Verify operation by visiting http://example.org/
```

Make sure that the script is executable with the proper ownership for the SSH login (usually ubuntu or ec2-user)

```bash
$ chmod +x post-receive
```

## Prepare your local SSH setup

This will ensure that git can find it for authentication later on.

Git may get confused when deciding which key to use, so use this "belt and braces" approach by creating (or updating) an SSH config file

You can infer the public HostName from the AWS console IP address even without an Elastic IP.

```bash
$ vi ~/.ssh/config
```

The contents should look like this:

```text
Host ExampleHost
HostName ec2-1-2-3-4.eu-west-1.compute.amazonaws.com
User ubuntu
IdentityFile ~/.ssh/examplekey.pem
IdentitiesOnly yes
```

## Add the remote repo to your local repo

Add a new remote repo called "production" to your local git repo based on the SSH path to the remote repo on the EC2 instance.

```bash
$ cd <path to local repo>/example-dropwizard/.git
$ git remote add production ExampleHost:/var/git/example-dropwizard
```

Manually review what git has done to be sure that there is no confusion with keys

```bash
$ cd .git
$ vi config
```

Add the following:

```text
[remote "production"]
        url = ExampleHost:/var/git/example-dropwizard
        fetch = +refs/heads/*:refs/remotes/production/*
```

To get the above to work, you'll need to have your examplekey.pem registered as ExampleHost with SSH (see earlier) or you'll get weird authentication errors. If you're really struggling you can attempt the SSH authentication manually with OpenSSL

```bash
$ ssh -v -i ~/.ssh/examplekey.pem ubuntu@1.2.3.4
```

This will shine a light on any problems that you'll otherwise be scratching your head over.

## Do the first push

If no-one has ever done this do an initial push of all data in the master branch (assuming that's where your production site content lives)

```bash
$ git push production +example-branch:refs/heads/master
```

For large repos over thin pipes this may take some time. You must have a bare remote repo (see earlier) otherwise git will starting whining at you. If you need to convert a repo back to a bare one then do the following on the server repo, but you'll lose any files checked out from the remote repo.

```bash
$ cd /var/git/example-dropwizard
$ mv .git .. && rm -rf *
$ mv ../.git .
$ mv .git/* .
$ rmdir .git
$ git config --bool core.bare true
```

## Link Dropwizard to nginx with additional static files

As root, do the following:

```bash
$ sudo -i
$ vi /etc/nginx/conf.d/example-dropwizard.conf
```

Add the following:

```text
# Site (port 80 -> 8080)
server {
  listen          80;       # Listen on port 80 for IPv4 requests
  server_name localhost;
  access_log      /var/log/nginx/site_access.log;
  error_log       /var/log/nginx/site_error.log;

  # Set the root of the static content
  root /var/www/html;

  # Redirect server error pages to the static page /50x.html
  error_page   500 502 503 504  /50x.html;
  location = /50x.html {
    root /var/www/html;
  }

  # Filter static content types and serve from the root
  location ~*\.(jpg|jpeg|gif|css|png|js|ico|html)$ {
    access_log off;
    expires max;
  }

  # Serve the dynamic content
  location / {
    # The application provides its own detailed logs
    access_log off;

    # Hand over to the application
    proxy_pass        http://localhost:8080/;
    proxy_set_header  Host             $http_host;
    proxy_set_header  X-Real-IP        $remote_addr;
    proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;

  }
}
```

By placing the file in `/etc/nginx/conf.d` with a `.conf` extension, nginx will automatically link to it. However you will probably have to disable the sites-enabled include in
`/etc/nginx/nginx.conf` to override the default server.

Restart nginx (as root) with

```bash
$ /etc/init.d/nginx restart
```

to get the following behaviour:

* / will serve the root of `example-dropwizard`
* Any static files are filtered on common extensions (`.jpg`,`.png` etc)
* Any other content is directed to the app serving on port 8080
* Any 50x error generated by the app will cause a redirect to the `/50x.html` static file

## The usual workflow

Make your changes locally, stage and commit as usual then use

```bash
$ git push production example-branch:master
```

This will cause the following behaviour

* The local `example-branch` branch will be pushed to the remote master branch
* The site will go offline almost immediately (not so good for high availability sites)
* A Maven build will take place and be seen in the console
* The site will be restarted as a `nohup` process

## Conclusion

So there you have it. A simple set of instructions to get your Dropwizard project fronted by nginx from scratch in just a few minutes.