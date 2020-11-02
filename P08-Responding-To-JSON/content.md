---
title: "Review - Responding to JSON"
slug: "review-responding-to-json"
---

1. ~~Implement simple search on the store~~
1. ~~Build out pagination~~
1. ~~Implement validations, success, and error handling~~
1. ~~Uploading files~~
1. ~~Integrating payment gateways~~
1. ~~Sending emails~~
1. ~~Building Full-text Search~~
1. **Responding to JSON**
    1. **Detect JSON Requests in addition to handling rendering as we did before**
    1. **Test API endpoints**

So far we have built a web server, but we have not built an API. If we wanted to add a mobile app to this server, we'd be stuck. A mobile app, desktop app, or an independent front end app (maybe built with React) would need to send and receive JSON. Our server would need to **respond to JSON**.

As a closing to this tutorial, let's do a quick review of how to set this up. It turns out there are two ways to do this.

# Detecting JSON Requests

If you want to build an API fast, you can  detect if the `Content-Type` of the request is `application/json` and then return JSON, otherwise return the HTML template. Let's try this with our landing page:

> [action]
>
> Update `routes/index.js` to the following:
>
```js
const Pet = require('../models/pet');
>
module.exports = (app) => {
>
  app.get('/', (req, res) => {
    const page = req.query.page || 1
>
    Pet.paginate({}, { page: page }).then((results) => {
      // If the request is JSON, we want to send a JSON response
      if (req.header('Content-Type') == 'application/json') {
        return res.json({ pets: results.docs, pagesCount: results.pages, currentPage: page });
      // Otherwise we do what we did before
      } else {
        res.render('pets-index', { pets: results.docs, pagesCount: results.pages, currentPage: page });
      }
    });
  });
}
```

In addition to this change, you'll have to tell your mobile developer, or your front end developer to update their requests to always explicitly request `application/json`.

# Testing API Endpoints

To test that your server is responding with both HTML and JSON, add additional tests for the JSON responses.

> [action]
>
> Add the following test to `/tests/test-pets.js` check for valid JSON responses:
>
```js
  it('should list ALL pets on /pets GET', function(done) {
    chai.request(server)
        .get('/')
        .set('content-type', 'application/json')
        .end(function(err, res){
          res.should.have.status(200);
          res.should.be.json;
          res.body.should.be.a('object');
          done();
        });
  });
```

# Another Option - New Controllers

If you are building a large, funded, revenue-generating web server, then you will want to build an entire separate set of controllers for your API. In a large enough company an API will be an entirely built by a different team!

The first step is to namespace your API routes with the preface `/api/` so all resourceful routes that go to your API controllers always are prefaced with `/api/`.

```
<!-- For example -->
show - GET - /api/pets/:id
create - POST - /api/pets
update - PUT - /api/pets/:id
delete - DELETE - /api/pets/:id
```

In each of these routes they don't use the `res.render()` function which would return HTML templates, they use the `res.json` function.

```js
// return res.render('pets-index', { pets: pets });
return res.json({ pets: pets });
```

# Now Commit

```bash
$ git add .
$ git commit -m 'Implemented responding to JSON'
$ git push
```

**Congrats on completing the multi-faceted store that is Proud Pete's Pet Emporium!** As you go through this BEW course, think about how you can tie the concepts you learned in this tutorial to the big ideas and concepts in the course itself.

# Feedback and Review - 2 minutes

**We promise this won't take longer than 2 minutes!**

Please take a moment to rate your understanding of learning outcomes from this tutorial, and how we can improve it via our [tutorial feedback form](https://goo.gl/forms/I0vguSrQT57NY7jX2)
