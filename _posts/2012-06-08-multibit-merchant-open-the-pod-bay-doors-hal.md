---
author: Gary Rowe
title: MultiBit Merchant - "Open the pod bay doors, HAL"
layout: post
tags:
  - HAL
  - MBM
redirect_from: /agilestack/2012/06/08/multibit-merchant-open-the-pod-bay-doors-hal/
---

As part of my ongoing open source project [MultiBit Merchant (MBM)][2] I am keeping a journal of my discoveries and thoughts along the way. This one deals with why I chose the [Hypertext Application Language (HAL)][3] as part of the communication protocol for the MBM RESTful API.

### Making an API that will last decades

Regular readers will recall that as part of the [“genesis requirements”][4] I had to design MBM to provide a web service API in accordance with RESTful principles. One of the promises of a design like that is that you can create an API **that can last decades**. Just let that sink in for a minute. Decades. Nothing lasts for decades – except maybe that accounting system [written in COBOL][5] running on a VAX machine in the sub-sub-basement. Nobody ever goes down there anymore…

Except, actually, there are APIs that have been running for decades that you take for granted every day. I’m talking about HTML, and when [you look at its earliest history today][6], it still [renders just fine][7]. “But HTML isn’t an API! It’s a markup language!”, you cry. Nope, hypertext *is* an API because it offers the concept of a link which allows traversal of states. Thus, the web is just a giant state machine.

### The Richardson Maturity Model

Martin Fowler has a great article about web services that describes the [Richardson Maturity Model][8]. You should take the time to read it – perhaps over lunch.

In it, he describes how web services are evolving from the “Swamp of POX” to the “Glory of REST”. This journey to greater simplicity brings to mind a Zen quote I heard a long time ago that I would like to share with you:

> Before one studies Zen, mountains are mountains and waters are waters;  
> after a first glimpse into the truth of Zen, mountains are no longer mountains  
> and waters are no longer waters; after enlightenment,  
> mountains are once again mountains and waters once again waters.

You see, [REST][9] is simply what the web was originally intended to be, [as Ryan Tomayko explained to his wife][10] back in 2004. HTTP provides the verbs (`GET`,`POST`,`PUT` and so on), URIs provide the nouns (every resource has a unique name, or identifier) what is missing is how to put it all together.

### Hypertext as the engine of application state (HATEOAS)

Awful acronyms aside, the [HATEOAS principle][11] removes the coupling between the client and the server. Frequently, the first thing that developers new to web services attempt to make when creating an API is a unified URI structure. Often this simply mirrors one aspect of the internal business domain (perhaps `customer/1/order/23`). Unfortunately, restricting the URI structure in this manner causes problems later on when it becomes apparent that this structure has started to drive the API, and even the underlying domain, rather than merely being a resource identifier. For example, what if a particular use case requires that the internal domain should be order-driven with non-numeric order IDs making a URI of `order/EX1-45/customer/1` more appropriate?

URI structure is irrelevant to machines, and humans hardly ever need to work it out since they rely on presented links instead. It is that linking, and the semantics behind it, where HATEOAS really shines. By providing a set of semantic guidelines ([see RFC 5988][12]), often in the form of a `rel` attribute on a link structure, a resource can provide a client with API updates in terms of [link relations that it can work with][13].

### Splitting the Atom

For many, the hardest thing about REST is [working with representations][14]. In the Java world, JAX-RS provides an excellent set of annotations to describe RESTful endpoints. Implementations like [Jersey][15] (my preference) or [RESTEasy][16] work closely with JAXB (for XML or JSON) and provide you with a powerful and concise foundation. However, it is all too easy to use your existing domain objects as the response bodies which introduces the coupling mentioned earlier. Some kind of intermediate representation format is needed.

It comes down to a choice between several representation formats: [AtomPub][17] (complex IETF standard), [OData][18] (Microsoft oriented full ecosystem) and [HAL][3] (lightweight and new). Each has its merits and drawbacks, and I’ve looked in detail at each one so that I can present a summary of my findings:

#### AtomPub

* Pro: Well established standard
* Pro: Works with XML and JSON
* Pro: Has excellent Java support through [Apache Abdera][19]
* Con: Abdera introduces a lot of dependencies
* Con: Very complex to work with on server side
* Con: Difficult to build a complete JavaScript client

#### OData

* Pro: Builds on AtomPub
* Pro: Works with XML and JSON
* Pro: Has good Java support through the [odata4j][20] project
* Pro: Provides a good URI query structure
* Con: Introduces a complete framework (essentially replaces Dropwizard)
* Con: Very complex to work with, particularly the entity data model (EDM)
* Con: Difficult to find a good JavaScript client library without relying on Windows-only tools for EDM

#### HAL

* Pro: Introduces a lightweight and extensible approach
* Pro: Works with XML and JSON
* Pro: Trivial to create a JAXB model to implement it (no dependencies)
* Pro: Provides a good linking framework
* Pro: Trivial to create a JavaScript client using jQuery XML parsing
* Con: Not ratified (although IETF have been approached)

Looking at the above list it is pretty clear to me that HAL is the more appropriate choice for me at this time. It is lightweight, extensible and sufficient for the kind of data that I’ll be exposing as part of my project. If I was going for a much higher grade data offering, I may have settled on OData because of the excellent filtering structure it offers. In fact, I may just think about incorporating something similar into MBM.

So now that we’ve covered why, it is time to to cover how. Let’s open those pod bay doors.

 [1]: https://twitter.com/share
 [2]: https://github.com/gary-rowe/MultiBitMerchant
 [3]: http://stateless.co/hal_specification.html
 [4]: http://gary-rowe.com/agilestack/2012/06/06/multibit-merchant-genesis/
 [5]: http://www.codinghorror.com/blog/2009/08/cobol-everywhere-and-nowhere.html
 [6]: http://infomesh.net/html/history/early/
 [7]: http://www.w3.org/History/19921103-hypertext/hypertext/WWW/Link.html
 [8]: http://martinfowler.com/articles/richardsonMaturityModel.html
 [9]: http://en.wikipedia.org/wiki/Representational_State_Transfer
 [10]: http://tomayko.com/writings/rest-to-my-wife
 [11]: http://en.wikipedia.org/wiki/HATEOAS
 [12]: http://tools.ietf.org/html/rfc5988
 [13]: http://www.iana.org/assignments/link-relations/link-relations.xml
 [14]: http://nicksda.apotomo.de/2011/04/rails-misapprehensions-the-hardest-thing-about-rest-is-working-with-representations/
 [15]: http://jersey.java.net/
 [16]: http://www.jboss.org/resteasy
 [17]: http://tools.ietf.org/html/rfc5023
 [18]: http://www.odata.org/
 [19]: http://abdera.apache.org/
 [20]: http://code.google.com/p/odata4j/