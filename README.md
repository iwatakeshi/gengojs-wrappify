# gengojs-wrappify
An Express, Hapi, an Koa wrapper for the [core of gengojs](https://github.com/iwatakeshi/gengojs-core). This module should be used when you need to test your gengojs plugins that need or use the `request` object.


The following is an example that shows you to:

* Make sure the plugin has properly loaded
* Make sure that the internal API is properly exposed.
* Make sure that the internal API's API are exposed as well.

Note that there are two version of Wrappify. One is without `harmony` and the other is.
The example below is with harmony. Therefore, make sure to use the `--harmony` flag
when running your tests.

```js
var assert = require('chai').assert;
var core = require('gengojs-core');
var header = require('your app path');
var wrappify = require('gengojs-wrappify/harmony');

describe('Header', function() {
  describe('load plugins', function() {
    it('should exist', function() {
     // Create an instance of the core.
      var gengo = core.create({}, header());
      gengo.utils._.forEach(gengo.plugins.headers, function(plugin) {
        assert.isDefined(plugin);
        assert.strictEqual(plugin.package.type, 'header');
        assert.strictEqual(plugin.package.name, 'gengojs-default-header');
      });
    });
  });
  describe('koa', function() {
    var gengo = core.create({}, header());
    var koa = require('koa');
    var app = koa();
    // Use Koa wrapper
    app.use(wrappify(gengo).koa());
    var request = require('supertest');
    it('should have the api exposed internally', function() {
      request(app.listen()).get('/').end(function() {
        assert.isDefined(gengo.header);
        assert.isDefined(gengo.header.getLocale);
      });
    });
  });

  describe('express', function() {
    var gengo = core.create({}, header());
    var express = require('express');
    var app = express();
    var request = require('supertest');
    // Use Express wrapper
    app.use(wrappify(gengo).express());
    it('should have the api exposed internally', function() {
      request(app).get('/').end(function() {
        assert.isDefined(gengo.header);
        assert.isDefined(gengo.header.getLocale);
      })
    });
  });

  describe('hapi', function() {
    var gengo = core.create({}, header());
    var Hapi = require('hapi');
    var server = new Hapi.Server();
    server.connection({
      port: 3000
    });
    // Register Hapi wrapper
    server.register(wrappify(gengo).hapi(), function(err) {});

    it('should have the api exposed internally', function() {
      server.inject('/', function(res) {
        assert.isDefined(gengo.header);
        assert.isDefined(gengo.header.getLocale);
      })
    });
  });
});
```