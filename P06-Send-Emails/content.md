---
title: "Send Emails"
slug: "send-emails"
---

Sending emails is a common requirement of any web service. And as you might expect, Express.js and Node.js makes it quite easy to send emails!

# Make a Plan

First off we'll make a plan of the outside-in, step-by-step process to get emails firing off.

1. Install and configure the `express-mailer` module
1. Make a new gmail account
1. Add your email credentials to your `.env` file
1. Send a sample email
1. Switch to sendgrid to send emails

# Express Mailer

Express Mailer is a wrapper for a lower-level module called `nodemailer`

First install express-mailer

```bash
$ npm install express-mailer --save
```

Next configure `express-mailer` in your `app.js` file.

```js
// project/app.js

var app = require('express')(),
    mailer = require('express-mailer');

mailer.extend(app, {
  from: 'no-reply@example.com',
  host: 'smtp.gmail.com', // hostname
  secureConnection: true, // use SSL
  port: 465, // port for secure SMTP
  transportMethod: 'SMTP', // default is SMTP. Accepts anything that nodemailer accepts
  auth: {
    user: 'gmail.user@gmail.com',
    pass: 'userpass'
  }
});
```

# Make a Gmail Account & Adding Credentials



# Sending Sample Email

```js
app.get('/', function (req, res, next) {
  app.mailer.send('email', {
    to: 'example@example.com', // REQUIRED. This can be a comma delimited string just like a normal email to field.
    subject: 'Test Email', // REQUIRED.
    otherProperty: 'Other Property' // All additional properties are also passed to the template as local variables.
  }, function (err) {
    if (err) {
      // handle error
      console.log(err);
      res.send('There was an error sending the email');
      return;
    }
    res.send('Email Sent');
  });
});
```

# Scalable Alternative: MailGun

You might use a gmail account for a minimum viable product, but gmail will only let you send ~500 emails per day. At that point you'll want a scalable option. [Mailgun](https://www.mailgun.com/) to the rescue.

`nodemailer-mailgun-transport` uses `consolidate.js` under the hood to add support for templating engines.


```js
var nodemailer = require('nodemailer');
var mg = require('nodemailer-mailgun-transport');

// This is your API key that you retrieve from www.mailgun.com/cp (free up to 10K monthly emails)
var auth = {
  auth: {
    api_key: 'key-1234123412341234',
    domain: 'one of your domain names listed at your https://mailgun.com/app/domains'
  },
  proxy: 'http://user:pass@localhost:8080' // optional proxy, default is false
}

var nodemailerMailgun = nodemailer.createTransport(mg(auth));


// SEND EMAIL
var user = {
  email: 'blah@gmail.com',
  age: '43'
};

nodemailerMailgun.sendMail({
  from: 'myemail@example.com',
  to: 'recipient@domain.com', // An array if you have multiple recipients.
  subject: 'Hey you, awesome!',
  template: {
    name: 'email.hbs',
    engine: 'handlebars',
    context: user
  }
}).then(info => {
  console.log('Response: ' + info);
}).catch(err => {
  console.log('Error: ' + err);
});
```

In order to use mailgun we have to unwrap and use `nodemailer` without `express-mailer`. Then we'll need to extend `nodemailer` with the `nodemailer-mailgun-transport` [docs](https://github.com/orliesaurus/nodemailer-mailgun-transport) to be able to use our templating engine.
