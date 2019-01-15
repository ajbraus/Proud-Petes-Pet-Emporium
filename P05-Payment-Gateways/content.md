---
title: "Payment Gateways: Stripe"
slug: "payment-gateways"
---

Everybody, eventually, has got to get paid. And the web is no different.

Generally online payments are conducted using credit and debit cards. More recently people have learned to accept cryptocurrencies as well, but accepting crypto is for a future lesson if and when these electronic currencies become more widely used.

There are laws around saving credit card information, called **PCI Compliance**. PCI Compliance means having various standards of encryption, security, and controls around a database system. Most companies do not want to maintain their own PCI Compliance, so they outsource it to services called **Payment Processors** like [Stripe](https://stripe.com/) and [Braintree](https://www.braintreepayments.com/).

In this tutorial we'll be using Stripe to process payments to buy some pets.

# Make A Plan

1. Sign up for Stripe and get our public/private access keys
1. Add the drop in stripe checkout
1. Add `stripe` middleware for interacting with the stripe API.
1. Process the payment on the server
1. Save when the pet was purchased and by whom

# Stripe Signup

1. Head over to [Stripe's registration page](https://dashboard.stripe.com/register) and fill out the information
1. Select **Integrate with Stripe's API** (we're developers after all)

Alright, we're registered!

## Public/Private API Keys

Now we need to get our Stripe public/private API keys.

From your dashboard (where you should have landed after the previous section), click on **Get your API Keys**

Here you can see your API keys:

* **Publishable key:** This is your public API key
* **Secret key:** This is your private API key

Keep this tab handy, as we'll need these keys later in this chapter.

# Drop In Stripe Checkout

To simplify this implementation, we are going to use Stripe's "Checkout" product that creates a high quality checkout experience with very little code.

Check out [Stripe's Card Payment Quickstart Docs](https://stripe.com/docs/quickstart) to get an idea of what we'll be getting into.

The Checkout code will take the user's credit/debit card information. It won't actually complete the payment though. It will just send that information to Stripe and get a token back.  To complete the payment, we'll have to send the token the Checkout code generates to our  server to actually complete the payment. Making a payment is called a `charge` in Stripe's **DSL (Domain Specific Language)**.

> [action]
> Put this `form` in the `.col-sm-8` element in `/views/pets-show.pug`:
>
```pug
.col-sm-8
>
...
>
  form(action='your-server-side-code', method='POST')
    script.stripe-button(src='https://checkout.stripe.com/checkout.js',
    data-key='pk_test_TYooMQauvdEDq54NiTphI7jx',
    data-amount='999',
    data-name='Stripe.com',
    data-description='Widget',
    data-image='https://stripe.com/img/documentation/checkout/marketplace.png',
    data-locale='auto',
    data-zip-code='true')
...
```

Reload, and click the button. We get a nice widget model asking for a bunch of information! And all we had to do was drop in a form!

This is just a test form though, so let's update it so that it works with our server.

# Customizing the Checkout Widget

What should replace `/your-server-side-code`? What should we name that route? This would not be a strict RESTful and Resourceful route, but we should do our best to follow these conventions.

Let's use `POST` verb, but we're creating something associated with a Pet. So probably our best bet is `/pets/:id/purchase`.

>[action]
> Update the `action` on the purchase form in `/views/pet-show.pug` to the following:
>
```pug
form(action=`/pets/${pet._id}/purchase`, method='POST')
```
>
> Next, create a `purchase` route in `/routes/pets.js`:
>
```js
app.post('/pets/:id/purchase', (req,res) => {
  console.log(`purchase body:${req.body}`);
});
```

Now add your public API key. You can do this by setting it in your `.env` file under `PUBLIC_STRIPE_API_KEY`, and then use `app.locals` to pass it forward into your template.

>[action]
> Add your public stripe api key to `.env`, replacing `public-api-key` with your actual **publishable key** from your Stripe dashboard:
>
```
PUBLIC_STRIPE_API_KEY=public-api-key
```
>
> Next add the following line in `server.js`:
>
```js
// server.js
app.locals.PUBLIC_STRIPE_API_KEY = process.env.PUBLIC_STRIPE_API_KEY
```
>
>Finally, update your `data-key` in the purchase form in `/views/pets-show.pug` to the following:
>
```pug
data-key=PUBLIC_STRIPE_API_KEY
```

Is the amount ok? Let's just leave `999` in the `data-amount` in there for now. Notice that it is in cents?

> [action]
Change the `data-name` in the purchase form in `/views/pets-show.pug` to Proud Pete's Pet Emporium:
>
```pug
data-name="Proud Pete's Pet Emporium"
```

# Test Checkout

Now you can submit the test credit card number `4242 4242 4242 4242`. For expiration date, you can submit any valid date in the future, any random number for the CVC number, and any 5-digit number for the zip code. You can get more from [Stripe's test cards doc](https://stripe.com/docs/testing#cards).

Remember we made our purchase endpoint log the request's body to the console to see what we are getting back. Make sure you're getting that log when you submit a purchase.

# Add Stripe Middleware & Process the Payment on the Server

To complete a charge you need to send two tokens to the Stripe API:

1. A valid **Checkout token** (generated by Checkout on front end)
1. **Private Stripe API Key** (like your app's Stripe password)

With these two pieces of info we can verify a charge in the Stripe system and get the money!

So we already have the Checkout token being sent to the `/pets/:id/purchase` route (check your console log to see for yourself). But we need to add the Private Stripe API key.

>[action]
> Add your Private API Key in your `.env` file, replacing the value with your actual Private API Key:
>
```
PRIVATE_STRIPE_API_KEY=sk_test_4eC39HqLyjWDarjtT1zdp7dc
```

So now we need to initiate a charge using the Stripe API. To do this we'll use the [stripe](https://www.npmjs.com/package/stripe) API wrapper called `stripe` (maintained by Stripe).

>[action]
> Update your `/pets/:id/purchase` route in `/routes/pets.js` to the following. Be sure to customize the Stripe Secret API Key to **your** key value:
>
```js
// PURCHASE
app.post('/pets/:id/purchase', (req, res) => {
  console.log(req.body);
  // Set your secret key: remember to change this to your live secret key in production
  // See your keys here: https://dashboard.stripe.com/account/apikeys
  var stripe = require("stripe")("sk_test_4eC39HqLyjWDarjtT1zdp7dc");
>
  // Token is created using Checkout or Elements!
  // Get the payment token ID submitted by the form:
  const token = req.body.stripeToken; // Using Express
>
  const charge = stripe.charges.create({
    amount: 999,
    currency: 'usd',
    description: 'Example charge',
    source: token,
  }).then(() => {
    res.redirect(`/pets/${req.params.id}`);
  });
});
```

# Testing A Purchase

Once your route is wired up, try submitting a payment now and make sure it runs successfully and that you're redirected to the `/pets/show` page.

Now check your [stripe payments](https://dashboard.stripe.com/test/payments) to see if the payment went through!

# Now Commit

```bash
git add .
git commit -m 'Implemented Stripe Payments'
git push
```

# Further Customization: Price & Description

It is clear now that the Pet model needs another attribute: a price. Add this as an integer called `price`. Can you set it in the seed.js and rerun it to see the database.

Also update the new and edit form to include `price`.

Now customize the `/pets/:id/purchase` to include the price. **Reminder** - multiply the price by 100 to get the price in cents.

Make the description the Pet's name and species.

# Save Who Bought the Pet

Now that the pet is saved, we better mark that the pet was saved by setting the current date and time on a new pet attribute: `purchased_at`.

Now mark all pets that are purchased as "Purchased!" in green text (using boostrap's `text-success` class).

Also if a pet is purchased, do not show the Checkout button.
