---
title: "Full Text Search"
slug: "full-text-search"
---

```js
// SEARCH
app.get('/search', function (req, res) {
  Tour
      .find(
          { $text : { $search : req.query.term } },
          { score : { $meta: "textScore" } }
      )
      .sort({ score : { $meta : 'textScore' } })
      // .sort('-createdAt')
      // .populate('user')
      .limit(20)
      .exec(function(err, tours) {
        if (err) { return res.status(400).send(err) }
        // if (tours.length == 0) { return res.status(400).send({ message: "Your query returned no campaigns" }) }
        // if (tours.length == 0) {
        //   return res.render('tours-search', { tours: [] });
        // }

        if (req.header('Content-Type') == 'application/json') {
          return res.json({ tours: tours });
        } else {
          return res.render('tours-search', { tours: tours, term: req.query.term });
        }
      });

  // Tour.paginate({}, { sort:'-createdAt', page: req.query.pages, populate: 'user' }).then(function (result) {
    // if (err) { return res.status(400).send(err) }
  // });
});
```
