---
title: "Validation, Success, and Error Handling"
slug: "validation-success-and-error-handling"
---

A codebase can have various "smells". Smelly code is, as you can imagine, not a good thing. Right now this project is pretty clean all around in its architecture with good routing names, and virtually 100% test coverage, but it is very janky in the **Validations** and **Error/Success Handling** department.

If you are making a hackathon project, or your project isn't live yet, you probably don't have to worry about these topics, but anything you want to take seriously, you should prioritize these two stories on your Kanban board:


1. **Validate information submitted to the database**
1. **Respond elegantly to errors and successes**

# Data Validation

### What is Data Validation?

Validation is short for **Data Validation**, which is all the controls software engineers put into place to make sure that only valid data goes into a database.

For example, all strings in an `email` field, should be formatted like an email address: `something@something.something`. Data entered into a `phone` field should look something like this `+1234567890`. These are valid formulations because we define them that way using validations.

### Dangers of Not Validating

If we didn't put in validations people could cause errors, customer unhappiness, and hackers could even use our lack of validation to exploit our system.

### Where to Validate

Validations can happen at four places:

1. In the client
1. In the server in the controller (**parameter sanitization**)
1. In the server in the model
1. In the database

In production in a major application there will probably some sort of validation at each of these levels to be absolutely certain that data is valid at all levels and to reduce the chance for hackers to screw up our database.

However, most validation happens at the **client** and **model** levels. This is because if something gets through the client validation, the model will block it, and send a message back to the client that the client can display.

**We're going to focus on implementing validation at those two levels for the pet emporium!**

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

1. Submit the form via `AJAX` (to prevent the page reloading upon an error)
1. Set model validations.
1. Catch model validation errors and respond elegantly.
1. Set client-side validations.

# Submit Form via AJAX with Axios

>[action]
> Add a link to `Axios` and to your `scripts.js` file in the `/views/layout.pug` template at the bottom with the other scripts:
>
```pug
.footer
...
>
script(src="https://unpkg.com/axios/dist/axios.min.js")
script(src="/javascripts/scripts.js")
```
>
Add an `alert('hello');` into `/public/javascripts/scripts.js`. Refresh the page to make sure you see the alert pop up.

Now let's change the form's action, add an id attribute of `new-pet`.

>[action]
> Update the `action` and `id` attributes of the `form` in `/views/pets-new.pug`:
>
```pug
//- pugs-new.jade
>
...
form(action="#" id="new-pet")
  ...
```
>
> In `/public/javascripts/scripts.js`, remove your `alert` and add the request to `POST` the form data to the server.
>
```js
if (document.querySelector('#new-pet')) {
    document.querySelector('#new-pet').addEventListener('submit', (e) => {
        e.preventDefault();
>
        let pet = {};
        const inputs = document.querySelectorAll('.form-control');
        for (const input of inputs) {
            pet[input.name] = input.value;
        }
>
        axios.post('/pets', pet)
            .then(function (response) {
                window.location.replace(`/pets/${response.data.pet._id}`);
            })
            .catch(function (error) {
                const alert = document.getElementById('alert')
                alert.classList.add('alert-warning');
                alert.textContent = 'Oops, something went wrong saving your pet. Please check your information and try again.';
                alert.style.display = 'block';
            });
    });
}
```

Finally, we update our server `/pets/create/` route to be ready to handle a `JSON` request:

>[action]
> Replace the current `create` route in `/controllers/pets.js` with the following:
>
```js
// pets.js
  // CREATE PET
  app.post('/pets', (req, res) => {
    var pet = new Pet(req.body);
>
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

Let's add a simple validation in our `pet` model using mongoose's built in validation options.

> [info]
> Take about 10 minutes to read Mongoose's [documentation](http://mongoosejs.com/docs/validation.html) on validations

As a reminder, these are the validations we want to have in our model:

* All pet attributes are required
* `description` should be longer than 140 characters
* `picUrl` and `picUrlSq` should both be URLs

>[action]
> Open up `/models/pet.js` and update the `schema` to the following:
>
```js
const PetSchema = new Schema({
  name: { type: String, required: true }
  , birthday: {type: String, required: true }
  , species: { type: String, required: true }
  , picUrl: { type: String, required: true }
  , picUrlSq: { type: String, required: true }
  , favoriteFood: { type: String, required: true }
  , description: { type: String, minlength: 140, required: true }
}, {
  timestamps: true
});
```

Now we have our validations set in the model. Will an invalid pet be saved anymore?

It sure can! Let's fix this to display a generic "Oops, something went wrong" alert using the Bootstrap alert module.

# Displaying Alerts

Let's add a hidden Bootstrap 4 alert module to our layout template that we we'll use JavaScript to manipulate in the case of an error.

> [action]
> Add the `alert` to the `container` module in `/views/layout.pug`:
>
```pug
.navbar
  ...
.container
  .alert#alert(style="display:none;")
  block content
```

Now we can manipulate this alert via JavaScript when there is an error (or a success!);

>[action]
> Add the following `catch` code to your own `catch` block in `/public/scripts.js`:
>
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

Now that we see the alert, can you use the [setTimeout(callback, milliseconds)](https://www.w3schools.com/js/js_timing.asp) method make it disappear after 3 seconds? Remember to remove the styling of the alert so it is ready to be used again next time.

> [solution]
>
> Add the folllowing at the end of the `catch` block  in `/public/scripts.js`:
>
```js
setTimeout(() => {
 alert.style.display = 'none';
 alert.classList.remove('alert-warning');
}, 3000)
```

# Client-Side Validations

Finally let's finish off our validations by adding client-side validations into our HTML form. This will prevent people from  even submitting a form that has invalid data.

> [action]
> Update all of the `.form-group` elements in `/views/pets-new.pug` to  the following:
>
```pug
//- pets-new.pug
...
>
.form-group
    label Name*
    input.form-control(name="name" required)
.form-group
    label Birthday*
    input.form-control(name="birthday" required)
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
>
...
```

Tool around with the form now and try to submit the form and see how the client-side validations look.

Now you have client-side validations as a first line of defense to malformed data getting into your database. And if anything get's through those, the error will be caught by the model validations. Ain't no invalid data getting through that!

# Now Commit

```bash
git add .
git commit -m 'Implemented validation'
git push
```

Onward!
