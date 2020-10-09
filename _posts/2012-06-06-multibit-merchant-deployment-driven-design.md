---
author: Gary Rowe
title: MultiBit Merchant - Deployment Driven Design?
layout: post
tags:
  - Dropwizard
  - MBM
redirect_from: /agilestack/2012/06/06/multibit-merchant-deployment-driven-design/
---

As part of my ongoing open source project [MultiBit Merchant (MBM)][2] I am keeping a journal of my discoveries and thoughts along the way. This one deals with why I chose [Dropwizard][3] as part of the design for MBM.

### A wake up call for application containers

Does deployment drive design? Given that most Java web applications require an application container like JBoss or Websphere and so on, does this mean that the application design is fixed? Is `WEB-INF/web.xml` all there is? I say no. In fact, I think that sticking with application containers right now is making a very serious long term mistake.

In the interests of brevity, let me [hand over to Mike Gualtieri of Forrester][4] who expresses all my concerns about application containers almost exactly as I would. I suggest that anyone interested in knowing why application containers are doomed should read it. The final item on the list nails it for me and serves as a TL;DR:

> “…I question even the need for a container. Seriously, why can’t Java web applications just run on the operating system like the containerless Microsoft .NET applications? The answer is: Then we would not be able to sell hundreds of millions of dollars in application servers. Remember when Netscape sold Web servers for $50,000 a pop? No one buys a Web server now. I think the [application server bubble is about to burst.][5]“

### Elasticity is the key

Any application that intends to be successful at a global scale will need to work within an architecture that can scale elastically. This means that at different times of the day, different parts of the application will be under higher or lower load than others. In order to optimise server running costs, this implies that it should not require operator intervention to switch on and configure new servers to meet this demand. The operating environment of the application should perform this function itself – this is the notion of application self-sufficiency.

Elastic compute clouds (such as [Amazon EC2][6] and [Heroku][7]) provide tools that allow this to happen, but it’s no good having to deploy a complete instance of JBoss followed by your web application every time a new server comes online. The complexities involved in that are considerable, and largely unnecessary.

A better approach would involve a collection of lightweight stateless web applications that run up in their own JVMs. Since each part is stateless it is horizontally scalable. Need more web request processing? Just fire up more web processing instances. Problem with data throughput to the database? Add more servers to the persistence processing instances. Database playing up? Introduce a NewSQL distributed database like [VoltDB][8] or go for NoSQL with [Neo4J][9] so that the database can grow.

So how can a typical Java web application be re-architected into this new world? Surely the grip of the application containers is unbreakable? In the words of Kryten from Red Dwarf: Not so.

### Enter the [Dropwizard][10]

The Dropwizard project provides an out of the box lightweight environment for creating standalone Java web applications. It uses the ultra-lightweight Jetty servlet container to manage the HTTP traffic and leaves the rest to a selection of chosen supporting technologies. By using Maven to include Dropwizard you get a preconfigured environment consisting of:

* Jetty for HTTP
* Jersey for REST
* Jackson for JSON
* Metrics for metrics
* Guava, Logback, SLF4J, Hibernate Validator

That’s a pretty good list, and almost exactly what I would have chosen myself.

More to the point, how does using Dropwizard help me to meet my [“genesis requirements”][11]?

Clearly, it promotes a RESTful architecture within an open source project based on Java. A more subtle point is that it enables the MBM application to run standalone if necessary. No need for the user to install a complex web application server to get going. Just a simple:

`java -jar mbm-1.0.0.jar server production.yml`

and they’re able to start serving. How they reach the wider web is another question, but it is entirely possible with a combination of [DynDNS][12] and a cable ISP to run this from home. Who knows, [one day IPv6 may be replaced][13] and then we’ll all be our own ISP.

### Where next?

Now that the deployment framework is in place, it is time to consider how clients will communicate with MBM. You can read more about my progress by following the articles labelled with the “MBM” tag. Or [jump straight to the next article now][14].

 [1]: https://twitter.com/share
 [2]: https://github.com/gary-rowe/MultiBitMerchant
 [3]: http://dropwizard.codahale.com
 [4]: http://blogs.forrester.com/mike_gualtieri/11-07-15-stop_wasting_money_on_weblogic_websphere_and_jboss_application_servers
 [5]: http://blogs.forrester.com/mike_gualtieri/11-04-27-the_application_server_bubble_is_about_to_burst
 [6]: http://aws.amazon.com/ec2/
 [7]: http://www.heroku.com/
 [8]: http://voltdb.com/
 [9]: http://neo4j.org/
 [10]: http://gunshowcomic.com/316
 [11]: http://gary-rowe.com/agilestack/2012/06/06/multibit-merchant-genesis/
 [12]: http://dyn.com/dns/dyndns-pro/
 [13]: https://github.com/cjdelisle/cjdns/blob/master/rfcs/Whitepaper.md
 [14]: http://gary-rowe.com/agilestack/2012/06/08/multibit-merchant-open-the-pod-bay-doors-hal/