# Introduction to Express

[Express](https://expressjs.com/) most popular Node web framework, and is the underlying library for a number of other popular [Node web frameworks](https://expressjs.com/en/resources/frameworks.html). It provides mechanisms to:

* Write handlers for requests with different HTTP verbs at different URL paths (routes).
* Integrate with "view" rendering engines in order to generate responses by inserting data into templates.
* Set common web application settings like the port to use for connecting, and the location of templates that are used for rendering the response.
* Add additional request processing "middleware" at any point within the request handling pipeline.

Developers have created compatible middleware packages to address almost any web development problem. There are libraries to work with cookies, sessions, user logins, URL parameters, POST data, security headers, and many more. You can find a list of middleware packages maintained by the Express team at Express [Middleware](http://expressjs.com/en/resources/middleware.html) (along with a list of some popular 3rd party packages).

## Express opionated?

Web frameworks often refer to themselves as "opinionated" or "unopinionated".

Opinionated frameworks are those with opinions about the "right way" to handle any particular task. They often support rapid development in a particular domain (solving problems of a particular type) because the right way to do anything is usually well-understood and well-documented. However they can be less flexible at solving problems outside their main domain, and tend to offer fewer choices for what components and approaches they can use.Eg. Ruby on Rails.

Unopinionated frameworks, by contrast, have far fewer restrictions on the best way to glue components together to achieve a goal, or even what components should be used. They make it easier for developers to use the most suitable tools to complete a particular task, albeit at the cost that you need to find those components yourself.

Express is unopinionated. You can insert almost any compatible middleware you like into the request handling chain, in almost any order you like. You can structure the app in one file or multiple files, and using any directory structure. You may sometimes feel that you have too many choices!

## Express Code

In a traditional data-driven website, a web application waits for HTTP requests from the web browser (or other client). When a request is received the application works out what action is needed based on the URL pattern and possibly associated information contained in POST data or GET data. Depending on what is required it may then read or write information from a database or perform other tasks required to satisfy the request. The application will then return a response to the web browser, often dynamically creating an HTML page for the browser to display by inserting the retrieved data into placeholders in an HTML template.

Express provides methods to specify what function is called for a particular HTTP verb (GET, POST, PUT, etc.) and URL pattern ("Route"), and methods to specify what template ("view") engine is used, where template files are located, and what template to use to render a response. You can use Express middleware to add support for cookies, sessions, and users, getting POST/GET parameters, etc. You can use any database mechanism supported by Node (Express does not define any database-related behaviour).

Create a text file in your project called "app.js" and add the content bellow.

```
const express = require('express')
const app = express()

//Creating route handler
app.get('/', (req, res) => res.send('Hello World!'))

app.listen(3000, () => console.log('Boom! listening on port 3000!'))
```

The first two lines require() (import) the express module and create an Express application. This object, which is traditionally named "app", has methods for routing HTTP requests, configuring middleware, rendering HTML views, registering a template engine, and modifying application settings that control how the application behaves (e.g. the environment mode, whether route definitions are case sensitive, etc.)

The middle part of the code (the three lines starting with app.get) shows a route definition. The app.get() method specifies a callback function that will be invoked whenever there is an HTTP GET request with a path ('/') relative to the site root. The callback function takes a request and a response object as arguments, and simply calls send() on the response to return the string "Hello World!"

The final block starts up the server on port '3000' and prints a log comment to the console. With the server running, you could go to localhost:3000 in your browser to see the example response returned.


## Routes
Routes allow you to match particular patterns of characters in a URL, and extract some values from the URL and pass them as parameters to the route handler (as attributes of the request object passed as a parameter).

Often it is useful to group route handlers for a particular part of a site together and access them using a common route-prefix (e.g. a site with a Wiki might have all wiki-related routes in one file and have them accessed with a route prefix of /wiki/). In Express this is achieved by using the express.Router object. For example, we can create our wiki route in a module named wiki.js, and then export the Router object, as shown below:

```
// wiki.js - Wiki route module

const express = require('express');
const router = express.Router();

// Home page route
router.get('/', (req, res) => res.send('Wiki home page'));

// About page route
router.get('/about', (req, res) => res.send('About this wiki') )

module.exports = router;
```



Note: Adding routes to the Router object is just like adding routes to the app object (as shown previously).

To use the router in our main app file we would then require() the route module (wiki.js), then call use() on the Express application to add the Router to the middleware handling path. The two routes will then be accessible from /wiki/ and /wiki/about/.

```
const wiki = require('./wiki.js');
// ...
app.use('/wiki', wiki);
```

## Middlewares

[Middleware](https://expressjs.com/en/guide/writing-middleware.html) is used extensively in Express apps, for tasks from serving static files to error handling, to compressing HTTP responses. Whereas route functions end the HTTP request-response cycle by returning some response to the HTTP client, middleware functions typically perform some operation on the request or response and then call the next function in the "stack", which might be more middleware or a route handler. The order that middleware is called is up to the app developer.

Example 1:
Here is a simple example of a middleware function called “myLogger”. This function just prints “LOGGED” when a request to the app passes through it. The middleware function is assigned to a variable named myLogger.

```
const myLogger = function (req, res, next) {
  console.log('LOGGED')
  next()
}

```

To load the middleware function, call app.use(), specifying the middleware function. For example, the following code loads the myLogger middleware function before the route to the root path (/).

```
const express = require('express')
const app = express()

const myLogger = function (req, res, next) {
  console.log('LOGGED')
  next()
}

app.use(myLogger)

app.get('/', (req, res) => res.send('Hello World!') )

app.listen(3000)

```

Every time the app receives a request, it prints the message “LOGGED” to the terminal.

The order of middleware loading is important: middleware functions that are loaded first are also executed first.

If myLogger is loaded after the route to the root path, the request never reaches it and the app doesn’t print “LOGGED”, because the route handler of the root path terminates the request-response cycle.

The middleware function myLogger simply prints a message, then passes on the request to the next middleware function in the stack by calling the next() function.

Example 2:
Next, we’ll create a middleware function called “requestTime” and add it as a property called requestTime to the request object.

```
const requestTime = function (req, res, next) {
  req.requestTime = Date.now()
  next()
}
```

The app now uses the requestTime middleware function. Also, the callback function of the root path route uses the property that the middleware function adds to req (the request object).

```
const express = require('express')
const app = express()

const requestTime = function (req, res, next) {
  req.requestTime = Date.now()
  next()
}

app.use(requestTime)

app.get('/', (req, res) => var responseText = 'Hello World!<br>' responseText += '<small>Requested at: ' + req.requestTime + '</small>' res.send(responseText) )

app.listen(3000)
```

When you make a request to the root of the app, the app now displays the timestamp of your request in the browser.

Because you have access to the request object, the response object, the next middleware function in the stack, and the whole Node.js API, the possibilities with middleware functions are endless.


## Serving static files

You can use the express.static middleware to serve static files, including your images, CSS and JavaScript (static() is the only middleware function that is actually part of Express). For example, you would use the line below to serve images, CSS files, and JavaScript files from a directory named 'public' at the same level as where you call node:

```
app.use(express.static('public'));
```

Any files in the public directory are served by adding their filename (relative to the base "public" directory) to the base URL. So for example:

```
http://localhost:3000/images/dog.jpg
http://localhost:3000/css/style.css
http://localhost:3000/js/app.js
http://localhost:3000/about.html
```

You can call static() multiple times to serve multiple directories. If a file cannot be found by one middleware function then it will simply be passed on to the subsequent middleware (the order that middleware is called is based on your declaration order).

```
app.use(express.static('public'));
app.use(express.static('media'));
```


You can also create a virtual prefix for your static URLs, rather than having the files added to the base URL. For example, here we specify a mount path so that the files are loaded with the prefix "/media":

```
app.use('/media', express.static('public'));
```

Now, you can load the files that are in the public directory from the /media path prefix.

```
http://localhost:3000/media/images/dog.jpg
http://localhost:3000/media/video/cat.mp4
http://localhost:3000/media/cry.mp3
```

### Error Handling

Errors are handled by one or more special middleware functions that have four arguments, instead of the usual three: (err, req, res, next). For example:

```
app.use(function(err, req, res, next) {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
```

These can return any content required, but must be called after all other app.use() and routes calls so that they are the last middleware in the request handling process!

Express comes with a built-in error handler, which takes care of any remaining errors that might be encountered in the app. This default error-handling middleware function is added at the end of the middleware function stack. If you pass an error to next() and you do not handle it in an error handler, it will be handled by the built-in error handler; the error will be written to the client with the stack trace.


### Rendering Data

Template engines (referred to as "view engines" by Express) allow you to specify the structure of an output document in a template, using placeholders for data that will be filled in when a page is generated. Templates are often used to create HTML, but can also create other types of documents. Express has support for a number of template engines, and there is a useful comparison of the more popular engines here: Comparing JavaScript Templating Engines: ejs, Jade, Mustache, Dust and More.

In your application settings code you set the template engine to use and the location where Express should look for templates using the 'views' and 'view engines' settings, as shown below (you will also have to install the package containing your template library too!)

```
const express = require('express');
const path = require('path');
const app = express();

// Set directory to contain the templates ('views')
app.set('views', path.join(__dirname, 'views'));

// Set view engine to use, in this case 'some_template_engine_name'
app.set('view engine', 'some_template_engine_name');
```

The appearance of the template will depend on what engine you use. Assuming that you have a template file named "index.<template_extension>" that contains placeholders for data variables named 'title' and "message", you would call Response.render() in a route handler function to create and send the HTML response:


```
app.get('/', (req, res) => res.render('index', { title: 'About dogs', message: 'Dogs rock!' }) )
```

### File Structure

Express makes no assumptions in terms of structure or what components you use. Routes, views, static files, and other application-specific logic can live in any number of files with any directory structure. While it is perfectly possible to have the whole Express application in one file, typically it makes sense to split your application into files based on function (e.g. account management, blogs, discussion boards) and architectural problem domain (e.g. model, view or controller if you happen to be using an [MVC](https://developer.mozilla.org/en-US/Apps/Fundamentals/Modern_web_app_architecture/MVC_architecture) architecture).
