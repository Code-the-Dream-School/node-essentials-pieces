## **REST Concepts**

The back end application you are building implements a protocol called REST.  You need to understand how REST works, and it helps to understand how the Internet works.

### **Internet Foundations**

All Internet traffic is based on layers of protocols.  The Internet runs on IP: Internet Protocol.  When data is sent over the Internet, it is broken up into packets.  Each packet has a source address, a destination address, a protocol and a port.  The addresses are four part numbers like 9.28.147.56.  The protocol is also a number, indicating what kind of packet it is, and the port is also a number, which endpoints use to figure out which process on a machine should get the packet.  A network of routers figures out where the destination machine is and forwards the packet.  For REST, you will use a protocol called TCP, which stands for Transmission Control Protocol.  TCP has reliable connections, where "reliable" means that each TCP endpoint sends acknowledgements when a packet arrives, and if the acknowledgement is slow in arriving, the source machine sends the packet again.  Retries continue until the acknowledgement arrives or a timeout occurs.  TCP connections have a server, which listens for connection requests, and a client, which initiates them.  Your browser is a client.  Servers and clients communicate over TCP using a programming interface called sockets, but sockets are just a programming interface of the operating system. Servers typically have a DNS name, like www.widgets.com.  DNS stands for Domain Name Service, and a network of domain name servers keep track of the names so that each endpoint can look them up.  TCP connections can be augmented with SSL, which stands for Secure Sockets Layer.  There are two advantages to SSL.  First, all the data sent by either end of the the connection is encrypted so that no one can listen in.  Second, when the SSL connection is established, the server proves, by means of a cryptographic exchange, that it really is www.widgets.com, and not some impostor.

Remember all of this.  It might come up during trivia night at your local bar.

### **HTTP**

REST requests flow over a protocol called HTTP, which stands for Hypertext Transfer Protocol.  HTTP over an SSL connection is called HTTPS.  Each HTTP request uses one of a small number of methods.

- GET
- POST
- PUT
- PATCH
- DELETE
- HEAD
- OPTIONS
- CONNECT
- TRACE

For REST requests, GET, POST, PUT, PATCH, and DELETE are used.  Each HTTP request or response packet has various parts, most of which are optional.

- A path, like "/info/dogs", which comes from the URL
- Query parameters, like "name="Spot".  These also come from the URL, for example as in "http://info/dogs?name=Spot&owner=Frank.  These are often used in REST requests.
- Headers.  These are key-value pairs.  For REST requests, the "Content-Type" header is always used, and it typicallyoften has the value "application/json".  There are many other headers.
- A body.  POST, PUT and PATCH requests often have a body. Responses for each of the methods also often have a body.  For REST, the body is usually JSON.  By convention, POST operations are used to create some data on the back end, PATCH to update that data, and PUT to replace that data.  Never use GET requests to change data!

Pay attention to this part.  You will use all the REST operations.  You will need to specify the path, the query parameters, a header, JSON in the body, and even a cookie, to complete your final project.

### **Stuff the Browser Keeps Track Of**

The browser keeps track of the origin for a request.  This is the address and port for the URL.  When a browser application makes a REST request, that may go to the origin of the application itself, or to a different origin.  If it goes to a different origin, that's a cross origin request.

The browser also keeps track of cookies, which are key-value pairs.  These are set because a Set-Cookie header was sent in a server response, and they store data, typically small amounts, but certainly less than 4k.  They also have various flags.  Browsers have policies for which Set-Cookie headers will be honored, and silently discard the rest.  Cookies can be sent on subsequent requests from the browser application, depending on the request and also on the browser policies, until such time as the cookie expires.  Cookie content is available to the JavaScript in the browser application, unless it is an HttpOnly cookie.  A server sets an HttpOnly cookie to store information about the client, such as whether the user has logged on and who that user is.  Cookies are not usually used unless the requesting application is running in a browser.

### **REST and JSON**

REST stands for Representational State Transfer, which means the exchange of HTTP requests and responses, where the management of the state of the conversation and the security governing the exchange is not a part of the REST protocol itself.  JSON stands for JavaScript Object Notation.  You need to know JSON.  Here is a [video introduction to JSON](https://www.youtube.com/watch?v=iiADhChRriM). Below is a summary.

The types in JSON are:

- Number, either an integer or a decimal.
- String, always in Unicode
- Boolean, either true or false
- Array, which is an ordered list of any of these data types.
- Object, a collection of name-value pairs.  The names are always strings.
- null.

You can put objects in objects, objects in arrays, arrays in objects, and so on, nesting as much as you like, so as to make the document as complicated as necessary.  A JSON document must either be an object or an array.  Here is an example from Wikipedia:

```JSON
{
  "first_name": "John",
  "last_name": "Smith",
  "is_alive": true,
  "age": 27,
  "address": {
    "street_address": "21 2nd Street",
    "city": "New York",
    "state": "NY",
    "postal_code": "10021-3100"
  },
  "phone_numbers": [
    {
      "type": "home",
      "number": "212 555-1234"
    },
    {
      "type": "office",
      "number": "646 555-4567"
    }
  ],
  "children": [
    "Catherine",
    "Thomas",
    "Trevor"
  ],
  "spouse": null
}
```

A JSON document is not a JavaScript object.  The keys in a JSON object are always strings in double quotes.  The string values in a JSON object are always specified in double quotes.  In JavaScript, a JSON object is just a string.  The following JavaScript functions are often used with JSON:

```js
const anObject = JSON.parse(aString);  // convert from a JSON string to a JavaScript object.
const aJSONString = JSON.stringify(anObject);  // convert from a JavaScript object to a JSON string.
```

Of course, not all JavaScript objects can be converted to JSON.  If the object contains functions, for example, they are omitted from the resulting JSON string, and this also happens with other JavaScript types.

Binary objects like JPEGs are never sent in JSON.  You can still do a REST request, but you use a different content type.

JSON objects can be parsed or created in any modern computer programming language: Python, Java, Rust, C++, etc..  In some NoSQL databases like MongoDB, every entry in the database is basically a JSON object.