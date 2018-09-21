---
title: "Validation, Success, and Error Handling"
slug: "validation-success-and-error-handling"
---

A codebase can have various "smells". Smelly code is, as you can imagine, not a good thing. Right now this project is pretty clean all around in its architecture with good routing names, and virtually 100% test coverage, but it is very janky in the **Validations** and **Error/Success Handling** department.

If you are making a hackathon project, or your project isn't live yet, you probably don't have to worry about these topics, but anything you want to take seriously, you should prioritize these two stories on your Kanban board:

```
Validate information submitted to the database
Respond elegantly to errors and successes
```

# Data Validation

### What is Data Validation?

Validation is short for "Data Validation" and means all the controls software engineers put into place to make sure that only valid data goes into a database.

For example, all strings in an `email` field, should be formatted like an email address: `something@something.something`. Data entered into a `phone` field should look something like this `+1234567890`. These are valid formulations because we define them that way using validations.

### Dangers of Not Validating

If we didn't put in validations people could cause errors, customer unhappiness, and hackers could even use our lack of validation to exploit our system.

### Where to Validate

Validations can happen at four places, but most validation happens at the **client** and **model** levels.

1. In the client (we'll do validations here!)
1. In the server in the controller (**parameter sanitization**)
1. In the server in the model (we'll do validation here!)
1. In the database

In production in a major application there will probably some sort of validation at each of these levels to be absolutely certain that data is valid at all levels and to reduce the chance for hackers to screw up our database.

In any project, though we should have some **model-level data validation**, so that is what we are going to add to our pet store today.

Validations are not the most elegant thing in the world, but the most elegant way to handle validations is to put them at the client level and then back those up at the model level. If something gets through the client validation, the model will block it, and send a message back to the client that the client can display.

Because this is the best practice, we are going to refactor our pet store to follow it. So we are going to first **Make a Plan**.

# Make a Plan

Here is the user narrative of how the user should experience errors:

1. User puts in invalid data and hits submit.
2. Without navigating away, user sees specifically the form elements that still need work with directions for what to do.
3. User submits data that is still invalid (<1% of time)
4. User sees a general and generic error message that something has occurred.

Here are the validations we want:

* All pet attributes are required
* `description` should be longer than 140 characters
* `picUrl` and `picUrlSq` should both be URLs

In order to accomplish this user flow, we have to implement client-side validation, and client-side JavaScript to submit the form.

1. Submit the form via AJAX (to prevent the page reloading upon an error)
1. Set model validations.
1. Catch model validation errors and respond elegantly.
1. Set client-side validations.

# Submit Form via AJAX with Axios

Add Axios and link to your `scripts.js` file in the `layout.pug` template at the bottom.

```pug
//- layout.pug
.footer
...

script(src="https://unpkg.com/axios/dist/axios.min.js")
script(src="/javascripts/scripts.js")
```

Add an `alert('hello')` into your scripts.js to make sure it is working.

Now let's change the form's action, and add an id attribute of `new-pet`.

```pug
//- pugs-new.jade

form(action="#" id="new-pet")
  ...
```

In your `scripts.js` let's add the request to POST the form data to the server.

```js
document.querySelector('#new-pet').addEventListener('submit', (e) => {
  e.preventDefault();

  let pet = {};
  const inputs = document.querySelectorAll('.form-control');''
  for (const input of inputs) {
    pet[input.name] = input.value;
  }

  axios.post('/pets', pet)
    .then(function (response) {
      window.location.replace(`/pets/${response.data._id}`);
    })
    .catch(function (error) {
      console.log(error)
    });
});
```

And then we update our server pets#create route to be ready to handle a JSON request:

```js
// pets.js
  // CREATE PET
  app.post('/pets', (req, res) => {
    var pet = new Pet(req.body);

    pet.save()
      .then((pet) => {
        res.send({ pet: pet });
      })
      .catch((err) => {
        // STATUS OF 400 FOR VALIDATIONS
        res.status(400).send(err.errors);
      }) ;
  });
```

Check if this allows you to save new pets. We haven't added any validations yet

# Validation in the Model

Open up your Pet model in the `pet.js` file, and let's add a simple validation using mongoose's built in validation options.

> info
> Take about 10 minutes to read Mongoose's [documentation](http://mongoosejs.com/docs/validation.html) on validations

Something special about mongoose is the canned standard validation messages are really terrible, so we are going to customize those for our app and I recommend you always do this.

```js
const PetSchema = new Schema({
    createdAt       : { type: Date }
  , updatedAt       : { type: Date }

  , name            : { type: String, required: true }
  , species         : { type: String, required: true }
  , picUrl          : { type: String, required: true }
  , picUrlSq        : { type: String, required: true }
  , favoriteFood    : { type: String, required: true }
  , description     : { type: String, min: 140 }
}, {
  timestamps: true
});
```

Now we have our validations set in the model. Will an invalid pet be saved anymore?

If you test sending in some invalid data, all you will have is a log in your client's console. Let's fix this to display a generic "Oops, something went wrong" alert using the Bootstrap alert module.

# Displaying Alerts

Let's add a hidden Bootstrap 4 alert module to our layout template that we we'll use JavaScript to manipulate in the case of an error.

```pug
//- layout.pug

.navbar
  ...
.container
  .alert#alert(style="display:none;")
```

Now we can manipulate this alert via JavaScript when there is an error (or a success!);

```js
axios.post('/pets', pet)
  .then(function (response) {
    window.location.replace(`/pets/${response.data._id}`);
  })
  .catch(function (error) {
    const alert = document.getElementById('alert')
    alert.classList.add('alert-warning');
    alert.textContent = 'Oops, something went wrong saving your pet. Please check your information and try again.';
    alert.style.display = 'block';
  });
```

Submit invalid data and see the alert appear.

Now that we see the alert, can you use the `setTimeout(callback, milliseconds)` method make it disappear after 3 seconds? Remember to remove the styling of the alert so it is ready to be used again next time.

> [solution]
> ```js
> setTimeout(() => {
>   alert.style.display = 'none';
>   alert.classList.remove('alert-warning');
> }, 3000)
> ```

# Client-Side Validations

Finally let's finish off our validations by adding client-side validations into our HTML form. This will prevent people from  even submitting a form that has invalid data.

```pug
//- pets-new.pug
...

.form-group
    label Name*
    input.form-control(name="name" required)
.form-group
    label Species*
    input.form-control(name="species" required)
.form-group
    label Rectangular Image URL*
    input.form-control(name="picUrl" required type="url")
.form-group
    label Square Image URL*
    input.form-control(name="picUrlSq" required type="url")   
.form-group
    label Favorite Food*
    input.form-control(name="favoriteFood" required)
.form-group
    label Description*
    textarea.form-control(name="description" required minlength="140")
```

Tool around with the form now and try to submit the form and see how the client-side validations look.

Now you have client-side validations as a first line of defense to malformed data getting into your database. And if anything get's through those, the error will be caught by the model validations. Ain't no invalid data getting through that!

Onward!
