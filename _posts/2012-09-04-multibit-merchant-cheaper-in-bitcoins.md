---
author: Gary Rowe
title: MultiBit Merchant - Cheaper in bitcoins
layout: post
tags:
  - Bitcoin
  - MBM
  - Principles
redirect_from: /agilestack/2012/09/04/multibit-merchant-cheaper-in-bitcoins/
---

As part of my ongoing open source project [MultiBit Merchant (MBM)][2] I am keeping a journal of my discoveries and thoughts along the way. This one deals with why Bitcoin offers merchants the highest profit margins for their sales by removing fraud from the payment process. 

### Fraud and its impact

Merchants operating online have to deal with fraud as part of their daily routine. This is because the current mechanisms for making payments are heavily biased to supporting the customer rather than the merchant. If a customer is unhappy with their purchase they are able to [issue a chargeback][3] and their payment processor (credit card, PayPal etc) will usually refund their money. This is hugely damaging to the merchant, and often requires them to take out chargeback insurance as an overhead cost.

For clarity, let’s walk through a typical situation that an online merchant faces when dealing with credit cards and chargebacks. This is the situation known as “cardholder not present” and means that the merchant is responsible for the loss.

Imagine a sales item costs £100, with a £22 margin. The credit card will typically charge 3.5% on the gross as a transaction fee (£3.50). Thus the net profit is £22.00 – £3.50 = £18.50.

Now let us say that the customer decides to hit the merchant with a chargeback. The credit card company will immediately take their side since it is a “cardholder not present” transaction and refund their £100. They will then go on to bill the merchant for this $100, and levy a chargeback fee of at least £25 to cover their administrative costs. (Merchants should note that some online sales are considered very high risk by credit card companies and chargeback fees of £100 and higher are not uncommon).

So, as a result of this chargeback the merchant has made a profit (£18.50) and incurred losses of (£100 + £25) making a net loss of £106.50. In order to recover from this the merchant will have to sell another 6 items (£106.50 / £18.50) so their maximum chargeback rate on sales is 16% (100 / 6). Any higher than that and they are going to go out of business. Repeated chargebacks will incur escalating fees and higher transaction fees thus reducing this maximum.

In the real world chargebacks occur at a rate of between 1% and 10% depending on the nature of the items. Clearly this can have a crippling effect on merchants offering these items for sale.

So what can be done? Enter Bitcoin.

### Irreversible transactions = much less fraud

One of the many advantages that Bitcoin brings to the table is the inherent fraud protection that comes from irreversible transactions. When a merchant receives a Bitcoin payment with at least 6 confirmations from the customer that transaction is final. The customer knows this and is therefore less likely to hand over payment if they intend to defraud. This is because they will have to rely on the legal system to get their money back in the case of a dispute.

This reliance on the legal system does not mean a court case. Most countries have the equivalent of the [Consumer Protection Regulations][4] which contains provisions for sales made without the customer being present. In the UK this means that anyone can return a product purchased via the Internet within 30 days so long as it meets certain criteria (not smashed up, worn out, etc). The merchant is obliged, under law, to accept the returned product and offer a refund. That said, there are subtleties and so interested readers are directed to read the linked Wikipedia article or to consult with a lawyer.

For the most part, therefore, customers dealing with a reputable company have nothing to fear when making a Bitcoin payment. If the goods arrive and are not what was expected then they can simply return them and get their refund from the merchant. This benefits both the customer and merchant since there are no financial penalties other than delivery costs which a merchant is obliged to bear under the regulations. In other countries, it is likely that the merchant would do this simply to retain the customer’s goodwill.

Any merchant working with Bitcoin should declare a “local currency” as part of their terms and conditions. This covers the volatility of Bitcoin over time. After all, a customer purchasing something worth £100 with Bitcoin at the start of the month would expect to get a refund worth £100 at the end of the month. This refund would be paid in the appropriate amount of bitcoins. This is the normal treatment for payments made in other currencies but is worth spelling out here.

### Less fraud = cheaper products

So the cost savings from using Bitcoin open the door to merchants being able to pass on these cost savings to their customers. In a highly competitive market (such as price comparisons on the Internet) any way that a company can reduce their offered prices and still retain a sufficient profit per sale means that they will benefit from increased volumes of sales. Therefore it is in the interests of the merchant to reduce their prices to attract this new custom.

MultiBit Merchant aims to make it trivial for anyone to set up in business online and to take payments for their goods and services thereafter. However, anyone wishing to do so should be aware of their obligations as a merchant to their customers, and how this is enforced under law.

 [1]: https://twitter.com/share
 [2]: https://github.com/gary-rowe/MultiBitMerchant
 [3]: http://en.wikipedia.org/wiki/Chargeback
 [4]: http://en.wikipedia.org/wiki/Consumer_Protection_(Distance_Selling)_Regulations_2000