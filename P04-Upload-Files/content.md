---
title: "Uploading Files to AWS S3"
slug: "uploading-files"
---

Uploading files and especially images is a very common pattern, but it is not that simple. Since Heroku and many server solutions do not allow you to store a lot of data in them, we need to use a third party service to save off any files or images we want to have access to later.

We'll be using **Amazon Web Services Simple Storage Service (AWS S3)** to save our images and files. This service's API will give us a unique URI we can use to retrieve the files later when we want to display them or download them.

# Make a Plan

In order to accomplish this pattern we have to take the following steps:

1. Get an AWS console account
1. Get our API public and secret keys and save them in our `.env` folder.
1. Make a new "bucket" in AWS S3
1. Change the form to use `multipart/form-data`
1. Add some middleware to accept `multipart/form-data`: `multer`
1. Add some middleware to interact with the S3 API: `s3-uploader`
1. Initialize and use the middleware in the controller route.
1. Save the image URL into the database
1. Display the image using the URL

# Sign Up for AWS

Navigate to the AWS Console and create an account.

1. Get an AWS console account
1. Get our API public and secret keys and save them in our `.env` folder.
1. Make a new "bucket" in AWS S3 - make sure this is in the N. California location.

# Multipart Form Data

Now let's set our form to submit `multipart/form-data` so we can submit a file along with other inputs.


```pug

```


So what does it mean to have our form send `multipart/form-data`?

*The following is from [this](https://stackoverflow.com/questions/4526273/what-does-enctype-multipart-form-data-mean) StackOverflow answer.*

> What does enctype='multipart/form-data' mean in an HTML form and when should we use it?

When you make a POST request, you have to encode the data that forms the body of the request in some way.

HTML forms provide three methods of encoding.

* application/x-www-form-urlencoded (the default)
* multipart/form-data
* text/plain

Work was being done on adding application/json, but that has been abandoned.

The specifics of the formats don't matter to most developers. The important points are:

When you are writing client-side code, all you need to know is use multipart/form-data when your form includes any <input type="file"> elements.

When you are writing server-side code: Use a prewritten form handling library and it will take care of the differences for you. Don't bother trying to parse the raw input received by the server.

*Never use text/plain.*

# Adding Middleware: s3-uploader

Now that we are submitting multipart/form-data we need to parse it using `multer` and then interact with the AWS api using `s3-uploader`. [s3-uploader docs](https://www.npmjs.com/package/s3-uploader)

Let's install the middleware:

```bash
$ npm install multer s3-uploader --save
```

Now let's load these, but no in our `app.js` file because we don't need it for the whole app, just in the specific route. So we'll load them right in the controller where we'll use it: `pets.js`.

```js
// pets.js

// UPLOADING TO AWS S3
const multer  = require('multer');
const upload = multer({ dest: 'uploads/' });
const Upload = require('s3-uploader');

```

Now we need to initialize and configure the `s3-uploader` object.

We want the configuration settings to:
1. Set the path in AWS to the bucket and with the access keys.
1. Clean up - when the upload is complete, we want to delete the originals and caches.
1. We want two versions: one a rectangle and one a square, neither wider than 300-400px.

```js
const client = new Upload(process.env.S3_BUCKET, {
  aws: {
    path: 'pets/avatar',
    region: process.env.S3_REGION,
    acl: 'public-read',
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
  },
  cleanup: {
    versions: true,
    original: true
  },
  versions: [{
    maxWidth: 400,
    aspect: '16:10',
    suffix: '-standard'
  },{
    maxWidth: 300,
    aspect: '1:1',
    suffix: '-square'
  }]
});
```

Finally we have to make some changes to our `POST` route to use `multer` and this `client` object we've just initialized and configured.

First we have to add `upload.single('avatar')` to the route itself.

```js
app.post('/pets', upload.single('avatar'), function (req, res) {
  console.log(req.file)
  ...
})
```

Now we can see that `req.file` is defined and coming in from our file input field from our form. Next we need to use `client` to save `req.file` to AWS and get back the URL our S3 bucket for the image.

After the pet is successfully saved, then we can save the image. We're going to get back both the version URL's but we just want the URL leaving off the `-standard` and `-square` because we can then save just one URL and when we call it we can add the `-standard` or `-square`. e.g.

```pug
img(src=pet.url + '-standard')
```

```js
app.post('/pets', upload.single('avatar'), function (req, res, next) {    
  var pet = new Pet(req.body);
  pet.save(function (err) {      
    if (req.file) {
      client.upload(req.file.path, {}, function (err, versions, meta) {
        if (err) { return res.status(400).send({ err: err }) };

        versions.forEach(function(image) {
          var urlArray = image.url.split('-');
          urlArray.pop();
          var url = urlArray.join('-');
          pet.avatarUrl = image.url;
          pet.save();
        });

        res.send({ pet: pet });
      });
    } else {
      res.send({ pet: pet });
    }
  }
})
```

# Displaying the Image

Now we need to display the images like this.

```pug
img(src=pet.url + '-standard')
```

Find all the places to replace the image and get-er done.

Yay! :D
