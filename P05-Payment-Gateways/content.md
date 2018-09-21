---
title: "Payment Gateways: Stripe"
slug: "payment-gateways"
---

Everybody, eventually, has got to get paid. And the web is no different.

Generally online payments are conducted using credit and debit cards. More recently people have learned to accept cryptocurrencies as well, but accepting crypto is for a future lesson if and when these electronic currencies become more widely used.

There are laws around saving credit card information, called **PCI Compliance**. PCI Compliance means having various standards of encryption, security, and controls around a database system. Most companies do not want to maintain their own PCI Compliance, so they outsource it to services called **Payment Processors** like stripe.com and braintreepayments.com.

In this tutorial we'll be using Stripe to process payments to buy some pets.

# Make A Plan

1. Sign up for Stripe and get our public/private access keys
1. Add the drop in stripe checkout
1. Add `stripe` middleware for interacting with the stripe API.
1. Process the payment on the server
1. Save when the pet was purchased and by whom

# Stripe

Head over to [Stripe's Payments](https://stripe.com/us/payments) page and create an account.

# Drop In Stripe Checkout

To simplify this implementation, we are going to use Stripe's "Checkout" product that creates a high quality checkout experience with very little code.

The Checkout code will take the user's credit/debit card information.

# Add Stripe Middleware



# Process the Payment on the Server



# Save Who Bought the Pet
