# **Lesson 1 — Introduction to Node**

## **Lesson Overview**

**Learning objective**: Students will gain foundational knowledge of the Node JavaScript environment.  They will learn about the Node event loop.  Students will gain some familiarity with the Node capabilities that are not present in browser side JavaScript, such as file system access and HTTP servers.  A referesher on asynchronous programming is included.

**Topics**:

1. What is Node?

## **4.1 What is Node**

The JavaScript language was created to run inside the browser, so as to create rich and responsive web applications.  It is an interpreted language that is platform independent, running on Mac, Linux, Windows, and other platforms.  Browser side JavaScript runs in a sandbox.  The code you load from the Internet, can't be trusted, so there are things that JavaScript isn't allowed to do when running in the browser, like accessing the local file system or opening a server side socket. To provide security protections, browser side JavaScript runs in a sandbox, a protected area that blocks off various functions.

A while back, some folks at Google had the thought: There are a lot of JavaScript programmers.  Wouldn't it be nice if they could write server side code in JavaScript? Wouldn't it be nice if they could develop web application servers in JavaScript?  At the time, the other leading platform independent language was Java, a more complicated language.

So, they created a version of JavaScript that runs locally on any machine, instead of in the browser. This version of the code has no sandbox. In Node, you can do anything that one could do in other programming languages, subject only to the security protections provided by the operating system.

But, there was a problem. JavaScript is single threaded. So, if code for a web application server were doing an operation that takes time: file system access, reading stuff from a network connection, accessing a database, and so on, all the web requests would have to wait.  The solution is the event loop.  If a "slow" operation is to be performed, the code makes the request, but does not wait for it to complete. Instead, it provides a callback that is called when the operation does complete, and continues on to do other stuff that doesn't depend on the outcome of the request.  There is a good video that gives the details here: [What the heck is the event loop?](https://www.youtube.com/watch?v=8aGhZQkoFbQ).  That video may give more information than you want or need now, but check it out at your leisure.  The stuff is good to know.  When you call an asynchronous API in the event loop, you are calling into an environment that is written in C++, and that environment **is** multithreaded.  Your request is picked up from the queue, and handed off to a thread, and that does the work, perhaps blocking for a time, but eventually issuing the callback. The event loop is present in both browser based JavaScript and Node, but there are more capabilities in the Node version.

Because of this approach, web application servers written in JavaScript are faster than those written in Python or Ruby. Those languages don't do as much in native code, at least for asynchronous operations, so web application servers in these languages have to be multithreaded, at a significant performance cost.  Web application servers in Node are pretty fast, though not quite as fast as those in Java or C++.  There are still some kinds of functions for which JavaScript is not a good choice, such as intensive numerical calculations.

## **4.2 Running Node**

