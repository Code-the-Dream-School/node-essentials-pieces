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

  There is another approach.  Instead of creating promises, you create async functions that do the callbacks, and that return when the callback completes, as follows:

  ```js
  const asyncFsOpen = async (...args) =>{
    return new Promise((resolve, reject) => {
      fn(...args, (err, result) => {
        if (err) return reject(err);
        resolve(result);
      });
    });
```
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