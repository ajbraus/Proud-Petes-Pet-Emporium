---
title: "Getting Started: Reading Code"
slug: getting-started
---

This tutorial will walk you through adding the some common advanced features to an Express.js server, such as pagination, adding payment gateways, and uploading files using S3.

# Why Is This Important?

There's a lot of different concepts we're throwing at you in this tutorial. We want you to get practice at a base level with some of these concepts, as you'll absolutely be using some number of these (if not all of them!) throughout your career. If you work on anything payments related, you'll need to know how a payment gateway works. If you want to upload files in your web app at all (and who doesn't?), you'll need to know how to do that with some tool (in this case S3 and AWS). Do you want pages in your app at all?

This isn't everything you need to know for each topic, but it'll give you a foundation to build off of so that when you get assigned that payment flow, or file upload feature, you'll have a solid baseline to begin building from!

# Learning Outcomes

By the end of this tutorial, you should be able to add the following features to an Express.js server:

 * Simple Search
 * Pagination
 * Adding Validations
 * Error and success messages
 * Uploading images and files
 * Adding payment gateways like Stripe
 * Sending emails
 * Respond to JSON

# Technical Planning

Here's how we're going to go about building out features in this tutorial:

1. Implement simple search on the store
1. Build out pagination
1. Implement validations, success, and error handling
1. Uploading files
1. Integrating payment gateways
1. Sending emails
1. Building Full-text Search
1. Responding to JSON

# Outside In Philosophy & Vaporware

We're going to build this whole app from the outside-in, meaning each step, we're always going to build what the user sees (the **views** or **templates**) first. We can even populate these with some mock data using arrays. This first step is called building **Vaporware** and is an excellent pattern that will save you time and improve your client and stakeholder happiness.

> [info]
**Vaporware** is when you build out the templates with mock data, and it is a fantastic pattern to build navigable mock ups for clients and stakeholders before committing to building the logic and database relationships.

# Commits with Git

As you go through this tutorial, you will also be making commits after completing milestones. **This is a requirement, you must make a commit whenever the tutorial prompts you**. This not only further enforces best practices for software engineering, but also will help you more easily figure out where a bug originated from if you break your progress up into discrete, trackable chunks.

When prompted to commit, you'll see a sample commit message. Feel free to use your own message, so long as it clearly and concisely covers the work done.

Lastly, the commit prompts in this tutorial should be the **minimum** amount of times you commit. If you want to do more commits, breaking your chunks into even smaller chunks, that is totally fine!

# Getting Started - Cloning the Starter Project

You won't always be starting with a **Green Field Project** - a project where you start from scratch - sometimes you'll work on a project that has already started. In this tutorial we'll simulate that by starting with a relatively simple starter project.

> [action]
> Go to the [starter repo](https://github.com/Product-College-Labs/petes-pets) and clone the repo locally
>
```bash
$ git clone git@github.com:Make-School-Labs/petes-pets.git
```

Now we need to change the remote so that you can commit/push/pull the changes you make to your own repo. **It is very important you do the below steps in order to get everything working properly.**

> [action]
> Go to GitHub and create an _empty_, public repository called REPO-NAME, and now associate it as a remote for your cloned starter code, and then push to it.
>
```bash
$ cd petes-pets
# can grab the url from the "Clone or download" link on your repo page
$ git remote set-url origin git@github.com:YOUR_USERNAME/REPO-NAME
$ git push -u origin master
```

Go to your repo on GitHub and make sure your previously empty repo is now full with starter code! Now when you add/commit/push, it'll be to your repo!

Once you have finished the above, continue on to the next steps:

> [action]
> Install your dependencies
>
```bash
npm install
```

Next you need to make sure  MongoDB is properly installed. We will be using this for our database!
> [action]
>
> Follow the [install instructions](https://zellwk.com/blog/install-mongodb/). **Pay attention to the Preparations (MacOS Catalina onwards) section!!**
>
> You'll then need to run the following commands to make sure `mongod` is working correctly:
>
```bash
$ sudo mongod --dbpath /System/Volumes/Data/data/db
$ alias mongod="sudo mongod --dbpath /System/Volumes/Data/data/db"
```
> The alias is so that you don't have to type out the long command every time you want to start `mongod`

<!--  -->

> [info]
>
> Official instructions [here](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/), but we liked the above resource better since it specifices how to handle Catalina changes.

Now there is just one more step to seed your database before running your server.

> [action]
> Install `node-mongo-seeds`
>
```bash
$ npm install -g node-mongo-seeds
$ seed-setup
```
> Next open the `seed.js` file you just created, and update it to the following:
>
```js
module.exports = {
	"undefined": "localhost/local",
	"dev": "localhost/local",
	"prod": "localhost/local"
}
```
> This allows for the initial seed data to be found by `node-mongo-seeds`
>
> Finally, let's run the last few commands to seed our database with some starter  data, and then launch our localhost!
>
```bash
$ seed
$ nodemon
```

Now your browser at `localhost:3000` should look like this:

![petes-pets](assets/petes-pets.png)

You can also run `$ mocha` and see the tests run. They should all pass!

# Reading the Code Base

If you try to navigate around, you'll find that the behavior is kind of funky. You can create new pets and view them, but what if you try to edit a pet? Everything is back to the way it was before.

To understand what is going on, let's do what we have to do every time we start with a new code base. We have to *READ IT*.

Start by looking through all the code starting with the `server.js` file.

Here are some characteristics of this project:

* *1 Resource* - How many resources are there in this application? What are they? - There is 1 resource: Pets
* *Pug* - This project uses the templating engine called [Pug](https://pugjs.org/api/getting-started.html). Pug is an *HTML preprocessor* that simplifies HTML into a python-like syntax. Watch out for indentation!
* *Seeding the DB* - The `seeds` folder contains files that we can use to seed the database.
* `/bin/www` - notice that the server runs out of the `bin/www` file. Weird! But actually a common convention. We are pulling the concern of starting the server (`app.listen()`) into its own file (just like we pulled out routes, models, and views).

What other characteristics can you find?

> [action]
> Before moving on complete the following:
> Are there any middleware that you do not recognize in `server.js`? Google them and comment your code with what they do.

Before we move on, let's make our first commit:

>[action]
>
```bash
$ git add .
$ git commit -m 'cloned starter and added comments'
$ git push origin master -u
```
