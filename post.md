This article will go through the changes you have to do to migrate your code and the default test runner `SpecRunner.html` to use RequireJS. We will cover:

*   Changes to the default Jasmine setup
*   RequireJS configuration
*   Changes to our libraries and specs.

Source code available at [gsans/jasmine-require-bootstrap](https://github.com/gsans/jasmine-require-bootstrap) (Github).

## Jasmine Default Setup

Find the steps to setup Jasmine [here](https://github.com/jasmine/jasmine). We are going to use this folder structure.

```
├───src
├───tests
│   └───lib
└───vendor
```

Your `SpecRunner.html` should look similar to this:

```markup
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Jasmine Spec Runner v2.0.0</title>

<link rel="shortcut icon" type="image/png" href="lib/jasmine_favicon.png">
<link rel="stylesheet" type="text/css" href="lib/jasmine.css">

<script type="text/javascript" src="lib/jasmine.js"></script>
<script type="text/javascript" src="lib/jasmine-html.js"></script>
<script type="text/javascript" src="lib/boot.js"></script>

<!-- include source files here... -->
<script type="text/javascript" src="../src/my-library.js"></script>

<!-- include spec files here... -->
<script type="text/javascript" src="../src/my-library.specs.js"></script>

</head>

<body>
</body>
</html>
```

This is how `my-library.js` and `my-library.specs.js` look:

```javascript
// my-library.js
var myLibrary = (function() {
  function sayHello() {
    return "Hello";
  }
  return {
    sayHello: sayHello
  };
})();
 
// my-library.specs.js
describe("my-library", function(){
  describe("sayHello", function(){
    it("should say Hello", function(){
      expect(myLibrary.sayHello()).toEqual("Hello");
    })
  })
})
```

If you have node installed in your machine you can use _http-server_ to run the _SpecRunner.html_ passing the only spec in our suite_._

![](https://d262ilb51hltx0.cloudfront.net/max/1000/1*-hFjUCMZbrknJQ2UJ0vW5w.png)
SpecRunner.html output

## RequireJS Setup

In order to use RequireJS we will have to change our previous _SpecRunner.html_ page to the following:

```markup
<!DOCTYPE HTML>
<html>
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  <title>Jasmine Spec Runner v2.0.0</title>
 
  <link rel="shortcut icon" type="image/png" href="lib/jasmine_favicon.png">
  <link rel="stylesheet" type="text/css" href="lib/jasmine.css">
  <script type="text/javascript" src="../vendor/require.js" data-main="main"></script>
</head>
<body>
</body>
</html>
```

We removed all external scripts files and only left _require.js_.
> Note how we omitted the&nbsp;*.js* extension for *main.js*. This is how RequireJS identifies files by default.

### RequireJS configuration file&#8202;—&#8202;main.js

In [_main.js_](http://requirejs.org/docs/api.html#config) we will set the configuration options to load all dependencies and bootstrap Jasmine.

```javascript
// Requirejs Configuration Options
require.config({
  // to set the default folder
  baseUrl: '../src', 
  // paths: maps ids with paths (no extension)
  paths: {
      'jasmine': ['../tests/lib/jasmine'],
      'jasmine-html': ['../tests/lib/jasmine-html'],
      'jasmine-boot': ['../tests/lib/boot']
  },
  // shim: makes external libraries compatible with requirejs (AMD)
  shim: {
    'jasmine-html': {
      deps : ['jasmine']
    },
    'jasmine-boot': {
      deps : ['jasmine', 'jasmine-html']
    }
  }
});
```

> We used jasmine-boot as an alias for boot.js

### RequireJS Bootstrap

This setup will start with jasmine-boot identifier. It has two dependencies: jasmine and jasmine-html. As jasmine-html has jasmine as a dependency it will then proceed to load jasmine.js, then jasmine-html.js and finally boot.js as it was doing on our original runner.

> We use **_require() _**to load dependencies before running our code.

```javascript
require(['jasmine-boot'], function () {
  require(['my-library.specs'], function(){
    //trigger Jasmine
    window.onload();
  })
});
```

The previous code will load all _jasmine-boot_ dependencies and continue with our specs in _my-library.specs.js_.
> Note that if we had moved _my-library.specs_ to the same level as_ jasmine-boot_ it could had been loaded before all jasmine-boot dependencies were done.

Once all script files are loaded we trigger _window.onload()_ as Jasmine hooks into this event to initialise its engine.

### Changes to our library

In order to create our library we will wrap it using define. As we don’t have any dependencies it will be empty. Note also how we added a return statement at the end, this is so it can be used on other modules via parameters without polluting the global _window_ object.

```javascript
define([], function(){
 
  var myLibrary = (function() {
    function sayHello() {
      return "Hello";
    }
    return {
      sayHello: sayHello
    };
  })();
 
  return myLibrary;
})
```

> We use *define()* to define our libraries. This can load dependencies and return an instance for others to use.

### Changes to our specs

In order to migrate our specs we will repeat the same as we did before. In this case we do have a dependency as we need to load my-library.js in order to run our tests.

```javascript
define(['my-library'], function(myLibrary){
 
  describe("my-library", function(){
    describe("sayHello", function(){
      it("should say Hello", function(){
        expect(myLibrary.sayHello()).toEqual("Hello");
      })
    })
  })
  
})
```

The *myLibrary* parameter will get the instance returned by our library new definition.

We can now run our tests taking advantage of RequireJS helper functions making the testing experience more enjoyable.

### Resources

[Angular&#8202;—&#8202;Unit Testing with Jasmine](https://medium.com/angularjs-meetup-south-london/angular-unit-testing-with-jasmine-24795a44998e)

![](https://d262ilb51hltx0.cloudfront.net/max/1000/1*VEsYW1ANSic04En6nx7yHw.png)
