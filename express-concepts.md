<!---
This would be part of a lesson, not an assignment.  The lesson might begin with the rest-concepts stuff
-->
## **Express Concepts**

In the previous section on REST and HTTP, you learned about the components of an HTTP request: a method (such as GET, POST, PUT, PATCH, or DELETE), a path, query parameters, headers, sometimes a body, and cookies.  In Express, you have the following elements:

1. An app, as created by a call to Express.

2. A collection of route handlers.  A request for a particular HTTP method and path are sent to a route handler.  For example, a POST for /notices would have a route handler that handles this route.  A route handler is a function with the parameters req, res, and sometimes next.  The req parameter is a structure with comprehensive the information about the request.  The res parameter is the way that the route handler sends the response.  The next parameter is needed in case the route handler needs to pass the request either to another route handler or to an error handler.

3. Middleware.  Middleware functions do some initial processing on the request.  Sometimes a middleware function checks to see if the request should be sent on to the next piece of middleware in the chain or the route handler.  If not, the middleware itself returns a response to the request.  In other cases, a middleware function may add additional information to the req object, and may also set headers, including sometimes set-cookie headers, in the res object.  Then it calls the next handler, which might be more middleware, and might be the route handler for the route.  Middleware functions typically have three parameters, req, res, and next.  A standard piece of middleware you'll create is the not-found handler, which is invoked when no route handler could be found for the method and path.

4. An error handler.  This is at the end of the chain, in case an error occurs.  There is only one, and it takes four parameters, err, req, res, and next, which is how Express knows it is an error handler.

5. A server.  This is created as a result of an app.listen() on a port.

The mainline code in the app does the following:

1. It creates the app.

2. It specifies a chain of middleware functions and route handlers to be called, each with filter conditions based on the method and path of the request.  These conditions determine each should be called.  Middleware functions are configured in the chain via an app.use() statement.  Route handlers are configured in the chain via app.get(), app.post(), and similar statements.

**Order matters in this configuration.**  The first app.use(), app.get(), or other such statement that is matched determines what function is called.  If that is a middleware function, it will often do a next, and then the next middleware function or route handler in the chain that matches the HTTP method and path is called.  While middleware often does a next(), route handlers only do a next(err), in cases when an error should be passed to the error handler.

3. It tells the app to listen on a port.

Here is an example of what app.js might look like.

```js
const express = require("express");

const app = express();

// the following statements configure the chain of middleware and route handlers.  Nothing happens with them until a request is received.

app.use((req, res, next) => {
  // this is called for every request received.  All methods, all paths
  req.additional = { this: 1, that: "two" };
  const content = req.get("content-type");
  if (req.method == "POST" && content != "application/json") {
    next(new Error("A bad content type was received")); // this invokes the error handler
  } else {
    next(); // as OK data was received, the request is passed on to it.
  }
});

app.get("/info", (req, res) => {
  // this is only called for get requests for the specific path
  res.send("We got good stuff here!");
});

app.use("/api", (req, res, next) => {
  // this is called for all methods, but only if the path begins with /api
  // and only if the request got past that first middleware.
  // ...
});

app.use((req, res) => {
  // this is the not found handler.  Nothing took care of the request, so we send the caller the bad news.  You always need one of these.
  res.status(404).send("That route is not present.");
});

app.use((err, req, res, next) => { // The error handler.  You always need one.
  console.log(err.constructor.name, err.message, err.stack);
  res.status(500).send("internal server error");
});

let server = null;

try {
  server = app.listen(3000);
  console.log("server up and running.");
} catch {
  console.log("couldn't get access to the port.");
}
```

Of course, the actual functions comprising the route handlers and middleware functions are not typically declared inline.  Suppose one set of route handlers deals with customers, via GET/POST/PATCH/PUT/DELETE requests on all paths that start with `/customers`, and suppose you have another set for `/orders`.  Typically you would have a `./controllers` folder, with a `customerController.js` for all the route handlers for customers and an `orderController.js` for all the route handlers for orders.  Typically also, you would have a `./routes` folder, for modules that create express routers.  An express router is a way of associating a collection of routes with a collecton of route handlers.  You'd also have a middleware folder.  In the mainline code, you might have statements like:

```js
const customerRouter = require("./routes/customer")
const customerMiddleware = require("./middleware/customerAuth)
app.use("/customers", customerAuth, customerRouter)
```

The customerMiddleware function might check if the customer is logged in.  If not, it could send a response with a 401 status code, possibly with an error message in the body.  If a customer is logged in, the middleware functiom might add additional information to the req object.  It would call next, so that processing would pass to the customerRouter.

For each request sent to the server, there must be exactly one response.  If no response is sent, a user might be waiting at the browser for a timeout.  If several responses are sent for one request, Express reports an error instead of sending the second one.

