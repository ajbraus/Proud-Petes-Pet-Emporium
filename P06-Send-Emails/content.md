---
title: "Send Emails"
slug: "send-emails"
---

1. ~~Implement simple search on the store~~
1. ~~Build out pagination~~
1. ~~Implement validations, success, and error handling~~
1. ~~Uploading files~~
1. ~~Integrating payment gateways~~
1. **Sending emails**
    1. **Install and configure the `nodemailer` and `nodemailer-mailgun-transport` modules.**
    1. **Sign up for an account with [Mailgun](https://www.mailgun.com)**
    1. **Add your email credentials to your `.env` file.**
    1. **Send a sample email**
    1. **Setup emails to send whenever a pet is sold.**
1. Building Full-text Search
1. Responding to JSON

Sending emails is a common requirement of any web service. And as you might expect, Express.js and Node.js makes it quite easy to send emails!

You might use a gmail account for a minimum viable product, but gmail will only let you send ~500 emails per day. At that point you'll want a scalable option. [Mailgun](https://www.mailgun.com/) to the rescue!

Then we'll need to extend `nodemailer` with the [nodemailer-mailgun-transport](https://github.com/orliesaurus/nodemailer-mailgun-transport) because `nodemailer-mailgun-transport` uses `consolidate.js` under the hood to add support for templating engines.

This means we can use handlebars templates for rendering our emails!

Lastly, we're going to try to make this code more modular. Rather than throwing it into our current routes or `server.js`, we'll make a new `mailer.js` file that will handle all mail related functions!

# Make a Plan

First off we'll make a plan of the outside-in, step-by-step process to get emails firing off.

1. Install and configure the `nodemailer` and `nodemailer-mailgun-transport` modules.
1. Sign up for an account with [Mailgun](https://www.mailgun.com)
1. Add your email credentials to your `.env` file.
1. Send a sample email
1. Setup emails to send whenever a pet is sold.

# Installing and Configuring nodemailer and nodemailer-mailgun-transport

Before we start integrating `nodemailer` into our app, let's do some simple tests with it in `server.js` to make sure the functionality is correct.

>[action]
> Install the `nodemailer` and `nodemailer-mailgun-transport` modules
>
```bash
$ npm install nodemailer nodemailer-mailgun-transport --save
```
>
> Now update `server.js` to include these modules:
>
```js
// server.js
...
// require our mailgun dependencies
const nodemailer = require('nodemailer');
const mg = require('nodemailer-mailgun-transport');
...
>
// auth with our mailgun API key and domain
const auth = {
  auth: {
    api_key: 'key-keyaldkjfadfasdfadsfadsf',
    domain: 'domain.com'
  }
}
>
// create a mailer
const nodemailerMailgun = nodemailer.createTransport(mg(auth));
```

# Make a Mailgun Account & Add Credentials

1. Head over to [Mailgun](https://www.mailgun.com) and create an account by clicking the **Start Sending** button
1. Fill out the form with your information. Don't worry about the credit card number, you won't go over 10,000 emails
1. Select **Flex Trial**, then click **Create Account**
![MG FLEX TRIAL](assets/mg-flex-trial.png)
1. Check your email and verify your account, go through the verification code process
1. From the Dashboard, you should see a **Getting Started** box. After you finish activating your account, click on **Add a custom domain**
![MG DASH DOMAIN](assets/mg-dash-domain.png)
1. Enter a unique domain name, leave other values as default and then click add domain **Add Domain**. Now you have a domain! *Don't worry about verifying your domain for now.*
![MG ADD DOMAIN](assets/mg-add-domain.png)
1. Finally, go back to the dashboard and scroll down. You should see an **API Keys** section at the right. Click on it, find the **Private API Key** and copy that.
![MG API KEY](assets/mg-api-key.png)

Now That you have both your **Private API key** and **domain**, let's add them to the project.

>[action]
> Add your Private API key and the  domain to your `.env` file. Replace the values with your actual MailGun API Key and Domain:
>
```
MAILGUN_API_KEY=mailgun-private-api-key
EMAIL_DOMAIN=mailgun-email-domain
```
>
> Now add those to your `auth` const in `server.js`

```js
const auth = {
  auth: {
    api_key: process.env.MAILGUN_API_KEY,
    domain: process.env.EMAIL_DOMAIN
  }
}
```

Now we are ready to try to send our first email.

# Sending Sample Email

Here's the code for sending an email with nodemailer transport:

> [action]
> Place this after your `nodemailerMailgun` declaration in `server.js`, remember to replace the `email` in `user` with your email so you can check it:
>
```js
// SEND EMAIL
const user = {
  email: 'YOUR@EMAIL.com',
  name: 'Emily',
  age: '43'
};
>
nodemailerMailgun.sendMail({
  from: 'no-reply@example.com',
  to: user.email, // An array if you have multiple recipients.
  subject: 'Hey you, awesome!',
  template: {
    name: 'email.handlebars',
    engine: 'handlebars',
    context: user
  }
}).then(info => {
  console.log('Response: ' + info);
}).catch(err => {
  console.log('Error: ' + err);
});
```

If you try and run that, it will probably throw an error because there is no `email.handlebars` template. Let's make that:

>[action]
> First install `handlebars`:
>
```bash
$ npm install handlebars express-handlebars --save
```
>
> Next in your root project folder, create an `email.handlebars` file and place the following code in it:
>
```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
</head>
<body>
    <div class="container">
        <p>Hello World!</p>
    </div>
</body>
</html>
```

<!-- -->

> [info]
> Reminder: since `email.handlebars` will not inherit from a layout template, you will need the full html boilerplate.

Run your code by starting `nodemon` up and see if it sent a "hello world" email to yourself.

Once emails are sending, see if you can display the `user` name and age with handlebars variables.

>[solution]
>
> Updates to `email.handlebars`:
>
```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
</head>
<body>
    <div class="container">
        <p>Hello {{name}}</p>
        <p>Your age is {{age}}</p>
    </div>
</body>
</html>
```

# Send an Email When Pets are Purchased.

Now we don't want to send emails just whenever we start our app! We want to send them when certain controller logic runs. For our Pet Store, there are a few places emails could go. When a pet is purchased, or when a pet is created. Let's do it as a notification when pets are purchased.

Move the email code so an email is sent to your email address whenever a pet is purchased.

**IMPORTANT NOTE:** For the purpose of this tutorial we're going to be using the same email we set for the user's Stripe email: `req.body.stripeEmail`. If you want to use a different email, you'll need to implement a solution different from the one below.

>[action]
> Create a new folder in the root of your project called `/utils`, and within it, a new file called `/utils/mailer.js`. Move the `const` declarations from `server.js` to the top of `/utils/mailer.js`:
>
```js
// mailer.js
>
// require our mailgun dependencies
const nodemailer = require('nodemailer');
const mg = require('nodemailer-mailgun-transport');
>
// auth with our mailgun API key and domain
const auth = {
  auth: {
    api_key: process.env.MAILGUN_API_KEY,
    domain: process.env.EMAIL_DOMAIN
  }
}
>
// create a mailer
const nodemailerMailgun = nodemailer.createTransport(mg(auth));
```
>
> Now move your `nodemailer` code from `server.js` to your `/utils/mailer.js` as well and export it as the `sendMail` function. You can remove the `user` const, as we'll be getting our user from the `/purchase` route:
>
```js
// export our send mail function
module.exports.sendMail = (user, req, res) => {
    // send an email to the user's email with a provided template
    nodemailerMailgun.sendMail({
        from: 'no-reply@example.com',
        to: user.email, // An array if you have multiple recipients.
        subject: 'Pet Purchased!',
        template: {
            name: 'email.handlebars',
            engine: 'handlebars',
            context: user
        }
    // One mail is sent, redirect to the purchased pet's page
    }).then(info => {
        console.log('Response: ' + info);
        res.redirect(`/pets/${req.params.id}`);
    // Catch error and redirect to the purchased pet's page
    }).catch(err => {
        console.log('Error: ' + err);
        res.redirect(`/pets/${req.params.id}`);
    });
}
```

Finally, we need to use this code in our `/purchase` route, so let's do that integration.

> [action]
>
> Update `/routes/pets.js` to include the `mailer` util and use it to send an email the purchase route
>
```js
// MODELS
const Pet = require('../models/pet');
>
const mailer = require('../utils/mailer');
>
...
>
// PURCHASE
  app.post('/pets/:id/purchase', (req, res) => {
    console.log(req.body);
    // Set your secret key: remember to change this to your live secret key in production
    // See your keys here: https://dashboard.stripe.com/account/apikeys
    var stripe = require("stripe")(process.env.PRIVATE_STRIPE_API_KEY);
>
    // Token is created using Checkout or Elements!
    // Get the payment token ID submitted by the form:
    const token = req.body.stripeToken; // Using Express
>
    // req.body.petId can become null through seeding,
    // this way we'll insure we use a non-null value
    let petId = req.body.petId || req.params.id;
>
    Pet.findById(petId).exec((err, pet) => {
      if(err) {
        console.log('Error: ' + err);
        res.redirect(`/pets/${req.params.id}`);
      }
      const charge = stripe.charges.create({
        amount: pet.price * 100,
        currency: 'usd',
        description: `Purchased ${pet.name}, ${pet.species}`,
        source: token,
      }).then((chg) => {
      // Convert the amount back to dollars for ease in displaying in the template
        const user = {
          email: req.body.stripeEmail,
          amount: chg.amount / 100,
          petName: pet.name
        };
        // Call our mail handler to manage sending emails
        mailer.sendMail(user, req, res);
      })
      .catch(err => {
        console.log('Error: ' + err);
      });
    })
  });
```
>
> Finally, change your `email.handlebars` to something more reflective of a pet purchase. The below is just an example, but feel free to get creative:
>
```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
</head>
<body>
    <div class="container">
        <p>Congrats on your new pet, {{petName}}!</p>
        <p>You Paid ${{amount}}</p>
        <p>Proud Pete's Pet Emporium thanks you!</p>
    </div>
</body>
</html>
```

# Product So Far

Now test this by purchasing a pet. Did you get an email?

Great work! Let's commit and move forward!

# Now Commit

```bash
$ git add .
$ git commit -m 'Implemented Emails'
$ git push
```

# Stretch Challenge

> [challenge]
>
> Update the purchase code in the `/purchase` route to be modular like `mailer`
