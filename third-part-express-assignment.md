## **Validation of User Input**

At present, your app stores whatever you throw at it with Postman.  There is no validation whatsoever.  Let's fix that.  There are various ways to validate user data.  We will eventually use a database access tool called Prisma, and it has validation built in, but it is very TypeScript oriented.  So we'll use a different library called Joi.  Do an npm install of it now.

Consider a user entry.  You need a name, an email, and a password.  You don't want any leading or trailing spaces.  You can't check whether the email is a real one, but you can check if it complies with the standards for email addresses.  You want to store the email address as lower case, because you need it to be unique in your data store, so you don't want to deal with case variations.  You don't want trivial, easily guessed passwords.  All of these attributes are required.

Consider a task entry.  You need a title.  You need a boolean for isCompleted.  If that is not provided, you want it to default to false.  The title is required in your req.body when you create the task entry, but if you are just updating the isCompleted, the patch request does not have to have a title.  We won't worry about the task id -- you automatically create this in your app.  In the database, each task will also have a userId, indicating which user owns the task, but that will be automatically created too.

Joi provides a very simple language to express these requirements.  If a user sends a request where the data doesn't meet the requirements, Joi can provide error messages to send back.  Create a folder called validation.  Create two files in that folder, userSchema.js and taskSchema.js.  Here's the code for userSchema.js:

```js
const Joi = require("joi");

const userSchema = Joi.object({
  email: Joi.string().trim().lowercase().email().required(),
  name: Joi.string().trim().min(3).max(30).required(),
  password: Joi.string()
    .trim()
    .min(8)
    .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^a-zA-Z0-9]).+$/)
    .required()
    .messages({
      "string.pattern.base":
        "Password must be at least 8 characters long and include upper and lower case letters, a number, and a special character.",
    }),
});

module.exports = { userSchema };
```

You can look at the code and guess what it does.  There are some nice convience functions, like email(), which checks for a syntactically valid email.  The only complicated one is the password.  This is a simple check for trivial passwords.  The password pattern is a regular expression, and the customized error message explains what is wrong if the password is inadequate.

Here is the code for taskSchema.js:

```js
const Joi = require("joi");

const taskSchema = Joi.object({
  title: Joi.string().trim().min(3).max(30).required(),
  isCompleted: Joi.boolean().default(false).not(null),
});

const patchTaskSchema = Joi.object({
  title: Joi.string().trim().min(3).max(30).not(null),
  isCompleted: Joi.boolean().not(null),
}).min(1);

module.exports = { taskSchema, patchTaskSchema };
```

The `min(1)` means that while both title and isCompleted are optional in a patch task request, you have to have one of those attributes -- otherwise there's nothing to do.  To do a validation, you do the following:

```js
const {error, value} = userSchema.validate({name: "Bob", email: "nonsense", password: "password", favoriteColor: "blue"}, {abortEarly: false})
```

You do `{abortEarly: false}` so that you can get all the error information to report to the user, not just the first failure.  When the validate() call returns, if error is not null, there is something wrong with the request, and error.message says what the error is.  If error is null, then value has the object you want to store, which may be different from the original.  The email would have been converted to lower case, for example.  In this case, the email is not correct, the password is not allowed, and favoriteColor is not part of the schema, so there are three errors.  The min(1) business doesn't really work very well, in that you get an error, but the error message isn't good enough for the end user.

Add validations to your create operations for users and tasks, and your to your update operation for tasks.  You validate req.body.  If you get an error, you return a BAD_REQUEST status, and you send back a JSON body with the error message provided by the validation.  If you don't get an error, you go ahead and store the returned value, returning a CREATED, or an OK if the update completes.  Then test your work with Postman, trying both good and bad requests.  