## **What do Request Handlers Do?**

Request handlers may retrieve data and send it back to the caller.  Or, they may store, modify, or delete data, and report the success or failure to the caller.  Or, they may manage the session state of the caller, as would happen, for example, with a logon.  The data accessed by request handlers may be in a database, or it may be accessed via some network request.  When sending the response, a request handler might send plain text, HTML, or JSON, or any number of other content types.  A request handler must either send a response or call the error handler to send a response.  Otherwise the request from the caller will wait until timeout.

Route handlers and middleware frequently do asynchronous operations, often for database access.  While the async request is being processed, other requests may come to the server, and they are dispatched as usual.  Route handlers and middleware may be declared as async, so that the async/await style of programming can be used.  These functions don't return a value of interest.

## **Middleware Functions, Route Handlers, and Error Handling**

Let's sum up common characteristics of middleware functions and response handlers.  Let's also

1. They are each called with the parameters req and res, or possibly req, res, and next.  They may be declared as async functions.

2. Once they are called, these functions do processing based on the information in the req object: method, path, path parameters, query parameters, headers, cookies, the body.  Every request has a method and path, but the other request attributes may or may not be present.

3. These functions must do one of the following, or the request times out:

- Send a response.
- Call next().
- Throw an error.

Even route handlers sometimes call next().  In these cases they call `next(error` to pass the error to the error handler.  Middleware functions often call next() withouut parameters, to call the next middleware in the chain or the route handler for the request, but they might call `next(error)` in some cases.

4. If `next(error)` is called or an error is thrown, the error handler is called and passed the error.  In Express 5, this happens even if the error is thrown while an async middleware function or route handler is waiting on an asynchronous operation and that error is not caught in the middleware function or route handler.  **However, please note:** Middleware functions and route handlers sometimes call functions that have callbacks.  They may send responses or call next() from with the callback.  That works fine.  But they must **never** throw an error from within a callback.  That would crash the server.  They must call `next(error)` instead.

## **Parsing the Body of a JSON Request**

One very common piece of middleware is the following:

```js
app.use(express.json())
```

This middleware parses the body of a request that has content-type "application/json".  The resulting object is stored in req.body.  There are other body parsers to be used in other circumstances, for example to catch data that is posted from an HTML form.

### **The req and res Objects**

You can access the following elements of the req:

req.method
req.path
req.params  HTML path parameters, if any.  When you configure a route with a route handler, you can tell Express where these are in the URL.
req.query  query parameters of the request, if any
req.body    The body of the request, if any
req.host    The host that this Express app is running on

There are many more.

The req.get(headerName) function returns the value of a header associated with the request, if that header is present.
`req.cookies[cookiename]` returns the cookie of that name associated with request, if one is present.

The res object has the following methods:

res.status()  This sets the HTTP status code
res.cookie()  Causes a Set-Cookie header to be attached to the response.
res.setHeader() Sets a header in the response
res.json()    When passed a JavaScript object, this method converts the object to JSON and sends it back to the originator of the request
res.send()    This sends plain text data, or perhaps HTML.

### **Check for Understanding**

1. What are the parameters always passed to a route handler?  What are they for?

2. What must a route handler always do?

3. How does a middleware function differ from a route handler?

4. If you do an await in a route handler, who has to wait?

5. You add a middleware function or router to the chain with an app.use() statement.  You add a route handler to the chain with an app.get() or similar statement.  Each of these has filter conditions.  How do the filter conditions differ?

### **Answers**

1. A route handler always gets the req and res objects.  The req object contains information from the request, perhaps with additional attributes added by middleware functions.  The res object has methods including res.send() and res.json() that enable the route handler to respond to the request.

2. A route hander must either respond to the request with res.send() or res.json(), or call the error handler with next(error), or throw the error.

3. A middleware function may or may not respond to the request.  Instead it may call next() to pass control to the next handler or middleware function in the chain.  It must either respond to the request or call next, or perhaps throw an error.

4. If you do an await in a route handler, the caller for the request has to wait.  You might be waiting, for example, on a response from the database.  You need the database response before you can send the HTTP response to the caller.  On the other hand, other callers don't have to wait, unless they make a request that ends up at an await statement.

5. An app.use() statement has an optional parameter, which is the path prefix.  You might have a statement like app.use("/api", middlewareFunction), and then middlewareFunction would be called for every request with a path starting with "/api", no matter if it is a GET, a POST, or whatever.  If the path prefix is omitted, the middleware function is called for every request.  An app.get() statement, on the other hand, calls the corresponding route handler only if the path matches exactly and the method is a GET.  In either case, you can pass additional middleware functions as parameters to be called in order, like:

```js
app.use("/api", middleware1, middleware2, middleware3);
app.get("/info", middleware1, infoHandler);
```