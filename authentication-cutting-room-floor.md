
At this point, a unit test might be nice.  But, hmm, we have callbacks all over the place.  Here is a unit test you might try, if you paste it into a temporary javascript file:

```js
require("dotenv").config();  // we need the JWT secret
require("./passport/passport"); // This configures the Passport strategies
const passport = require("passport");
const jwt = require("jsonwebtoken");

let myResolve;
let promise;
let req;

const errorReporter = (e) => {
  if (e) {
    console.log("error: ", e.message);
  }
};
const reportPassportResult = (err, user) => {
  if (err) {
    console.log("error returned", err.message);
  } else if (user) {
    console.log("user returned", JSON.stringify(user));
  } else {
    console.log("authentication failed, we better send a 401");
  }
  myResolve();
};

const theTest = async () => {
  console.log("for this test, passport should report back with the user info");
  promise = new Promise((resolve) => {
    myResolve = resolve;
  });
  req = { body: { email: "jim@sample.com", password: "magic" } };

  passport.authenticate("local", reportPassportResult)(req, {}, errorReporter);
  await promise;

  console.log("for this test, passport should report that authencation failed");
  promise = new Promise((resolve) => {
    myResolve = resolve;
  });
  req = { body: { email: "jim@sample.com", password: "notMagic" } };
  passport.authenticate("local", reportPassportResult)(req, {}, errorReporter);
  await promise;

  console.log("for this test, passport should return the user info");
  let token = jwt.sign(
    { name: "Frank", email: "frank@sample.com" },
    process.env.JWT_SECRET,
    { expiresIn: "1h" },
  );
  req = { cookies: { jwt: token } };
  promise = new Promise((resolve) => {
    myResolve = resolve;
  });
  passport.authenticate("jwt", reportPassportResult)(req, {}, errorReporter);
  await promise;

  console.log(
    "for this test, passport should report that authentication failed",
  );
  token = jwt.sign(
    { name: "Frank", email: "frank@sample.com" },
    "wrongsecret",
    { expiresIn: "1h" },
  );
  promise = new Promise((resolve) => {
    myResolve = resolve;
  });
  req = { cookies: { jwt: token } };
  passport.authenticate("jwt", reportPassportResult)(req, {}, errorReporter);
  await promise;
};

try {
  theTest();
} catch (e) {
  console.log("error occurred: ", e.message);
}
```

Yeah, a little long for a unit test.  You can try it if you like. **This code is important because it shows how Passport works.**  Each time we call passport, we have to provide a callback.  The myPassportCallback is the one we use for the test, because it gives the results that Passport returns.  We want to wait on the completion of each test.  So we create a Promise each time, and the myPassportCallback resolves the promise when it completes.  Then we can just wait on the promise.  
