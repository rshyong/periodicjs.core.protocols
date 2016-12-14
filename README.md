# periodicjs.core.protocols
[![Build Status](https://travis-ci.org/typesettin/periodicjs.core.protocols.svg?branch=master)](https://travis-ci.org/typesettin/periodicjs.core.protocols) [![NPM version](https://badge.fury.io/js/periodicjs.core.protocols.svg)](http://badge.fury.io/js/periodicjs.core.protocols) [![Coverage Status](https://coveralls.io/repos/github/typesettin/periodicjs.core.protocols/badge.svg?branch=master)](https://coveralls.io/github/typesettin/periodicjs.core.protocols?branch=master)  [![Join the chat at https://gitter.im/typesettin/periodicjs.core.data](https://badges.gitter.im/typesettin/periodicjs.core.protocols.svg)](https://gitter.im/typesettin/periodicjs.core.protocols?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)


### Description
Core protocols is a component of periodicjs.core.controller that exposes a standard set of methods across different transport protcols with options for implementing different API strategies.  With core protocols implementing a RESTful API for HTTP requests looks no different than standing up a JSONRPC API for websocket based communications.

### [Full Documentation](https://github.com/typesettin/periodicjs.core.protocols/blob/master/doc/api.md)

### Usage (basic)
```javascript
const express = require('express');
const mongoose = require('mongoose');
const ProtocolInterface = require('periodicjs.core.protocols');
const DBInterface = require('periodicjs.core.data');
const ResponderInterface = require('periodicjs.core.responder');

//Connect to the DB and start a HTTP server
mongoose.connect();
mongoose.connection.once('open', () => {
  //Get the registered model or register a model
  let Example = mongoose.model('Example');
  let Server = express().listen(3000);
  //This configuration will allow for RESTful routes to be generated and mounted for the "Example" mongo collection and will by default respond in JSON
  let http_adapter = ProtocolInterface.create({
    express: Server,
    resources: {},
    db: {
      example: DBInterface.create({ adapter: 'mongo', model: Example });
    },
    responder: ResponderInterface.create({ adapter: 'json' }),
    api: 'rest',
    adapter: 'http'
  });
  //The implement method called with no arguments will mount a sub-router for each db adapter indexed by model name in the db object
  http_adapter.implement();
  //http adapter will now have a .router if you wish to access the express router directly
});
/*
  With a JSON responder requests will by default receive a JSON response unless it requires a view to be rendered
  Implemented RESTful routes
  GET localhost:3000/example => Get All
  POST localhost:3000/example => Create One
  GET localhost:3000/example/:id => Get One
  PUT localhost:3000/example/:id => Update One
  DELETE localhost:3000/example/:id => Remove One
 */
```
### Usage (advanced)
```javascript
//In this example assume that mongo is already connected and the HTTP server is already started
let http_adapter = ProtocolInterface.create({
  express: require('express'), //Instead of the running server provide the express module
  resources: {},
  db: {
    example: DBInterface.create({ adapter: 'mongo', model: Example })
  },
  responder: ResponderInterface.create({ adapter: 'json' }),
  api: 'rest',
  adapter: 'http'
});
//This implements a router specifically for the example model. The .dirname option specifies a view directory if your view files are not in one of the default directories
let router = http_adapter.api.implement({ model_name: 'example', dirname: ['./some/path/to/view/dir'] });
//Because this returns a router you must manually mount it on the applications main router this does however allow for more control over the path
Server.use('/api/v1', router);
/*
  Once again the JSON responder will by default be used unless a view must be rendered. If .strict option is passed when constructing the protocol adapter or in the this.api.implement call or this.implement call all responses will come a JSON. Additionally, the JSON responder will be used if req.query.format = "json"
  Implemented RESTful routes
  GET localhost:3000/api/v1/example?format=json => Get All
  POST localhost:3000/api/v1/example?format=json => Create One
  GET localhost:3000/api/v1/example/:id?format=json => Get One
  PUT localhost:3000/api/v1/example/:id?format=json => Update One
  DELETE localhost:3000/api/v1/example/:id?format=json => Remove One
 */
```

### Development
*Make sure you have grunt installed*
```sh
$ npm install -g grunt-cli jsdoc-to-markdown
```

For generating documentation
```sh
$ grunt doc
$ jsdoc2md adapters/**/*.js api_adapters/**/*.js utility/**/*.js index.js > doc/api.md
```
### Notes
* Check out [https://github.com/typesettin/periodicjs](https://github.com/typesettin/periodicjs) for the full Periodic Documentation

### Testing
```sh
$ npm i
$ grunt test
```
### Contributing
License
----

MIT
