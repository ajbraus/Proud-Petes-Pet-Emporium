---
title: "Validation, Success, and Error Handling"
slug: "validation-success-and-error-handling"
---

A codebase can have various "smells". Smell code is, as you can imagine, not a good thing. Right now this project is pretty clean all around in its architecture with good routing names, and virtually 100% test coverage, but it is very janky in the **Validations** and **Error/Success Handling** department.

If you are making a hackathon project, or your project has been live for less than a week, you probably don't have to worry about these topics, but anything you want to take seriously, you should prioritize these two stories on your Kanban board:

```
Validate information submitted to the database
Respond elegantly to errors and successes
```


# Data Validation

### What is Data Validation?

Validation is short for "Data Validation" and means all the controls software engineers put into place to make sure that only valid data goes into a database.

For example, all strings in an `email` field, should be formated like an email address: `something@something.something`. Data entered into a `phone` field should look something like this `+1234567890`. These are valid formulations because we define them that way using validations.

### Dangers of Not Validating

If we didn't put in validations people could cause errors, customer unhappiness, and hackers could even use our lack of validation to exploit our system.

### Where to Validate

Validations can happen at four places, but most validation happens at the model level.

1. In the client
1. In the server in the controller (**parameter sanitization**)
1. In the server in the model (put validation here!)
1. In the database

In production in a major application there will probably some sort of validation at each of these levels to be absolutely certain that data is valid at all levels.

In any project, though we should have some model-level data validation, so that is what we are going to add to our pet store today.

Validations are not elegant, but the most elegant way to handle validations is to put them at the client level and then back those up at the model level. If something gets through the client validation, the model will block it, and send a message back to the client that the client can display.

Because this is the best practice, we are going to refactor our pet store to follow it. So we are going to first **Make a Plan**.

# Make a Plan

Here is the user narrative of how the user should experience errors:

1. User puts in invalid data and hits submit.
2. Without navigating away, user sees specifically the form elements that still need work with directions for what to do.
3. User submits data that is still invalid (<1% of time)
4. User sees a general and generic error message that something has occurred.

In order to accomplish this user flow, we have to implement client-side validation, and client-side JavaScript to submit the form.

1. Refactor to submit the form with the library [axios](https://github.com/axios/axios).
2. Add client-side validation library [jQuery Validate](https://jqueryvalidation.org/) (jQuery).
3. Configure jQuery Validate.
4. Set model validations.
5. Catch model validation errors and respond elegantly.
6. Display model validation errors elegantly for the user.

# Submit Form with Axios


# jQuery Validate



# Validation in the Model

Open up your Pet model in the `pet.js` file, and let's add a simple validation using mongoose's built in validation options.

> info
> Take about 10 minutes to read Mongoose's [documentation](http://mongoosejs.com/docs/validation.html) on validations

Something special about mongoose is the canned standard validation messages are really terrible, so we are going to customize those for our app and I recommend you always do this.

```js
const PetSchema = new Schema({
    createdAt       : { type: Date }
  , updatedAt       : { type: Date }

  , name            : { type: String, required: [true, 'Give that little guy a name!',  }
  , species         : { type: String, required: [true, 'uh-oh forgot the species'] }
  , picUrl          : { type: String, required: [true, 'Needs a rectangular picture'] }
  , picUrlSq        : { type: String, required: [true, 'Needs a square picture'] }
  , favoriteFood    : { type: String, required: [true, 'What about a favorite food?'] }
  , description     : { type: String, min: [100, 'Pets find homes faster with a good description.'] }
});
```

Now we have some validations, and custom messages, try to test it. What's not working?

# Catching Validations & Handling Errors

Did you figure it out? Our routes actually do not handle errors, they just go right through and redirect to the `pets#show` route, even if the pet resource was not created. We've got to catch these errors and have the program respond intelligently and let the user know that there is a problem. This is called **Error Handling**.

The simplest step is to just console log what we're getting as an error so we'll add `if (err) { return console.log(err) }`.

```js
// CREATE PET
app.post('/pets', (req, res) => {    
  var pet = new Pet(req.body);

  pet.save()
    .then((pet) => {
      res.redirect(`/pets/${pet._id}`);
    })
    .catch((err) => {
      if (err) { return console.log(err) }
    }) ;
});
```

Now to get this long list of validation messages to work well, we're going to have to do some careful steps to solve a few problems:

1.