At your terminal, type `Node`.  (You should have completed the setup assignment.  If this command doesn't do anything, go back and do the setup.)  This starts the environment, and you can enter and run JavaScript statements.  Try a console.log().  You may notice one difference from the browser environment.  Where does the output appear?  It appears in your terminal.  Obvious, right?  If you open up the console in your browser developer tools, you will not see the output for Node console.log() statements.  We have had some folks fresh from the React class who develop a Node/Express application, put console.log() debugging statements in, and go to the browser console to see them.

You should have set up your `node-homework` repository and folder.  In the terminal, cd to that folder and start VSCode.  Create a first.js file in the assignment1 directory, with a console.log() statement in it.  Start a VSCode terminal, and type "node ./assignment1/first".  You do not have to give the `.js` extension.  Ok, so much for the very simple stuff.

## **4.3 File System Access with Async Operations**

As we've said, Node let's you access the file system.  The functions you use are documented here: [https://nodejs.org/api/fs.html](https://nodejs.org/api/fs.html).  Create a file, './assignment1/fileAccess.js`.  It should read as follows:

```js
const fs = require("fs");

fs.open("./tmp/file.txt", "w", (err, fileHandle) => {
  if (err) {
    console.log("file open failed: ", err.message);
  } else {
    console.log("file open succeeded.  The file handle is: ", fileHandle);
  }
});
console.log("last statement");
```

Now, before you run this file, consider: What will be the last sentence written to the console?  Ok, now run this program.  As you observe, the "file open succeeded" happens after the "last statement".  Do you understand why?  The fs.open() just queues the request.  Then the "last statement" is printed.  Then the node event loop finishes the file open and calls the callback.  Then the "file open succeeded" happens.  Ok, now add a statement to write a line to the file.  You use:

```js
fs.write(fileHandle, "first string to write\n", (err, bytesWritten)=> {

});
```

Where in your program do put this statement?  Put some console.log() statements into the callback so that you know what happened.  Then try it out.  Also have a look at `./tmp/file.txt` after the program runs.  Then add another statement to write another line after this one, with a different string.  Then a statement to close the file.  You use:

```js
fs.close(fileHandle, (err) =>{
  console.log("error on file close: ", err.message);
});
```

Then try the program, and have a look at the file you have written.  Does all look correct?  Are the events happening in the order you want?  Each successive operation on the file must occur in the callback for the previous operation.

If you had to make 100 separate writes to the program, the appearance would be messy.  This is "callback hell".  In the old days, people would do some clever stuff with recursion to keep their code legible. But, as you know, now we have promises.  There are two solutions built into the Node fs package for the issue.  First, you could fs.openSync() and other synchronous APIs.  This is a bad idea for an Express program, but it might be reasonable if you are doing some scripting.  Second, you could use the promises based version of the Node filesystem APIs.  We'll do that in a minute.  But, some APIs, such as setTimeout(), don't support promises.  We'll use such a package later in the course.  You need to learn how to wrapper callback APIs with promises.  Create filePromise1.js in your assignment1 folder, and put in the following:

```js
const fs = require("fs");

const fileOps = async () => {
  let promise = new Promise((resolve, reject) => {
    fs.open("./tmp/file.txt", "w", (err, fileHandle) =>
      err ? reject(err) : resolve(fileHandle),
    );
  });
  const fileHandle = await promise;
  promise = new Promise((resolve, reject) => {
    fs.write(fileHandle, "first string to write\n", (err, bytesWritten) =>
      err ? reject(err) : resolve(bytesWritten),
    );
  });
  let bytesWritten = await promise;
  console.log(bytesWritten, " bytes written.");
  promise = new Promise((resolve, reject) => {
    fs.write(fileHandle, "second string to write\n", (err, bytesWritten) =>
      err ? reject(err) : resolve(bytesWritten),
    );
  });
  bytesWritten = await promise;
  console.log(bytesWritten, " bytes written.");
  promise = new Promise((resolve, reject) => {
    fs.close(fileHandle, (err) => {
      if (err) return reject(err);
      resolve();
    });
  });
  await promise;
};

try {
  fileOps();
} catch (err) {
  console.log("error occurred: ", err.message);
}
```

Then try it out.  It isn't exactly terse, but there is no messy nesting of callbacks. **You do need to understand how to do the wrappering described.**  Look carefully at the pattern.  It isn't very hard.  You just have to use the resolve and reject provided when you create a promise in the callback. 

To use the await keyword, You have to create an async function, because you can't put an await statement into the mainline of a Node program.  You also have to have a try/catch, in case there are errors.

There is another style for this:

```js
new Promise((resolve, reject) => {
  fs.open("./tmp/file.txt", "w", (err, fileHandle) =>
    err ? reject(err) : resolve(fileHandle),
  );
})
  .then((fileHandle) => {
    new Promise((resolve, reject) => {
      fs.write(fileHandle, "first string to write\n", (err, bytesWritten) =>
        err ? reject(err) : resolve(bytesWritten),
      );
    }).then((bytesWritten) => {
      console.log(bytesWritten, " bytes written.");
      new Promise((resolve, reject) => {
        fs.write(fileHandle, "second string to write\n", (err, bytesWritten) =>
          err ? reject(err) : resolve(bytesWritten),
        );
      }).then((bytesWritten) => {
        console.log(bytesWritten, " bytes written.");
        new Promise((resolve, reject) => {
          fs.close(fileHandle, (err) => {
            if (err) return reject(err);
            resolve();
          });
        });
      });
    });
  })
  .catch((err) => {
    console.log("An error occurred: ", err.message);
  });
  ```
  This might be called "then hell".  Don't use then!

  Fortunately, most APIs you use these days do support promises, including the file system APIs.  So, you can do:

  ```js
const fs = require("fs/promises");

const fileOps = async () => {
  const fileHandle = await fs.open("./tmp/file.txt", "w");
  let { bytesWritten } = await fileHandle.write("first string to write\n");
  console.log(bytesWritten, " bytes written.");
  ({ bytesWritten }= await fileHandle.write("second string to write\n"));
  console.log(bytesWritten, " bytes written.");
  await fileHandle.close();
};

try {
  fileOps();
} catch (err) {
  console.log("error occurred: ", err.message);
}
```

Ok, that's a little more civilized.

## **More on Async Functions**

In your `node-homework/assignment1` folder are two programs called `callsync.js` and `callsync2.js`.  Here is the first of these:

```js
function syncfunc() {
    console.log("In syncfunc.  No async operations here.")
    return "Returned from syncfunc."
}

async function asyncCaller() {
    console.log("About to wait.")
    const res = await syncfunc()
    console.log(res)
    return "asyncCaller complete."
}

console.log("Calling asyncCaller.")
const r = asyncCaller()
console.log(`Got back a value from asyncCaller of type ${typeof r}`)
if (typeof r == "object") {
    console.log(`That object is of class ${r.constructor.name}`)
}
r.then(resolvesTo => {
    console.log("The promise resolves to: ", resolvesTo)
})
console.log("Finished.")
```

See if you can guess which order the statements will appear in the log.  Then run the program.  Were you right?

The asyncCaller() method calls syncfunc(), which is a synchronous function, with an await.  This is valid, and the await statement returns the value that syncfunc() returns.  But asyncCaller() is an async function, so it returns to the mainline code at the time of the first await statement, and what it returns is a promise.  Processing continues in asyncCaller only after the `Finished` statement appears, because that is the point at which the event loop gets a chance to return from await.  The subsequent `return` statement in asyncCaller is different from a `return` in a synchronous function.  A return statement in an async function resolves the promise returned by the async function to a value.  

There are two ways to get the value a promise resolves to, those being, `await` and `.then`.  We can't do `await` in mainline code, so we use `.then` in this case.  The `.then` statement provides a callback that retrieves the value.  But the mainline code doesn't wait for the `.then` callback to complete.  Instead it announces `Finished.`, and only when the callback completes do we see what the promise resolves to.

The other program, `callsync2.js`, is slightly different, and if you run it, you see that the order the logged statements appear in is slightly different.  The only functional change is that asyncCaller() does not do an await when it calls syncfunc(), so it runs all the way to the end before the mainline code resumes.  But what asyncCaller returns is still a promise, and the `.then` for that promise still doesn't complete until after the mainline code reports the `Finished.`  Run this program to see the difference.  Async functions always return a promise.  You can use the `await` statement to call a synchronous function, or to resolve a promise, or to resolve a `thenable` which is an object that works like a promise, but may have additional capabilities.

## **Exiting Cleanly**

Your Express program opens a port.  You need to be sure that port is closed when the program exits.  If there are other open connections, such as database connections, they must also be cleaned up.  If not, you may find that you program becomes a zombie process, and that the port you had been listening on is still tied up.  This is especially important when you are running a debugger or an automated test.  Here is some code to put at the bottom of app.js to ensure a clean exit.

```js
let isShuttingDown = false;
async function shutdown() {
  if (isShuttingDown) return;
  isShuttingDown = true;
  console.log('Shutting down gracefully...');
  // Here add code as needed to disconnect gracefully from the database
};

process.on('SIGINT', shutdown);
process.on('SIGTERM', shutdown);
process.on('uncaughtException', (err) => {
  console.error('Uncaught exception:', err);
  shutdown();
});
process.on('unhandledRejection', (reason) => {
  console.error('Unhandled rejection:', reason);
  shutdown();
});