---
title: "Full Text Search"
slug: "full-text-search"
---

Mongo ships with native **Full Text Search**. All you have to do is add an index on the attributes you want to search on. You can even add **weights** to the various attributes, if you'd like to have text in the title to be worth more in the search than text in the content.

```js
// your-model.js

// without weights
schema.index({ animal: 'text', color: 'text', pattern: 'text', size: 'text' });

// with weights
schema.index({ animal: 'text', color: 'text', pattern: 'text', size: 'text' }, {name: 'My text index', weights: {animal: 10, color: 4, pattern: 2, size: 1}});
```

Now to call the search, use the `$text` option. The `$meta: 'textScore'` tells Mongoose to use the weights.

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

        if (req.header('Content-Type') == 'application/json') {
          return res.json({ pets: pets });
        } else {
          return res.render('pets-search', { pets: pets, term: req.query.term });
        }
      });
});
```
