---
title: Learnings from Stripe integration for collecting payments 
layout: post
published: true
category: programming
tags: [ruby, stripe]
comments: true
---

When it comes to payment integration / collecting payment for any application everyone thinks of integrating with some third party service that would do the job for them instead of reinventing the wheel because there is very very high need of security when it comes to receiving payments.

[PayPal](https://www.paypal.com/){:target="_blank"} was the market leader in this space but it is quite complex in terms of managing the user account, recurring payments, shipping integration, tax integration etc. It is not that user friendly from a non-technical person's point of view.

This is where [Stripe](https://stripe.com/){:target="_blank"} comes into picture. It is far more user friendly, has a nicer user account's page, provides shipping & tax integration via external shipping and tax service providers. It also handles recurring payments without any additional complexity.

I integrated both PayPal & Stripe for one of my application and found out that Stripe is much more easier to integrate from a developer's perspective as well.

When you sign-up with Stripe, it also provides you guidance in test mode regarding what API calls from Stripe API to use using your test key, which you just need to enter into console to see it working and get a feel of it.

This was the most amazing thing for me to get me going. Just to make it clear once again, if you are logged-in into stripe and visit the getting started section in [Developer Documention](https://stripe.com/docs){:target="_blank"} then it will use your test API key to guide you through its various API calls.

Though integrating Stripe was very fun but there were also some areas where I found it to be slightly less obvious about some of its features that I got to know after lot of searching in the docs (which I wasn't able to find easily on googling). These are :

1. **Emails are to be turned ON seperately from the Account Settings section.**

2. **Emails are not sent for test mode.**

   **To view emails for test accounts go to :**

   **Dashboard > Payments > click on any payment > view receipt**

3. **Understand that you only need to use webhooks for behind-the-scenes transactions. The results of most Stripe requests—including charge attempts and customer creations—are reported synchronously to your code, and don't require webhooks for verification. (For example, with a charge request, the charge will immediately succeed and a Charge object returned, or the charge will immediately fail and an exception will be thrown.)**

Next, I want to highlight few points about shipping & tax integration.

For shipping & tax integration PayPal didn't provide any integration with other third party shipping services. Instead of re-inventing the wheel and create those services from scatch Stripe did a nice thing by providing seamless integration who have already specialized these fields.

**Shipping Integration :**

Stripe has partnered with [EasyPost](https://easypost.com/){:target="_blank"} and [Shippo](https://goshippo.com/){:target="_blank"} to help you get shipping rates directly from USPS, UPS, FedEx, and other carriers for your Stripe orders.

I integrated with EasyPost in my application and it was fairly easy. It was so seamless that I didn't even looked into Shippo.

To use EasyPost using Ruby, install the easypost gem using the command :

`gem install easypost`

Sample code for the EasyPost :

<script src="https://gist.github.com/Amit-Thawait/1d221dcf5e75de587674048e717228f9.js"></script>

You can inspect the `EasyPost::Shipment` object to check shipping options and rates.

**Tax Integration :**

For tax integration, Stripe has partnered with [Avalare](http://www.info.avalara.com/Stripe){:target="_blank"}, [TaxJar](https://taxjar.com/){:target="_blank"} and [Taxamo](https://taxamo.com/){:target="_blank"}. Since I was working a US based customer he suggested me to go integrate TaxJar since it is the most widely used in US.

The reason why we need to integrate with a third party service for tax calculation as well is : because in US tax rate is different is different states that too varies basing on the product category. Quoting the statement from Stripe's tax integration page :

> "Charging the legally appropriate amount of tax on orders is especially tricky for online sales. The right percentage to charge–if any–depends upon the customer's country or US state, the types of products being purchased, and the order total. To help businesses dynamically calculate and apply accurate taxes in real-time, Stripe has partnered with Avalara, TaxJar, and Taxamo."

To use TaxJar using Ruby, install the taxjar-ruby gem using the command :

`gem install taxjar-ruby`

Sample code for TaxJar integration :

<script src="https://gist.github.com/Amit-Thawait/0b5cfea4e5327bdb5fa0eb463eeca419.js"></script>

There are two kinds of thing that you can sell on stripe :

1. A physical thing that is shipped.

2. Subscriptions or SaaS (Software as a service) which can be charged mothly or yearly.

Few points related to tax integration that were no so obvious were :

1. **Tax rate for SaaS products are not governed automatically. You have to make a seperated API call to TaxJar by passing the `product_tax_code` and get the tax rate for that product. Product tax code for SaaS product is `30070` which you check from taxjar categories API call.**

	**Then you need to pass that tax_rate as `tax_percent` while creating a stripe customer record by calling `Stripe::Customer.create` call along with `plan` id which the user wants to purchase.**

	Ex:

	<script src="https://gist.github.com/Amit-Thawait/1ca6fcc8dd2fd507ab11fd389a2b4d45.js"></script>

2. **Now if you want to make an API call for purchasing a product you need to pass the `customer_id` generated from the above call as a parameter which make a stripe order create call.**

	For Ex :

	<script src="https://gist.github.com/Amit-Thawait/347169ca985a1daa724c2323df1939c1.js"></script>

	By attaching `customer_id` to this API call you will be able to see the details related to Subscription and Order both in the Customer details page in the Customer listing in stripe.

I hope these point are helpful in some or the other way. Thanks for reading till the end. :-)