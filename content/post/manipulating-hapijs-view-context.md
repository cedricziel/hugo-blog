+++
title = "Preconfiguring the view context with hapi"
subtitle = "Injecting variables and helper functions into a hapi-jade-view-context."
tagline = "" 
author = "Cedric Ziel"
template = "post.html"
keywords = "HapiJS, hapi, view, jade"
changefreq = "monthly"
priority = "0.9"
date = "2014-02-11T21:58:29+01:00"
draft = false

+++

While [handlebars](http://handlebarsjs.com/) is able to use the ``registerHelpers`` method via a ``helpersPath`` option on your
``server`` (or ``pack``) object according to the [``views`` property](https://github.com/spumko/hapi/blob/master/docs/Reference.md#server.config.views), the [jade](http://jade-lang.com/) engine has no compatible mechanisms.

So.. How would you be able to add such functionality in a hapi based application without circumventing the view-initialization completely?

What I wanted to achieve, was a concept like in [connect-assets](https://github.com/adunkman/connect-assets/).
Simply put: An asset-pipeline, which is configurable in the view-without much overhead. But to be able to do so, there had to be a helper
in the view which would parse the asset files and _transformipile_<sup>TM</sup> them. This would also involve interchanging the
source path in the generated html output for each view.

## The easy path

Easy paths are easy. With jade, I would have added a function to the view context erverytime the handler is executed:

```javascript
exports.indexAction = function indexAction(request, reply) {

  return reply.view('home', {
    css: function(param){
      // For the sake of completeness: This would be a required module/function
      return transpileIt(param)
    }
  });
}
```

**BUT**: Would I really want to do this in every single handler? Importing, assignment.. _Much overhead, less wow!_

What i came up with, is a solution inspired by [Eran Hammer](https://twitter.com/eranhammer)'s [postmile-web](https://github.com/hueniverse/postmile-web) module:
Hook into the ``onPreRepsonse`` [lifecycle event](https://github.com/spumko/hapi/blob/master/docs/Reference.md#request-lifecycle),
and if the ``variety`` is ``view`` inject the function mentioned above into the view-context.

So how would you do this in practice?

## Your App

In the most simple case, your app looks something like that:

### index.js
```javascript
var Hapi = require('hapi');

// import them controllers
var homeController = require('./lib/controllers/home');


// configure the server
var options = {
  views: {
    engines: { jade: 'jade' },
    path: __dirname + '/views',
    compileOptions: {
      // should be set only in dev-context!
      pretty: true
    }
  }
};


// Create a new server
var server = new Hapi.Server('localhost', 8000, options);


// Extend the server for the onPreResponse
// - allows additional logic after arguments are parsed
//    and before the handler is invoked
server.ext('onPreResponse', function assetPipeHook(request, reply) {
  var response = request.response;
  if (response.variety === 'view') {
    var context = response.source.context;
    context.css = function (source) {
      var attrs = [];
      attrs.push('href=\"/static/css/' + source + '.css\"');
      return '\n<link ' + attrs.join(" ") + '/>';
    }
    return reply();
  }
  return reply();
});


// Simple route
server.route([
  {
    method: 'GET',
    path: '/',
    config: {
      handler: homeController.indexAction
    }
  }
]);

server.start(function (err) {
  if (err) throw err;
  console.log('✔ Hapi server listening on port ' + 8000);
})
```

### lib/controllers/home.js
```javascript
/**
 * A simple controllerAction to render the view
 *
 * We could have done this inside the route definition,
 * as it adds a little overhead-but you never know what happens to your little views.
 *
 * @param request
 * @param reply
 */
exports.indexAction = function indexAction(request, reply) {

  return reply.view('home', {
  });
}
```

### views/home.jade
```jade
extends layout

  block content
```

### views/layout.jade
```jade
doctype html
html
  head
    meta(charset='utf-8')
    meta(http-equiv='X-UA-Compatible', content='IE=edge')
    meta(name='viewport', content='width=device-width, initial-scale=1.0')
    title Very simple, much view-so wow!
    != css('styles')
  body
    block content
```

## Conclusion

Once you're feeling comfortable with hooking in deeply to the framework, which I want to encourage you to, it's fairly
easy to manipulate contexts.

### How to move from here?

I felt very uncomfortable with doing everything within the request to my template route, as it's done within ``connect-assets``.
Even though you would precompile the assets for production, and the given link would be generated much faster, this didn't feel right.

I'll go on and add a route handler for the ``/static`` routes to be able to transform on-request.

Thank you for reading!
