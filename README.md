# Router middleware for [koa](https://github.com/koajs/koa)

[![Build Status](https://secure.travis-ci.org/alexmingoia/koa-router.png)](http://travis-ci.org/alexmingoia/koa-router) 
[![Dependency Status](https://david-dm.org/alexmingoia/koa-router.png)](http://david-dm.org/alexmingoia/koa-router)

* REST routing using `app.get`, `app.put`, `app.post`, etc.
* Rails-like resource routing, with nested resources.
* Named parameters.
* Multiple route callbacks.
* Multiple routers.

## Install

koa-router is available using [npm](https://npmjs.org):

```
npm install --global koa-router
```

## Usage

Require the router and mount the middleware:

```javascript
var koa = require('koa')
  , router = require('koa-router')
  , app = koa();

app.use(router(app));
```

After the router has been initialized, you can register routes or resources:

```javascript
app.get('/users/:id', function *(id) {
  var user = yield User.findOne(id);
  this.body = user;
});

app.resource('forums', require('./controllers/forums'));
```

You can use multiple routers and sets of routes by omitting the `app`
argument. For example, separate routers for two versions of an API:

```javascript
var APIv1 = new Router();
var APIv2 = new Router();

APIv1.get('/sign-in', function *() {
  // ...
});

APIv2.get('/sign-in', function *() {
  // ...
});

app.use(mount('/v1', APIv1.middleware()));
app.use(mount('/v2', APIv2.middleware()));
```

### app.verb(path, callback, [callback...])

Match URL patterns to callback functions or controller actions using `app.verb()`,
where **verb** is one of the HTTP verbs such as `app.get()` or `app.post()`.

```javascript
app.get('/', function *(next) {
  this.body = 'Hello World!';
});
```

Route paths will be translated to regular expressions used to match requests.
Query strings will not be considered when matching requests.

Multiple callbacks may be given, and each one will be called sequentially:

```javascript
app.get(
  '/users/:id',
  function *(id) {
    user = yield User.findOne(id);
    return [user];
  }, function *(user) {
    console.log(user);
    // => { id: 17, name: "Alex" }
  }
);
```

You can modify the route parameters for subsequent callbacks by returning an
array of arguments to apply, as shown in the example above.

#### Named parameters

Named route parameters are captured and passed as arguments to the route callback.
They are also available in the app context using `this.params`.

```javascript
app.get('/:category/:title', function *(category, title, next) {
  console.log(this.params);
  // => { category: 'programming', title: 'how-to-node' }
});
```

#### Regular expressions

Control route matching exactly by specifying a regular expression instead of
a path string when creating the route. For example, it might be useful to match
date formats for a blog, such as `/blog/2013-09-04`:

```javascript
app.get(/^\/blog\/\d{4}-\d{2}-\d{2}\/?$/i, function *(next) {
  // ...
});
```

#### Multiple methods

You can map routes to multiple HTTP methods using `app.map()`:

```javascript
app.map(['GET', 'POST'], '/', function *(next) {
  // ...
});
```

You can map to all methods use `app.all()`:

```javascript
app.all('/', function *(next) {
  // ...
});
```

### app.resource(path, actions)

Resource routing is provided by the `app.resource()` method. `app.resource()`
registers routes for corresponding controller actions, and returns a
`Resource` object that can be used to further nest resources.

```javascript
var app    = require('koa')()
  , router = require('koa-router')(app);

app.use(router);

app.resource('users', require('./user'));
```

#### Action mapping

Actions are then mapped accordingly:

```javascript
GET     /users             ->  index
GET     /users/new         ->  new
POST    /users             ->  create
GET     /users/:user       ->  show
GET     /users/:user/edit  ->  edit
PUT     /users/:user       ->  update
DELETE  /users/:user       ->  destroy
```

#### Top-level resource

Omit the resource name to specify a top-level resource:

```javascript
app.resource(require('./frontpage'));
```

Top-level controller actions are mapped as follows:

```javascript
GET     /          ->  index
GET     /new       ->  new
POST    /          ->  create
GET     /:id       ->  show
GET     /:id/edit  ->  edit
PUT     /:id       ->  update
DELETE  /:id       ->  destroy
```

#### Auto-loading

Automatically load requested resources by specifying the `load` action
on your controller:

```javascript
var actions = {
  show: function *(user) {
    this.body = user;
  },
  load: function *(id) {
    return users[id];
  }
};

app.resource('users', actions);
```

The `user` object will then be available to the relevant controller actions.
You can also pass the load method as an option:

```javascript
app.resource('users', require('./users'), { load: User.findOne });
```

#### Nesting

Resources can be nested using `resource.add()`:

```javascript
var forums = app.resource('forums', require('./forum'), { load: Forum.findOne });
var theads = app.resource('threads', require('./threads'), { load: Thread.findOne });

forums.add(threads);
```

### app.redirect(path, destination, [code])

Redirect `path` to `destination` URL with optional 30x status `code`.

```javascript
app.redirect('/login', 'sign-in');
```

This is equivalent to:

```javascript
app.all('/login', function *() {
  this.redirect('/sign-in');
  this.status = 301;
});
```

## Tests

Tests use [mocha](https://github.com/visionmedia/mocha) and can be run 
with [npm](https://npmjs.org):

```
npm test
```

## License

The MIT License (MIT)

Copyright (c) 2013 Alexander C. Mingoia

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
