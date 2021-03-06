[![Build Status](https://travis-ci.org/ifwe/monocle-api.png)](https://travis-ci.org/ifwe/monocle-api)

Monocle API Router for Connect
==============================

Monocle is a an API router that focuses on consistency, flexibility and performance.

## Features

### Client-driven

Only each client knows their own needs, so when communicating with the API, each client can specify the properties of a resource it is interested. Want the `user` resource without getting a ton of extra data that you'll never use? No problem!

### Schemas

API Router uses [JSON Schema](http://json-schema.org/) to configure and validate APIs. This encourages API consistency by validating return values and allows you to view the schema by appending `?schema` to any resource URL.

## Basic Usage

```js
var connect = require('connect');
var app = connect();
var Promise = require('bluebird');

// Allow parsing of JSON-encoded request body
var bodyParser = require('body-parser');
app.use(bodyParser.json());

// Create an API Router instance
var Monocle = require('../lib');
var api = new Monocle();

// For this simple demo we'll set up a simple in-memory data store for the user resource.
var user = {
    displayName: 'Alice',
    age: 27,
    gender: 'F'
};

// Configure your first API route
api.route(
    // Define the URL pattern for this resource
    '/user',

    // Define the schema for this resource. The schema will be shared across the supported HTTP methods.
    {
        type: 'object',
        properties: {
            displayName: { type: 'string' },
            age: { type: 'integer' },
            gender: { type: 'string' }
        }
    },

    // Define the HTTP methods that are supported by this url.
    {
        // Handle GET requests for this resource
        get: function(request) {
            return new Promise(function(resolve, reject) {
                if (!user) {
                    return reject("No user found.");
                }

                // Resolve promise with the user object and it will be converted to JSON automatically
                resolve(user);
            });
        },

        // Handle PUT requests for this resource
        put: function(request) {
            return new Promise(function(resolve, reject) {
                // Replace entire user object with provided resource, which is automatically JSON-decoded
                user = request.getResource();

                // Resolve promise with the updated user object
                resolve(user);
            });
        }
    }
);

// Add the API middleware to your connect app
app.use(api.middleware());

// Create web server and listen on port 5150
var http = require('http');
http.createServer(app).listen(5150, function() {
    console.log("Monocle API is now listening on port 5150");
});
```

Monocle API supports RESTful API calls:

```bash
$ curl -i http://127.0.0.1:5150/user
HTTP/1.1 200 OK
Content-Type: application/json
Date: Thu, 13 Aug 2015 17:50:29 GMT
Connection: keep-alive
Transfer-Encoding: chunked

{
  "displayName": "Alice",
  "age": 27,
  "gender": "F"
}

$ curl -X PUT -d '{"displayName": "Joe", "age": 42, "gender": "M"}' -i http://127.0.0.1:5150/user
HTTP/1.1 200 OK
Content-Type: application/json
Date: Thu, 13 Aug 2015 17:50:29 GMT
Connection: keep-alive
Transfer-Encoding: chunked

{
  "displayName": "Joe",
  "age": 42,
  "gender": "M"
}

$ curl -X POST -i http://127.0.0.1:5150/user
HTTP/1.1 404 Not Found
Content-Type: application/json
Date: Thu, 03 Sep 2015 18:28:10 GMT
Connection: keep-alive
Transfer-Encoding: chunked

{
  "error": "Not found",
  "exception": {
    "error": "No POST handler for /user"
  }
}

```

In addition, Monocle APIs provided flexibility by allowing the client to decide how much data to get back:

```bash
$ curl -i http://127.0.0.1:5150/user?props=displayName,age
HTTP/1.1 200 OK
Content-Type: application/json
Date: Thu, 13 Aug 2015 17:50:29 GMT
Connection: keep-alive
Transfer-Encoding: chunked

{
  "displayName": "Alice",
  "age": 27
}
```

View the details of any endoint by making an OPTIONS request.

```bash
$ curl -X OPTIONS http://127.0.0.1:5150/user
HTTP/1.1 200 OK
Content-Type: application/json
Date: Thu, 13 Aug 2015 20:47:46 GMT
Connection: keep-alive
Transfer-Encoding: chunked

{
  "pattern": "/user",
  "methods": [
    "GET",
    "PUT",
    "OPTIONS"
  ],
  "schema": {
    "type": "object",
    "properties": {
      "displayName": {
        "type": "string"
      },
      "age": {
        "type": "integer"
      },
      "gender": {
        "type": "string"
      }
    }
  }
}
```

See `demo/index.js` for advanced usage.

## Files and Directory Structure

The following describes the various files in this repo and the directory structure.

**Note:** Files and directories prefixed by `*` are auto-generated and excluded from the
repository via `.gitignore`.

    .
    ├── Gruntfile.js            # grunt task configuration
    ├── README.md               # this file
    ├── *docs                   # autogenerated documentation
    │   └── *index.html         # each JS file in `./lib` has a corresponding HTML file for documentation
    ├── lib                     # all code for this library will be placed here
    │   └── index.js            # main entry point for the API router
    ├── *node_modules           # all dependencies will be installed here by npm
    ├── package.json            # description of this package for npm, including dependency lists
    └── test                    # unit test configuration, reports, and specs
        ├── *coverage.html      # code coverage report
        ├── lib                 # specs go here with a 1:1 mapping to code in `./lib`
        │   └── index_test.js   # spec for `./lib/index.js`
        ├── mocha.opts          # runtime options for mocha
        └── test_runner.js      # configures mocha environment (e.g. chai, sinon, etc.)

## Development

### Grunt

Grunt is a JavaScript task runner to automate common actions. The API Router project
supports the following grunt tasks:

**test**

Runs all unit tests through mocha.

    $ grunt test

**coverage**

Runs all unit tests and generates a code coverage report in `./test/coverage.html`

    $ grunt coverage

**watch**

Automatically runs mocha tests each time a file changes in `./lib` or `./test`.

    $ grunt watch

**docs**

Generates documentation for all JS files within `./lib` using docco. Documentation is
written to `./docs`.

    $ grunt docs

**clean**

Deletes all auto-generated files, including `./docs` and `./test/coverage.html`

### Mocha, Sinon, Chai, Blanket

The ultimate TDD environment for node. Place your specs in `./test/lib`, and run `grunt test`.

See `./test/lib/index_test.js` for examples.
