
1. ~~Implement simple search on the store~~
1. ~~Build out pagination~~
1. ~~Implement validations, success, and error handling~~
1. **Uploading files**
    1. **Get an AWS console account**
    1. **Get our API public and secret keys and save them in our `.env` file.**
    1. **Make a new "bucket" in AWS S3**
    1. **Change the form to use `multipart/form-data`**
    1. **Add some middleware to accept `multipart/form-data`: `multer`**
    1. **Add some middleware to interact with the S3 API: `s3-uploader`**
    1. **Initialize and use the middleware in the controller route.**
    1. **Save the image URL into the database**
    1. **Display the image using the URL**
1. Integrating payment gateways
1. Sending emails
1. Building Full-text Search
1. Responding to JSON

Uploading files and especially images is a very common pattern, but it is not that simple. Since Heroku and many server solutions do not allow you to store a lot of data in them, we need to use a third party service to save off any files or images we want to have access to later.

We'll be using **Amazon Web Services Simple Storage Service (AWS S3)** to save our images and files. This service's API will give us a unique URI we can use to retrieve the files later when we want to display them or download them.

# Make a Plan

In order to accomplish this pattern we have to take the following steps:

1. Get an AWS console account
1. Get our API public and secret keys and save them in our `.env` file.
1. Make a new "bucket" in AWS S3
1. Change the form to use `multipart/form-data`
1. Add some middleware to accept `multipart/form-data`: `multer`
1. Add some middleware to interact with the S3 API
1. Initialize and use the middleware in the controller route.
1. Modify and standardize the shape and file type of the image.
1. Save the image URL into the database
1. Display the image using the URL

# Video Tutorial

How best to achieve these steps changes every year or two, so the most up-to-date tutorial for doing this is this excellent video:

[Storing Images in S3 from Node Server by Sam Meech-Ward](https://www.youtube.com/watch?v=eQAIojcArRY)

Follow the steps in this video before moving on.

# Product So Far

> Try uploading a new pet using an avatar! Make sure your old pets all still display too.

Next go check on your bucket to make sure your new pet images uploaded successfully!

> Go to [your bucket](https://console.aws.amazon.com/s3/home?region=us-west-1), click on the bucket name. There should now be a `pets` folder. Click on that and ensure that your images were uploaded successfully!

> You should notice that an `uploads/` folder gets created. **Make sure to add `uploads` to your `.gitignore` file so you don't push up all your uploaded images to GitHub.**

Congrats on getting AWS S3 up and running with your code! That's no small feat. Let's commit this!

# Now Commit

```bash
$ git add .
$ git commit -m 'Implemented S3 file uploads'
$ git push
```

# Stretch Challenges

1. You got this working for new pets, but what about if you *edit* a pet? Make sure you can edit a pet, and that the avatar is a valid field in the form.
1. Right now, _anyone_ who accesses your website can make a New Pet. How can you change your AWS policies to only let you (or other users you authorize) create new pets?
1. Following from the previous challenge, make the website user friendly so that unauthorized users who click on the New Pet button will not be allowed to navigate to the page, and will receive a notification stating why.
