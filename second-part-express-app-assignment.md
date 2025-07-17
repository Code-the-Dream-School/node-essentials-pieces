## **More Route Handlers and Testing with Postman**

For your final project, you'll have users with todo lists.  A user will be able to register with the application, log on, and create, modify, and delete tasks in their todo lists.  You'll now create the route that does the register.  That's a POST operation for the '/user' path.  Add that to app.js, before the 404 handler.  For now, you can just have it return a message.

You can't test this with the browser.  Browsers send GET requests, and only do POSTs from within forms.  Postman is the tool you'll use.  Start it up.  On the upper left hand side, you see a `new` button.  Create a new collection, called `node-homework`.  On the upper right hand side, you see an icon that is a rectangle with a little eye.  No, it doesn't mean the Illumnati.  This is the Postman environment.  Create an enviroment variable called host, with a value of `http://localhost:3000`.  This is the base URL for your requests.  When it comes time to test your application as it is deployed on the internet, you can just change this environment variable.

Hover over the node-homework collection and you'll see three dots. Click on those, and select 'add request'.  Give it a name, perhaps `register`.  A new request, by default, is a GET, but there is a pulldown to switch it to POST.  Save the requestn, and then send it.  If your Express app is running, you should see your message come back.  Of course, to create a user record, you need data in the body of the request.  So, click on the body tab for the request.  Select the `raw` option.  There's a pulldown to the right that says `Text`.  Click on that, and choose the JSON option.  Then, put JSON in for the user you want to create.  You need a name, an email, and a password.  Remember that this is JSON, not a JavaScript object, so you have to have double quotes around the attribute names and string values.  Save the request again, and then send it.  The result is the same of course -- the request handler doesn't do more than send a message at the moment.

Go back to app.js.  You need to be able to get the body of the request.  For that you need middleware, in this case middleware that Express provides.  Add this line above your other routes:

```js
app.use(express.json({ limit: "1kb" }));
```

This tells Express to parse JSON request bodies as they come in.  The express.json() middleware only parses the request body if the Content-Type header says "application/json". The resulting object is stored in req.body.  Of course, any routes that need to look at the request body have to come after this app.use().

Make the following change to the request handler:

```js
app.post("/user", (req, res)=>{
    console.log("This data was posted", JSON.stringify(req.body));
    res.send("parsed the data");
});
```

Then try the Postman request again.  You see the body in your server log, but you are still just sending back a message.  What you should do for this request is store the user record.  Eventually you'll store it in a database, but we haven't learned how to do that yet.  So, for the moment, you can just store it in memory.  Create a directory called util, and a file within it called memoryStore.js.  There's not much to this file:

```js
const storedUsers = [];

let loggedOnUser = null;

module.exports = {storedUsers, loggedOnUser};
```

Now, in app.js, above your app.post(), add this statement:

```js
const { storedUsers, loggedOnUser } = require("./util/memoryStore");
```

And then, change the app.post() as follows:

```js
app.post("/user", (req, res)=>{
    const newUser = {...req.body}; // this makes a copy
    storedUsers.push(newUser);
    loggedOnUser = newUser;  // After the registration step, the user is set to logged on.
    delete req.body.password;
    res.status(201).json(req.body);
});
```

Test this with your Postman request.

### **Why the Memory Store is Crude**

Let's list all the hokey things you just did.

1. No validation.  You don't know if there was a valid body.  Hopefully your Postman request did send one.

2. You stored to memory.  When you restart the server, the data's gone.  Your users will not be happy.

3. You don't know if the email is unique.  You are going to use the email as the userid, but a bunch of entries could be created with the same email.

4. You stored the plain text password, very bad for security.

5. Only one user can be logged on at a time.

Well ... we'll fix all of that, over time.

### **Keeping Your Code Organized: Creating a Controller**

You are going to have to create a couple more post routes.  Also, you are going to have to add a lot of logic, to solve problems 1 through 5 above.  You don't want all of that in app.js.  So, create a directory called controllers. Within it, create a file called userController.js.  Within that, create a function called register.  The register() function takes a req and a res, and the body is just as above.  You can move the require() statement for the memoryStore over there (but you have to use a relative path).  You should also do a require for http-status-codes, and instead of using 201, you use StatusCodes.CREATED.  Then, you put register inside the module.exports object for this module.

### **On Naming**

In the general case, you can name modules and functions as you choose.  However, we are providing tests for what you develop, and so you need to use the names specified below, so that the tests work:

```
/controllers/userController.js with functions login, register, and logoff
/controllers/taskController.js with functions index, create, show, update, and deleteTask.
```  

