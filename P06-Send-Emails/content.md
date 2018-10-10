---
title: "Send Emails"
slug: "send-emails"
---

Sending emails is a common requirement of any web service. And as you might expect, Express.js and Node.js makes it quite easy to send emails!

You might use a gmail account for a minimum viable product, but gmail will only let you send ~500 emails per day. At that point you'll want a scalable option. [Mailgun](https://www.mailgun.com/) to the rescue.

Then we'll need to extend `nodemailer` with the `nodemailer-mailgun-transport` [docs](https://github.com/orliesaurus/nodemailer-mailgun-transport) to because `nodemailer-mailgun-transport` uses `consolidate.js` under the hood to add support for templating engines. That way we can use handlebars templates for rendering our emails.

# Make a Plan

First off we'll make a plan of the outside-in, step-by-step process to get emails firing off.

1. Install and configure the `nodemailer` and `nodemailer-mailgun-transport` modules.
1. Sign up for an account with [Mailgun](https://www.mailgun.com)
1. Add your email credentials to your `.env` file.
1. Send a sample email
1. Setup emails to send whenever a pet is sold.

```js
// project/app.js
...
const nodemailer = require('nodemailer');
const mg = require('nodemailer-mailgun-transport');
...

const auth = {
  auth: {
    api_key: 'key-keyaldkjfadfasdfadsfadsf',
    domain: 'domain.com'
  }
}

const nodemailerMailgun = nodemailer.createTransport(mg(auth));
```

# Make a Mailgun Account & Adding Credentials

Head over to [Mailgun](https://wwww.mailgun.com) and create an account. Now find your API key for your sandbox domain.

Add your sandbox API key and the sandbox domain to your `.env` file and add those to your initialization of nodemailer transport. e.g.

```js
{
  api_key: process.env.MAILGUN_API_KEY,
  domain: process.env.EMAIL_DOMAIN  
}
```

Now we are ready to try to send our first email.

# Sending Sample Email

Here's the code for sending an email with nodemailer transport:

```js
// SEND EMAIL
const user = {
  email: 'YOUR@EMAIL.com',
  name: 'Emily',
  age: '43'
};

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

If you try and run that, it will probably throw an error because there is no `email.handlebars` template. Go ahead and make that.

> [info]
> Reminder: since `email.handlebars` will not inherit from a layout template, you will need the full html boilerplate.

Can you send a "hello world" email to yourself?

Once emails are sending, see if you can display the `user` name and age with handlebars variables.

# Send an Email When Pets are Purchased.

Now we don't want to send emails just whenever we send our app! We want to send them when certain controller logic runs. For our Pet Store, there are a few places emails could go. When a pet is purchased, or when a pet is created. Let's do it as a notification when pets are purchased.

Move the email code so an email is sent to your email address whenever a pet is purchased.

Now test this by purchasing a pet.
