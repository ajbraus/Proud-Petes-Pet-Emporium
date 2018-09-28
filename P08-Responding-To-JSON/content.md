---
title: "Responding to JSON"
slug: "responding-to-json"
---

So far we have built a web server, but we have not built an API. If we wanted to add a mobile app to this server, we'd be stuck. A mobile app, desktop app, or an independent front end app (maybe built with React) would need to send and receive JSON. Our server would need to **respond to JSON**.

It turns out there are two ways to do this.

# Option 1 - New Controllers

If you are building a large, funded, revenue-generating web server, then you will want to build an entire separate set of controllers for your API. In a large enough company an API will be an entirely different team!

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
return res.render('pets-index', { pets: pets });
```

# Option 2 (shortcut) - Detecting JSON Requests

If you are building something that is unfunded, small, and lean and you want to build an API fast, you can just detect if the `Content-Type` of the request is `application/json` and then return JSON, otherwise return the HTML template.

```js
// INSIDE ANY ROUTE

// REPLACE
return res.render('pets-index', { pets: pets });

// WITH
if (req.header('Content-Type') == 'application/json') {
  return res.json({ pets: pets, categories: categories });
} else {
  return res.render('pets-index', { pets: pets });
}
```

In addition to this change, you'll have to tell your mobile developer, or your front end developer to update their requests to always explicitly request `application/json`.

# Testing API Endpoints

To test that your server is responding with both HTML and JSON, add additional tests for the JSON responses that look like this (for Option #2):

```js
  it('should list ALL pets on /pets GET', (done) => {
    chai.request(server)
        .set('content-type', 'application/json')
        .get('/pets')
        .end(function(err, res){
          res.should.have.status(200);
          res.should.be.json;
          res.body.should.be.a('array');
          done();
        });
  });
```
