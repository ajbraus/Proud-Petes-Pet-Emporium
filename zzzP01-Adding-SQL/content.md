---
title: "Adding SQL to Express.js"
slug: "adding-sql"
---

So we have this vaporware app with three resources. Now we need to swap out that vaporware with a *data layer*.

# Adding PostgreSQL to your Machine

We'll be using SQL - pronounced either _seequel_ or _es-queue-el_ - as our pet store's database. SQL is a very popular database that is based on tables and relationships.

> [info]
> For more information watch this great ~1 hr video explaining and demonstrating SQL [SQL for Beginners. Learn Basics of SQL in 1 Hour](https://www.youtube.com/watch?v=7Vtl2WggqOg)

In order to add it to our project we are going to start with our primary resource: Pets.

We'll be using PostgreSQL as our local and remote SQL database (on heroku). So first you must install PostgreSQL on your machine:

```bash
$ brew install postgres
```

Now download and install [Postico](https://eggerapps.at/postico/) in order to have a GUI to interact with your local postgres databases.

Now download and install the [Postgres.app](https://postgresapp.com/documentation/gui-tools.html) in order to turn on your postgres database on your machine.

# Your ORM Sequelize.js

We are using SQL, but we are going to make things a lot simpler and more abstract by using an ORM *Object Relationship Mapper*.

Sequelize.js is the most popular SQL ORM *Object Relationship Mapper* that is available. That does not make it the best. It is quite complex and its documentation is very inadequate and can be very confusing. We'll try here to simplify and explain how to use it, but prepare yourself for some major debugging.

Start by adding the Sequelize.js command line interface

```bash
$ npm install -g sequelize-cli
```

Now add `sequelize`, `pg`, and `pg-hstore` modules to your project to use sequelize.

```bash
$ npm install --save sequelize pg pg-hstore
```

We'll also create a `.sequelizerc` file - a sequelize configuration file. We'll use this file to set the paths to all our database/orm related folders and files into a `db` folder.

```bash
$ touch .sequelizerc
```

```js
const path = require('path');
module.exports = {
  "config": path.resolve('./db/config', 'config.json'),
  "models-path": path.resolve('./db/models'),
  "seeders-path": path.resolve('./db/seeders'),
  "migrations-path": path.resolve('./db/migrations')
};
```

Alright now we're ready to initialize sequelize

```bash
$ sequelize init
```

See that this created a `db` folder with four folders inside.

```
db
|- config
  |- config.json
|- migrations
|- models
|- seeders
```

We will be using only the `migration` and `models` folders. Leave seeders and the config alone for now.

Next let's create a `Pet` model. To create new models we are going to use the `sequelize-cli` package we added before in order to save time from writing a lot of boilerplate code. We'll start with adding a `Pet` model with just two attributes, and add more later to the *migration*.

```bash
$ sequelize model:create --name Pet --attributes name:string,species:string
```

`sequelize model:create` creates a model, but also a *migration* file.

> [info]
> A SQL database has a defined structure that must be programmed into it before any data can be saved or retrieved from it. A *Migration* is how a software developer (that's you!) updates the structure of a SQL database. Inside a *Migration* you can add, update or delete tables, columns, and indexes.

You can see that this command created a file `pet.js` in the `models` folder and the file `20180516221256-create-pet.js` in the `migrations` folder. Migration file names are prefaced a timestamp so the server knows which ones are newer and older and which have already been run. You'll understand better once we start running them.

Finally let's create the database:

```bash
$ createdb petes-pets-development
```
