# **Assignment Event Handlers and HTTP Servers**

## **Assignment Instructions**

Your assignment files are to be created in the assignment2 folder of your node-homework folder.  Create an assignment2 git branch before you start.

### **Task 1: Practice With An Event Emitter and Listener**

Create a file called events.js.  Create an emitter.  Use an emmitter.on() statement to listen to this emitter for the 'time' event.  Whenever the listener receives the event, it should print out "Time received: " followed by the string it receives.  Then, call `setInterval(callback, 5000)`.  Your callback for the setInterval should emit a 'time' message with the current time as a string.  Try it out.  You use Ctrl-C to end the program.

### **Task 2: Practice with the HTTP Server**

Modify your sampleHTTP.js.  First, add the following to the top of your file:

```js
const htmlString = `
<!DOCTYPE html>
<html>
<body>
<h1>Clock</h1>
<button id="getTimeBtn">Get the Time</button>
<p id="time"></p>
<script>
document.getElementById('getTimeBtn').addEventListener('click', async () => {
    const res = await fetch('/time');
    const timeObj = await res.json();
    console.log(timeObj);
    const timeP = document.getElementById('time');
    timeP.textContent = timeObj.time;
});
</script>
</body>
</html>
`;
```

This, of course, is a web page.  Then change your logic so that you handle requests for the url "/time.  A JSON document should be returned with an attribute "time" that has a value of the current time as a string.  Once you've got that code in place, restart your server and test the new URL from your browser.  Then, add logic to handle requests for "/timePage".  It should return the page above.  You will need to set the header content-type to be: "text/html; charset=utf-8".  Then restart your server and try that URL.  You should see the page, with a button.  Click on the button, and you should see the time.  The button causes a fetch to your server.  You have now coded your first REST API.

### **Task 3: Add a Post Route Handler**

Modify your app.js.  Add an app.post() for "/testpost".  Send something back.  Then test this new route handler using Postman.  You do not need to put anything in the body.

### **Task 4: Add Logging Middleware**

Modify your Express app.js.  Add an app.use() statement, above your other routes.  The middleware function you add should do a console.log() of the req.method, the req.path, and the req.query.  Don't forget to call next(), as this is middleware.  Then start the server and try sending various requests to your server from your browser, to see the log messages.  You could also try post requests using Postman.  The `req.query` object gives the query parameters.  You can add them into your request from your browser or from Postman by putting something like `?height=7&color=brown` to the end of the URL you send from your browser or Postman.  This middleware is useful -- it could help with debugging.  We haven't explained how to get req.body, but eventually you could get that as well.

There is no TDD for this week.

📌 **Follow these steps to submit your work:**

#### **1️⃣ Add, Commit, and Push Your Changes**

- Within your node-homework folder, do a git add and a git commit for the files you have created, so that they are added to the `assignment2` branch.
- Push that branch to GitHub.

#### **2️⃣ Create a Pull Request**

- Log on to your GitHub account.
- Open your `node-homework` repository.
- Select your `assignment2` branch. It should be one or several commits ahead of your main branch.
- Create a pull request.

#### **3️⃣ Submit Your GitHub Link**

- Your browser now has the link to your pull request. Copy that link, to be included in your homework submission form.