<!---
This would be part of a lesson, not an assignment.  The lesson might begin with the rest-concepts stuff
-->
## **Express Concepts**

In the previous section on REST and HTTP, you learned about the components of an HTTP request: a method (such as GET, POST, PUT, PATCH, or DELETE), a path, query parameters, headers, sometimes a body, and cookies.  In Express, you have the following elements:

1. An app, as created by a call to Express

2. A collection of route handlers.  A request for a particular HTTP method and path are sent to a route handler.  For example, a POST for /notices would have a route handler that handles this route.  A route handler is a function with the parameters req, res, and sometimes next.  The req parameter is a structure with comprehensive the information about the request.  The res parameter is the way that the route handler sends the response.  The next parameter is needed in case the route handler needs to pass the request either to another route handler or to an error handler.

3. Middleware.  Middleware does some initial processing on the request.  Sometimes middleware checks to see if the request should be sent on to the next piece of middleware in the chain or the route handler, and if not, the middleware returns a response to the request itself.  The middleware may add additional information to the req object, and may also set headers, including sometimes set-cookie headers, in the response object.  Then it calls the next handler, which might be middleware, and might be the route handler.  Middleware always has three parameters, req, res, and next.  A standard piece of middleware is the not-found handler, which is invoked when no route handler could be found for the method and path.

4. An error handler.  This is at the end of the chain, in case an error occurs.  There is only one, and it takes four parameters, err, req, res, and next.

5. A server.  This is created as a result of an app.listen() on a port.

The mainline code in the app does the following:

1. It creates the app.

2. It specifies a list of middleware and route handlers to be called, with filter conditions based on the method and path that specify when each should be called.  Middleware is specified via app.use().  Route handlers are specified via app.get(), app.post(), and similar statements.  **Order matters in this configuration.**  The first app.use(), app.get(), or other such statement that is matched determines what is called.  If that is middleware, it will often do a next, and then the next middleware or route handler that matches the HTTP method and path is called.  While middleware often does a next(), route handlers only do a next(err), in cases when an error should be passed to the error handler.

3. It tells the app to listen on a port.

Here is an example of what app.js might look like.

```js
const express = require("express");

const app = express();

// the following statements declare the middleware and route handlers.  Nothing happens with them until a request is received.

app.use((req,res,next)=> {  // this is called for every request received.  All methods, all paths
    req.additional = {this: 1, that: "two"};
    const content = req.get("content-type");
    if (method == "POST" && content != "application/json") {
        next(new Error("A bad content type was received"); // this invokes the error handler
    } else {
        next(); // as OK data was received, the request is passed on to it.
    }  
}

app.get("/info", (req, res) => {  // this is only called for get requests for the specific path 
    res.send("We got good stuff here!")
}

app.use("/api", (req, res, next) => { // this is called for all methods, but only if the path begins with /api
                                      // and only if the request got past that first middleware.
    ...
})

app.use((req, res) =>{ // this is the not found handler.  Nothing took care of the request, so we send the caller the bad news.
    res.status(404).send("That route is not present.")
})

app.use((err, req, res, next) =>{
    console.log(err.constructor.name, err.message, err.stack)
    res.status(500).send("internal server error")
})

let server = null;

try {
    server = app.listen(3000)
    console.log("server up and running.")
} catch  {
    console.log("couldn't get access to the port.")
}
```

Of course, the actual functions comprising the route handlers and middleware are not typically declared inline.  Suppose one set of route handlers deals with customers, via GET/POST/PATCH/PUT/DELETE requests on all paths that start with /customers, and suppose you have another set for /orders.  Typically you would have a ./controllers folder, with a customerController.js for all the route handlers for customers and an orderController.js for all the route handlers for orders.  Typically also, you would have a ./routes folder, where modules configure express routers.  An express router is a way of associating a collection of routes with a collecton of route handlers.  You'd also have a middleware folder.  In the mainline code, you might have statements like:

```js
const customerRouter = require("./routes/customer")
const customerMiddleware = require("./middleware/customerAuth)
app.use("/customers", customerAuth, customerRouter)
```

The customerMiddleware might check if the customer is logged in, return a 401 if not, but if so call next, so that processing would pass to the customerRouter.

For each request sent to the server, there must be exactly one response.  If no response is sent, a user might be waiting at the browser for a timeout.  If several responses are sent for one request, Express reports an error instead of sending the second one.

## **What do Request Handlers Do?**

Request handlers may retrieve data and send it back to the caller.  Or, they may store, modify, or delete data, and report the success or failure to the caller.  Or, they may manage the session state of the caller, as would happen, for example, with a logon.  The data accessed by request handlers may be in a database, or it may be accessed via some network request.  When sending the response, a request handler might send plain text, HTML, or JSON, or any number of other content types.

Route handlers and middleware frequently do asynchronous operations, often for database access.  While the async request is being processed, other requests may come to the server, and they are dispatched as usual.  Route handlers and middleware may be declared as async, so that the async/await style of programming can be used.  These functions don't return a value.  Instead, they send a response, or pass control to the route handler that sends the response.

One very common piece of middleware is the following:

```js
app.use(express.json())
```

This middleware parses the body of a request that has content-type "application/json".  The resulting object is stored in req.body.

### **The req and res Objects**

You can access the following elements of the req:

req.method
req.path
req.params -- these are HTML path parameters
req.query  Query parameters
req.body    The body of the request, if any
req.host    The host that this Express app is running on

There are many more.

The req.get(headerName) function returns the value of a header associated with the request, if that header is present.
`req.cookies[cookiename]` returns the cookie of that name associated with request, if one is present.

The res object has the following methods:

res.status  This sets the HTTP status code
res.cookie  Causes a Set-Cookie header to be attached to the response.
res.setHeader Sets a header in the response
res.json    When passed a JavaScript object, this method converts the object to JSON and sends it back to the originator of the request
res.send    This sends plain text data, or perhaps HTML.

