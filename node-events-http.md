# Lesson 2 More Node Basics and an Introduction To Express


## **2.x Event Emitters and Listeners**

Event emitters allow communication between different parts of an application.  Once an event emitter has been created, listeners can sign up to get called whenever the emitter emits an event.  An event has a name, and may also pass various arguments to the listeners.  Here is an example:

```js
const EventEmitter = require("events");
const emitter = new EventEmitter();

emitter.on("tell", (message) => {
  // this registers a listener
  console.log("listener 1 got a tell message:", message);
});

emitter.on("tell", (message) => {
  // listener 2.  You don't want too many in the chain
  console.log("listener 2 got a tell message:", message);
});

emitter.on("error", (error) => {
  // a listener for errors.  It's a good idea to have one per emitter
  console.log("The emitter reported an error.", error.message);
});

emitter.emit("tell", "Hi there!");
emitter.emit("tell", "second message");
emitter.emit("tell", "all done");
```

Try this program out, if you like.  One way is to start the Node command line and paste this in.  Of course, we only have "tell" and "error" as event names in this case, but typically there would be more named events.  The listeners for a given event are called in the order they register, and the emitting of events is synchronous, although it be made asynchronous.

This may seem pretty simple, and it is -- but it enables you to write programs where one function communicates with many others, depending on the conditions.  The mainline code could have logic that says, if X happens, notify a, b, and c, but if Y happens, notify c and d.  This gives plugpoints where developers can add modules to listen for events.  This simple model is exploited extensively in the Node `http` package.

Another package you should know about in Node is the `net` package.  This allows you to create, for example, server processes for which the protocol is not HTTP.  Mail servers, Domain Name Servers, whatever you like.  We won't do that in this class.

## **2.x A Simple HTTP Server**

The HTTP package allows you to create a server process that listens on a port.  Each time an HTTP request is received, the http server calls the callback you specify, passing a req object, which contains information from the request, and a res object, which gives you a way to send a response to the request.  Here is a sample.  Create a file in your `node-homework/assignment2` folder called `sampleHttp.js` with the following content. Then start the program in Node.

```js
const http = require('http');

const server = http.createServer({ keepAliveTimeout: 60000 }, (req, res) => {
    
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({
    data: 'Hello World!',
  }));
});

server.listen(8000);
```

You can access the server in your browser at `http://localhost:8000`.  However, it doesn't do too much, in that, no matter what request you send to the server, all you get back is "Hello World".  The callback you provide for `(req, res)`  is called for every incoming request.  That callback, in this case, calls two methods of the res object: `res.writeHead()`, which puts the HTTP result code and a header into the response, and `res.end()`, which sends the actual data.  You note that your Node program keeps running.  In Node, if there is an open network operation, or a promise being waited on, or pending file I/O, the Node process keeps running.  You stop it with Ctrl-C.

When the server gets an inbound HTTP request, it parses the basic information describing the request, including, in particular, the method, the url, the headers, and the cookies, and then issues the callback.  All of that information is in the req object -- but not the body of the request, if any body is present.  The data in the body is not available until you do another step.  Let's add some logic:

```js
const http = require("http");

const server = http.createServer({ keepAliveTimeout: 60000 }, (req, res) => {
  if (
    req.method === "POST" &&
    req.url === "/" &&
    req.headers["content-type"] === "application/json"
  ) {
    let body = "";
    req.on("data", (chunk) => (body += chunk)); // this is how you assemble the body.
    req.on("end", () => { // this event is emitted when the body is completely assembled.  If there isn't a body, it is emitted when the request arrives.
      const parsedBody = JSON.parse(body);
      res.writeHead(200, { "Content-Type": "application/json" });
      res.end(
        JSON.stringify({
          weReceived: parsedBody,
        }),
      );
    });
  } else if (req.method != "GET") {
    res.writeHead(404, { "Content-Type": "application/json" });
    res.end(
      JSON.stringify({
        message: "That route is not available.",
      }),
    );
  } else if (req.url.pathname === "/secret") {
    res.writeHead(200, { "Content-Type": "application/json" });
    res.end(
      JSON.stringify({
        message: "The secret word is 'Swordfish'.",
      }),
    );
  } else {
    res.writeHead(200, { "Content-Type": "application/json" });
    res.end(
      JSON.stringify({
        pathEntered: req.url,
      }),
    );
  }
});

server.listen(8000);
```

The idea, which we'll pursue further when we get to Express, is that we have a look at the request, and depending on what is in it, we send the appropriate response.  Here, the logic checks the method, which is one of GET, POST, PATCH, PUT, or DELETE, and the url.  In this case, the HTTP server provides not full URL but the URL path.  Depending on the values of the method and the path, different responses are returned.

If the server is running, stop it with a Ctrl-C, put in the code above, and restart it.  You can then try `http://localhost:8000/testPath` and `http://localhost:8000/secret` from your browser.  But, you also need to test the POST request.  For that you use Postman.

Start Postman, and click on the New button on the upper right.  Select New Http Request.  You will see a pulldown that defaults to "GET".  Switch it to "POST".  Then, where it says "Enter Request URL", put in `http://localhost:8000`.  Then click on the "Body" tab, and select "raw".  Then, in the pulldown that defaults to "Text", choose "JSON".  Then paste the following into the body:

```JSON
{
    "firstAttribute": "value1",
    "secondAttribute": 42
}
```

Then, click on the send button, and see what you get back.  You get a JSON response in this case.  If you change the URL to `http://localhost:8000/not-here`, you'll see you get back a 404.

This is one perfectly valid way of creating a web application back end.  Your server could respond with any kind of data, including HTML, and could handle a variety of incoming requests.  As no doubt you have noticed, this is still pretty low level.  You have to add logic for each URL path and method.  You have to read the body.  You have to parse the body.  You have to handle errors if any.  For example, the code above will crash the server if the incoming JSON is invalid, because the parse will throw an error.  Fortunately, the Express package makes your development task much easier.