The show function returns a single task, and the index function returns all the tasks for the logged on user (or 404 if there aren't any.)

### **Back to the Coding***

When creating a new record, it is standard practice to return the object just created, but of course, you don't want to send back the password.

Change the code in for the route as follows:

```js
const { register } = require("./controllers/userController");
app.post("/user", register);
```

Test again with Postman to make sure it works.

### **More on Staying Organized: Creating a Router**

You are going to create several more user post routes, one for logon, and one for logoff.  You could have app.post() statements in app.js for each.  But as your application gets more complex, you don't want all that stuff in app.js.  So, you create a router.  Create a folder called routes.  Within that, create a file called user.js.  It should read as follows:

```js
const express = require("express");

const router = express.Router();
const { register } = require("../controllers/userController");

router.route("/").post(register);

module.exports = router;
```

Then, change app.js to take out the app.post().  Instead, put this:

```js
const userRouter = require("./routes/user");
app.use("/user", userRouter);
```

The user router is called for the routes that start with "/user".  You don't include that part of the URL path when you create the router itself.

All of the data sent or received by this app is JSON.  You are creating a back end that just does JSON REST requests.  So, you really shouldn't do res.send("everything worked.").  You should always do this instead:

```js
res.json({message: "everything worked."});
```

At this time, change the res.send() calls you have in your app and middleware to res.json() calls.

### **The Other User Routes**

Here's a spec.

1. You need to have a `/user/logon` POST route.  That one would get a JSON body with an email and a password.  The controller function has to do a find() on the storedUsers array for an entry with a matching email.  If it finds one, it checks to see if the password matches.  If it does, it returns a status code of OK, and a JSON body with the user name.  The user name is convenient for the front end, because it can show who is logged on.  The controller function for the route would also set the value of loggedOnUser to be the entry in the storedUsers array that it finds.  (You don't make a copy, you just set the reference.)  If the email is not found, or if the password doesn't match, the controller returns an UNAUTHORIZED status code, with a message that says Authentication Failed.

2. You need to have a `/user/logoff` POST route.  That one would just set the loggedOnUser to null and return a status code of OK.  You could do res.sendStatus(), because you don't need to send a body.

3. You add the handler functions to the userController, and you add the routes to the user.js router, doing the necessary exports and requires.

4. You test with Postman to make sure all of this works.

### **The Task Routes**

Create a task controller and a task router.  You need to support the following routes:

1. POST "/tasks".  This creates a new entry in the list of tasks for the currently logged on user.

2. GET "/tasks".  This returns the list of tasks for the currently logged on user.

3. GET "/tasks/:id".  This returns the task with a particular ID for the currently logged on user.

4. PATCH "/tasks/:id.  This updates the task with a particular ID for the currently logged on user.

5. DELETE "/tasks/:id.  This deletes the task with a particular ID for the currently logged on user.

So, that's five functions you need in the task controller, and five routes that you need in the task router.  But, we have a few problems:

- What if there is no currently logged on user?
- How do you assign an ID for each task?
- To get, patch, or delete a task, how do you figure out which one you are going to work on?

Let's solve each of these.  First, for every task route, we need to check whether there is a currently logged on user, and to return a 401 if there isn't.  If there is a logged on user, the job should pass to the task controller, and the task controller should handle the request.  So -- that's middleware.  Create a `/middleware/auth.js` file.  In it, you need a single function.  The function doesn't have to have a name, because it's going to be the only export.  It checks: is there a logged on user?  If not, it returns an UNAUTHORIZED status code and a JSON message that says "unauthorized".  If there is a logged on user, it calls next().  That sends the request on to the tasks controller.  Be careful that you don't do both of these: res.json() combined with next() would mess things up.

In app.js, you can then do:

```js
const authMiddleware = require("./middleware/auth");
```

But, `app.use(authMiddleware)` would protect any route.  Then no one could register or logon.  You want it only in front of the tasks routes.  So, you do the following:

```js
const taskRouter = require("./routers/task");
app.use("/tasks", authMiddleware, taskRouter);
```

That solves the first problem.  The authMiddleware gets called before any of the task routes, and it makes sure that no one can get to those routes without being logged on.  These are called "protected routes" because they require authentication.

Let's go on to problem 2.  Within your tasks controller, `loggedOnUser` is a reference to an object, and you want to have a list of tasks within that object.  Each should have a unique ID  You didn't create that list when you stored the user object. First, create a little counter function in taskController.js, as follows:

```js
const taskCounter = (() => {
  let lastTaskNumber = 0;
  return () => {
    lastTaskNumber += 1;
    return lastTaskNumber;
  }
})();
```

This is a closure.  You are sometimes asked to write a closure in job interviews.  we can use this to generate a unique ID for each task -- but of course, restart the server and you start over.

In taskController.js, you need a function called `create(req, res)`. And inside that, you do:

```js
if (loggedOnUser.tasklist === undefined) {
    loggedOnUser.tasklist = [];
};
req.body.id = taskCounter();
const newTask = {...req.body}; // make a copy
loggedOnUser.tasklist.push(newTask);
res.json(newTask);  // send it back, with an id attached
```

Now for problem 3.  When you have a route defined with a colon `:`, that has a special meaning.  The string following the colon is the name of a variable, and the value of the variable is in req.params.  For the routes above, you would have `req.params.id`.  Now, be careful: this is a string, not an integer, so you need to convert it to an integer before you go looking for the right task.  Here's how you could do it in a deleteTask(req,res) function in your task controller:

```js
let task = null;
const taskToFind = parseInt(req.params.id);
if (loggedOnUser.tasklist) {  // if we have a list
  const taskIndex = loggedOnUser.tasklist.find((task)=> task.id === taskToFind);
  if (taskIndex != -1) {
    task = loggedOnUser.tasklist[taskIndex];
    loggedOnUser.tasklist.splice(taskIndex, 1); // do the delete
  }
};
if (task) { // was it found?
  res.status(StatusCodes.OK).json(task); // return the entry just deleted
} else {
  res.sendStatus(StatusCodes.NOT_FOUND); // else it's a 404.
}
```

So, write the remaining methods, set up the routes, and test everything with Postman.  To test the operations that use a task ID, you would get all of the tasks for the currently logged on user, so you know what the IDs are.  Postman will show you what is sent back.  Then you can show or patch or delete one of them.

### **The Automated Tests**

Run `npm run tdd assignmentxxx` to see if your code works as expected.



