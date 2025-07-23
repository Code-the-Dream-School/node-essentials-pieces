## **Debugging an Express Application**

There are various techniques to debug an Express application.

First, you should try to produce the error condition.  You do this one of two ways.  You may send a browser request to the application, in which case the network tab of browser developer tools will show you exactly what was sent and received for each request.  Or, you may use Postman to send requests, and Postman will show you exactly what was sent and received.

Second, if you have a hunch what is causing the error, you put console.log() or console.info() or console.debug() statements.  You would do this, in particular, if an error condition is happening in test or production that you can't duplicate.  When the tester or user triggers the error, your log may tell you what is going on.  You'll use this technique plenty in development too.

Third, you can write debugging middleware that can identify and log the error condition.  Your middleware function could report the content of the request, and perhaps some information about the response.  In some cases, you could insert a middleware function that is to be called after the middleware you want to debug, to see if that is working correctly.

Fourth, you could use the debugger built into VSCode.  

### **Using the VSCode Debugger**

1. Edit your node-homework `.vscode/launch.json`.  Here is one configuration you could use:

```js
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Express App",
      "runtimeExecutable": "nodemon",
      "runtimeArgs": [
        "--inspect-brk",
        "app.js"
      ],
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen",
      "restart": true,
      "env": {
        "NODE_ENV": "development",
        "PORT": "3000"
      }
    }
  ]
}
```

2. Edit a source code file, such as app.js.  There are line numbers for each line in the file.  If you click just to the left of the line number, a little red dot appears.  This is a breakpoint.  You can put them whereever you like in your code.  Click again to remove the breakpoint.

3. At the top of your VSCode window, there is a Run tab.  Click on this and select "start debugging".

Voilà.  Your program runs to the first breakpoint.  Your source file is displayed, and the program statement that is to run next is highlighted.  On the left side of your VSCode window, you see the debugger screen. The Variables section shows all the variables that are active in the context.  You can edit the values associated with each.  You can add variable names to the Watch section, so you can see how they change as you step through your program.  If you open the Call Stack section, you see the call stack.  An entry in the call stack will say `paused` or `paused on breakpoint`.  If you hover your mouse pointer over that, you see icons for continue, step over, step into, step out of, restart, and stop.  If the next line is a function call, step over runs the entire function call, step into goes into the function, and step out of runs all instructions until the return from the current function has completed.  If you do hit the continue button, it will run the program, and your Call Stack section will display a pause button that is usually ineffective.  You should first set a breakpoint before you hit continue, or add a breakpoint as it runs and then send a Postman request that will cause the breakpoint to be reached.  At the top of your VSCode window is a red square.  You click on that to end the debug session.

### **Sample Middleware for Debugging**

The following middleware function might be helpful.

```js
app.use((req, res, next) => {
    res.on('finish', () => {
// Here you'd log information that might be helpful to know about the req and/or the res.
    });
    next();
});
```
The "finish" event fires after the response has been sent to the requester.  You'd put this middleware function into the chain before any middleware function or request handler you are trying to debug.  You have to register the res.on() finish event callback before the response is actually sent, or it won't catch anything.

This middleware can log whatever is interesting in the req and res objects, with one exception.  You can get the headers and result code from the response, but you can't get the body of the response.  The body will already have been streamed to the network in response to the request.