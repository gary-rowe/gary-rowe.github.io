---
author: Gary Rowe
title: How to accept bitcoins on your blog with no code
layout: post
tags:
  - Bitcoin
  - Tutorial
  - HowTo
redirect_from: /agilestack/2012/01/09/how-to-accept-bitcoins-on-your-blog-with-no-code/
---

I’m often asked how bloggers and other online content producers can begin accepting bitcoins on their website, so I thought I’d quickly put together an article to save others the trouble.

### Why accept bitcoins?

The first question is why should a blogger want to use bitcoins in the first place? Well, most bloggers want to have their efforts rewarded in some way, and most resort to providing context sensitive advertising. The income from this kind of offering can be very low, especially if the blog is in a niche that does not get well served by ads. There is also a sense of being disconnected from one’s audience, which works both ways. When I appreciate someone”s work I want to donate directly to them and I don’t want some advertiser or payment processor taking a cut.

Bitcoin solves this problem, quickly and efficiently.

### Getting started

The quickest way is to offer a “Bitcoin QR code” (see the [MultiBit FAQ][2] for more information) as an image link with a Bitcoin URI and a suggested donation. For example, my blog [has one on the home page](http://gary-rowe.com) using the following HTML:

```html
<div>
  <a href="bitcoin:1KzTSfqjF2iKCduwz59nv2uqh1W2JsTxZH?amount=0.5&label=Agile%20Stack">
  <img src="http://gary-rowe.com/img/donation.png" ></a>
  <p>1KzTSfqjF2iKCduwz59nv2uqh1W2JsTxZH</p>
</div>
```

The Bitcoin QR code image was dragged out of the [MultiBit][5] “Receive Bitcoins” screen onto the desktop and then uploaded using the standard WordPress image import process. MultiBit is a free and open source Bitcoin client.

You’ll notice that the “href” attribute uses a different protocol than the usual “http”. If someone has installed a Bitcoin client onto their system then it will very likely be configured as a handler for that protocol. Clicking on that link will cause the appropriate application (or browser plugin) to pop up, usually in the “Send bitcoins” screen with the details provided already filled in. MultiBit does this from version 0.3 onwards.

It doesn’t matter which browser is being used because it is the operating system that manages protocol handlers. Developers interested in getting this kind of functionality to work in their own systems may want to look at [this Stack Exchange answer][6].

The QR component of the Bitcoin QR code allows people with smartphones to make donations using a Bitcoin wallet, or to use [drag and drop payment][7]. You’ll notice that I’ve also left the raw address visible. This is to allow people who do not have a suitable Bitcoin client to be able to copy paste the address into their respective client as a last resort.

### Adding a donation counter (optional)

Finally, if you’re accepting donations rather than selling a product, it might be useful to provide a “donated so far” label to give people an indication of your ongoing campaign. The [Block Explorer site][8] provides a wealth of useful information about Bitcoin addresses, and coupled with a [Text-to-Image web service][9] can give a nice result without any server-side processing.

For example, the following snippet (inspired by the [Bitcoin Trader][10] blog and offered up by [Jim Burton][11]) provides the amount sent to a given address.

```text
<img src="http://ansrv.com/png?s=http://blockexplorer.com/q/getreceivedbyaddress/1KzTSfqjF2iKCduwz59nv2uqh1W2JsTxZH&amp;c=000000&amp;b=FFFFFF&amp;size=5" />
```

as shown here:

![BTCs donated so far][12]

Why not see it change value by sending it a bitcoin? ;-)

By cycling your public addresses you can reset the amount as required and provide a label indicating the time span.

### Protect yourself – invest in HTTPS (it’s free!)

Remember that bitcoins have value, and like anything valuable they should be managed within a secure environment. As part of standard website security you should be aware that if you are running your blog over HTTP then anyone in control of any of servers between you and your reader is able to read, and modify, the contents of the traffic. This is known as a man-in-the-middle attack and there is [more detail available on the Security Stack Exchange site][13].

Since you are publishing your Bitcoin address then it is technically possible for someone to intercept this traffic and rewrite your Bitcoin address to their own. This is similar to someone forging a credit card payment form and stealing credit card numbers because the site owner didn’t use HTTPS. 

The simple fix for this is to invest in an HTTPS certificate for your website. This removes this possibility and promotes your site as a secure place to visit. You can obtain a [free SSL certificate][14] for this purpose from StartSSL. Alternatively, you can approach your website provider and ask them how to go about it.

### Known problems

It does appear that if you are using WordPress.com to host your blog, then it may not allow you to use to bitcoin: protocol and will change it to http: or omit the link altogether. If you encounter this problem then contact the WordPress.com staff and they may be able to help you. If you host your own version of WordPress, or run your blog on Blogger.com, then this is not an issue.

### Final word

So, if your blog offers content that is of real value to others then they can now show their appreciation by donating a small amount (less than a dollar) very easily. No fees. No registration. No hassle. Just you and them.

To all those who have donated to my blog – I thank you, personally.

 [1]: https://twitter.com/share
 [2]: https://github.com/Multibit-Legacy/multibit-website-legacy/blob/develop/src/main/resources/views/html/en/faq.html "MultiBit FAQ"
 [3]: bitcoin:1KzTSfqjF2iKCduwz59nv2uqh1W2JsTxZH?amount=0.5&label=Agile%20Stack
 [4]: view-source:http://gary-rowe.com/agilestack/wp-content/uploads/2011/10/AgileStack-0.5BTC.jpg
 [5]: https://github.com/Multibit-Legacy "MultiBit download"
 [6]: http://stackoverflow.com/a/8679698/396747 "MultiBit protocol handler code"
 [7]: http://www.youtube.com/watch?v=LlFPYBYIayU "MultiBit drag and drop payment with bitcoin QR code"
 [8]: http://blockexplorer.com "Block Explorer"
 [9]: http://ansrv.com/png/?help "Ansrv PNG service"
 [10]: http://www.thebitcointrader.com/ "Bitcoin Trader"
 [11]: https://plus.google.com/104492169332623586657 "Jim Burton on Google+"
 [12]: http://ansrv.com/png?s=http://blockexplorer.com/q/getreceivedbyaddress/1KzTSfqjF2iKCduwz59nv2uqh1W2JsTxZH&c=000000&b=FFFFFF&size=5 "Donated so far"
 [13]: http://security.stackexchange.com/questions/12041/are-man-in-the-middle-attacks-extremely-rare
 [14]: http://www.startssl.com/