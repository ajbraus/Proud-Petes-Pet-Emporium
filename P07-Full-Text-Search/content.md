---
title: "Full Text Search"
slug: "full-text-search"
---

Mongo ships with native [Full Text Search](https://en.wikipedia.org/wiki/Full-text_search). All you have to do is add an **index** on the attributes you want to search on. You can even add **weights** to the various attributes, if you'd like to have text in the title to be worth more in the search than text in the content.

# Add an index

Let's start with an unweighted full text search index. **Unweighted** means that if a word or piece of a word is found in any of the attributes, it will not *"weight"* the results towards one attribute. For example, if a movie website ran a search where search term is "Austin", the index will return anything with the word "Austin" in any attribute (title, location, director, actor, etc.). If we weighted our index towards the `title` attribute, then probably "Austin Powers" would be at the top, because the search term is in the title.

## Remove Existing Indexes
Before we add an index to our model, let's make sure we're clear of indexes (since we did fork this repo):

>[action]
> Make sure `nodemon` isn't running, and then clear the pre-existing index from your db. Run each of the comments sequentially:
>
```bash
$ mongo
$ use petes-pets
$ db.pets.dropIndex('animal_text_color_text_pattern_text_size_text');
```

If you run into a Mongo error where it says the index "already exists with different options", you will have to run this and delete the index that's causing problems before assigning a new one.

## Add Your Own Index

Now you can add your own index to your `pet` model!

>[action]
> Pick one of the following models to add to `/models/pet.js`, or edit them to suit what you want:
>
```js
...
>
// without weights
PetSchema.index({ name: 'text', species: 'text', favoriteFood: 'text', description: 'text' });
>
// with weights
PetSchema.index({ name: 'text', species: 'text', favoriteFood: 'text', description: 'text' }, {name: 'My text index', weights: {name: 10, species: 4, favoriteFood: 2, description: 1}});
```

# Update your Query

Now to call the search, use the `$text` option. The `$meta: 'textScore'` tells Mongoose to use the weights.

>[action]
> Replace your `/search` route in `/routes/pets.js` with the following:
>
```js
// SEARCH
app.get('/search', function (req, res) {
  Pet
      .find(
          { $text : { $search : req.query.term } },
          { score : { $meta: "textScore" } }
      )
      .sort({ score : { $meta : 'textScore' } })
      .limit(20)
      .exec(function(err, pets) {
        if (err) { return res.status(400).send(err) }
>
        if (req.header('Content-Type') == 'application/json') {
          return res.json({ pets: pets });
        } else {
          return res.render('pets-search', { pets: pets, term: req.query.term });
        }
      });
});
```

# Now Test!

Now test your search and see if it is working the way you'd like. Remember we're **not using Regex in the code above, so we will no longer match partial strings**

# Now Commit

```bash
$ git add .
$ git commit -m 'Implemented Full Text Search'
$ git push
```
