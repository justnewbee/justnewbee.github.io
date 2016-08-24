---
layout: post
title: Angular 十大错误「译」
date: 2015-01-05 20:19:41
categories: translation
tags: angular
---

https://www.airpair.com/angularjs/posts/top-10-mistakes-angularjs-developers-make

FREE WORKSHOP: Tips for AngularJS DevelopersMark Meyer is giving a free virtual workshop on Angular best practices and how to avoid some common mistakes. >> Sign up to secure a spot

# Introduction

AngularJS is one of the most popular Javascript frameworks available today. One of AngularJS's goals is to simplify the development process which makes it great for prototyping small apps, but its power allows scaling to full featured client side applications. The combination ease of development, breadth of features, and performance has led to wide adoption, and with wide adoption comes many common pitfalls. This list captures common AngularJS mistakes, especially when scaling an app.

# 1 MVC directory structure

AngularJS is, for lack of a better term, an MVC framework. The models aren't as clearly defined as a framework like backbone.js, but the architectural pattern still fits well. When working in an MVC framework a common practice is to group files together based on file type:

```
templates/
    _login.html
    _feed.html
app/
    app.js
    controllers/
        LoginController.js
        FeedController.js
    directives/
        FeedEntryDirective.js
    services/
        LoginService.js
        FeedService.js
    filters/
        CapatalizeFilter.js
```

This seems like an obvious layout, especially if coming from a Rails background. However once the app begins to scale, this layout causes many folders to be open at once. Whether using Sublime, Visual Studio, or Vim with Nerd Tree, a lot of time is spent scrolling through the directory tree.

Instead of keeping the files grouped by type, group files based on features:

```
app/
    app.js
    Feed/
        _feed.html
        FeedController.js
        FeedEntryDirective.js
        FeedService.js
    Login/
        _login.html
        LoginController.js
        LoginService.js
    Shared/
        CapatalizeFilter.js
```

This directory structure makes it much easier to find all the files related to a particular feature which will speed up development. It may be controversial to keep .html files with .js files, but the time savings are more valuable.

# 2 Modules (or lack thereof)

It's common when starting out to hang everything off of the main module. This works fine when starting a small app, but it quickly becomes unmanageable.

```
var app = angular.module('app',[]);
app.service('MyService', function(){
    //service code
});
app.controller('MyCtrl', function($scope, MyService){
    //controller code
});
```

A common strategy after this is to group similar types of objects:

```
var services = angular.module('services',[]);
services.service('MyService', function(){
    //service code
});

var controllers = angular.module('controllers',['services']);
controllers.controller('MyCtrl', function($scope, MyService){
    //controller code
});

var app = angular.module('app',['controllers', 'services']);
```

This strategy scales as well as the directory structures from part 1: not great. Following the same concept of grouping features together will allow scalability.

var sharedServicesModule = angular.module('sharedServices',[]);
sharedServices.service('NetworkService', function($http){});

var loginModule = angular.module('login',['sharedServices']);
loginModule.service('loginService', function(NetworkService){});
loginModule.controller('loginCtrl', function($scope, loginService){});

var app = angular.module('app', ['sharedServices', 'login']);
Like learning from posts like this? Subscribe for more!

When working on large applications everything might not be contained on a single page, and by having features contained within modules it's much simpler to reuse modules across apps.

# 3 Dependency injection

Dependency injection is one of AngularJS's best patterns. It makes testing much simpler, as well as making it more clear upon which any particular object depends. AngularJS is very flexible on how things can be injected. The simplest version requires just passing the name of the dependency into the function for the module:

```
var app = angular.module('app',[]);

app.controller('MainCtrl', function($scope, $timeout){
    $timeout(function(){
        console.log($scope);
    }, 1000);
});
```

Here, it's very clear that MainCtrl depends on $scope and $timeout.

This works well until you're ready to go to production and want to minify your code. Using UglifyJS the example becomes the following:

var app=angular.module("app",[]);app.controller("MainCtrl",function(e,t){t(function(){console.log(e)},1e3)})
Like learning from posts like this? Subscribe for more!

Now how does AngularJS know what MainCtrl depends upon? AngularJS provides a very simple solution to this; pass the dependencies as an array of strings, with the last element of the array being a function which takes all the dependencies as parameters.

```
app.controller('MainCtrl', ['$scope', '$timeout', function($scope, $timeout){
    $timeout(function(){
        console.log($scope);
    }, 1000);
}]);
```

This then minifies to code with clear dependencies that AngularJS knows how to interpret:

app.controller("MainCtrl",["$scope","$timeout",function(e,t){t(function(){console.log(e)},1e3)}])
Like learning from posts like this? Subscribe for more!

## 3.1 Global dependencies

Often when writing AngularJS apps there will be a dependency on an object that binds itself to the global scope. This means it's available in any AngularJS code, but this breaks the dependency injection model and leads to a few issues, especially in testing.

AngularJS makes it simple to encapsulate these globals into modules so they can be injected like standard AngularJS modules.

Underscore.js is a great library for simplifying Javascript code in a functional pattern, and it can be turned into a module by doing the following:

```
var underscore = angular.module('underscore', []);
underscore.factory('_', function() {
  return window._; //Underscore must already be loaded on the page
});
var app = angular.module('app', ['underscore']);

app.controller('MainCtrl', ['$scope', '_', function($scope, _) {
    init = function() {
          _.keys($scope);
      }

      init();
}]);
```

This allows the application to continue in the AngularJS dependency injection fashion which would allow underscore to be swapped out at test time.

This may seem trivial and like unnecessary work, but if your code is using use strict (and it should be!), then this becomes a requirement.
4 Controller bloat

Controllers are the meat and potatoes of AngularJS apps. It's easy, especially when starting out, to put too much logic in the controller. Controllers should never do DOM manipulation or hold DOM selectors; that's where directives and using ng-model come in. Likewise business logic should live in services, not controllers.

Data should also be stored in services, except where it is being bound to the $scope. Services are singletons that persist throughout the lifetime of the application, while controllers are transient between application states. If data is stored in the controller then it will need to be fetched from somewhere when it is reinstantiated. Even if the data is stored in localStorage, it's an order of magnitude slower to retrieve than from with a Javascript variable.

AngularJS works best when following the Single Responsibility Principle (SRP). If the controller is a coordinator between the view and the model, then the amount of logic it has should be minimal. This will also make testing much simpler.
5 Service vs Factory

These names cause confusion for almost every AngularJS developer when starting out. They really shouldn't because they're syntactic sugar for (almost) the same thing!

Here are their definitions from the AngularJS source:

```
function factory(name, factoryFn) { 
    return provider(name, { $get: factoryFn }); 
}

function service(name, constructor) {
    return factory(name, ['$injector', function($injector) {
      return $injector.instantiate(constructor);
    }]);
}
```

From the source you can see that service just calls the factory function which in turn calls the provider function. In fact, AngularJS also offers a few additional provider wrappers with value, constant, and decorator. These other objects don't cause nearly the same level of confusion and the docs are pretty clear on their use cases.

Since service just calls the factory function, what makes it different? The clue is in $injector.instantiate; within this function the $injector creates a new instance of the service's constructor function.

Here's an example of a service and a factory that accomplish the same thing.

```
var app = angular.module('app',[]);

app.service('helloWorldService', function(){
    this.hello = function() {
        return "Hello World";
    };
});

app.factory('helloWorldFactory', function(){
    return {
        hello: function() {
            return "Hello World";
        }
    }
});
```

When either helloWorldService or helloWorldFactory are injected into a controller, they will both have a hello method that returns "Hello World". The service constructor function is instantiated once at declaration and the factory object is passed around every time it is injected, but there is still just one instance of the factory. All providers are singletons.

Why do the two styles exist if they can accomplish the same thing? Factories offer slightly more flexibility than services because they can return functions which can then be new'd. This follows the factory pattern from object oriented programming. A factory can be an object for creating other objects.

```
app.factory('helloFactory', function() {
    return function(name) {
        this.name = name;

        this.hello = function() {
            return "Hello " + this.name;
        };
    };
});
```

Here's an example controller using the service and the two factories. With the helloFactory returning a function, the name value is set when the object is new'd.

```
app.controller('helloCtrl', function($scope, helloWorldService, helloWorldFactory, helloFactory) {
    init = function() {
      helloWorldService.hello(); //'Hello World'
      helloWorldFactory.hello(); //'Hello World'
      new helloFactory('Readers').hello() //'Hello Readers'
    }

    init();
});
```

When starting out it is best to just use services.

Factories can also be more useful when designing a class with more private methods:

```
app.factory('privateFactory', function(){
    var privateFunc = function(name) {
        return name.split("").reverse().join(""); //reverses the name
    };

    return {
        hello: function(name){
          return "Hello " + privateFunc(name);
        }
    };
});
```

With this example it's possible to have the privateFunc not accessible to the public API of privateFactory. This pattern is achievable in services, but factories make it more clear.
# 6 Not utilizing Batarang

Batarang is an excellent Chrome extension for developing and debugging AngularJS apps.

Batarang offers Model browsing to get a look inside what Angular has determined to be the models bound to scopes. This can be useful when working with directives and to isolate scopes to see where values are bound.

Batarang also offers a dependency graph. If coming into an untested codebase, this would be a useful tool to determine which services should get the most attention.

Finally, Batarang offers performance analysis. AngularJS is quite performant out of the box, but as an app grows with custom directives and complex logic, it can sometimes feel less than smooth. Using the Batarang performance tool it is quite simple to see which functions are taking the most time in the digest cycle. The performance tool also shows the full watch tree, which can be useful when having too many watchers.

# 7 Too many watchers

As mentioned in the last point, AngularJS is quite performant out of the box. Because of the dirty checking done in a digest cycle, once the number of watchers exceeds about 2,000 the cycle can cause noticeable performance issues. (The 2,000 number isn't guaranteed to be a sharp drop off, but it is a good rule of thumb. Changes are coming in the 1.3 release of AngularJS that allow tighter control over digest cycles. Aaron Gray has a nice write up on this.)

This Immediately Invoked Function Expression (IIFE) will print out the number of watchers currently on the page. Simply paste it into the console to see how many watchers are currently on the page. This IIFE was taken from Words Like Jared's answer on StackOverflow:

```
(function () { 
    var root = $(document.getElementsByTagName('body'));
    var watchers = [];

    var f = function (element) {
        if (element.data().hasOwnProperty('$scope')) {
            angular.forEach(element.data().$scope.$$watchers, function (watcher) {
                watchers.push(watcher);
            });
        }

        angular.forEach(element.children(), function (childElement) {
            f($(childElement));
        });
    };

    f(root);

    console.log(watchers.length);
})();
```

By using this to determine the number of watchers and the watch tree from the performance section of Batarang, it should be possible to see where duplication exists or where unchanging data has a watch.

When there is data that does not change, but you want it to be templated using Angular, consider using bindonce. Bindonce is a simple directive which allows you to use templates within Angular, but this does not add a watch to keep the count from increasing.
8 Scoping $scope's

Javascript's prototype-based inheritance differs from class-based inheritance in nuanced ways. This normally isn't a problem, but the nuances often arise when working with $scope. In AngularJS every $scope inherits from its parent $scope with the highest level being $rootScope. ($scope behaves slightly differently in directives, with isolate scopes only inherit properties explicitly declared.)

Sharing data from a parent to a child is trivial because of the prototype inheritance. It is easy however to shadow a property of the parent $scope if caution is not taken.

Let's say we want to have a username displayed in a navbar, and it is entered in a login form. This would be a good first try at how this might work:

```
<div ng-controller="navCtrl">
   <span>{{user}}</span>
   <div ng-controller="loginCtrl">
        <span>{{user}}</span>
        <input ng-model="user"></input>
   </div>
</div>
```

Quiz time: when a user types in the text input with the user ng-model set on it, which template will update? The navCtrl, loginCtrl, or both?

If you selected loginCtrl then you probably already understand how prototypical inheritance works.

When looking up literal values, the prototype chain is not consulted. If navCtrl is to be updated simultaneously then a prototype chain lookup is required; this will happen when the value is an object. (Remember, in Javascript, functions, arrays, and objects are objects)

So to get the desired behavior it is necessary to create an object on the navCtrl that can be referenced from loginCtrl.

```
<div ng-controller="navCtrl">
   <span>{{user.name}}</span>
   <div ng-controller="loginCtrl">
        <span>{{user.name}}</span>
        <input ng-model="user.name"></input>
   </div>
</div>
```

Now since user is an object, the prototype chain will be consulted and the navCtrl's template and $scope will be updated along with loginCtrl.

This may seem like a contrived example, but when working with directives that create child $scope's like ngRepeat this issue can arise easily.

# 9. Manual Testing

While TDD might not be every developer's preferred development method, every time a developer checks to see if code works or has broken something else he or she is doing manual testing.

There are no excuses for not testing an AngularJS app. AngularJS was designed to be testable from the ground up. Dependency injection and the ngMock module are evidence of this. The core team has developed a couple of tools that can bring testing to the next level.

## 9.1 Protractor

Unit tests are the building blocks of a test suite, but as the complexity of an app grows integration tests tease out more realistic situations. Luckily the AngularJS core team has provided the necessary tool.

> We have built Protractor, an end to end test runner which simulates user interactions that will help you verify the health of your Angular application.

Protractor uses the Jasmine test framework for defining tests. Protractor has a very robust API for different page interactions.

There are other end to end test tools, but Protractor has the advantage of understanding how to work with AngularJS code, especially when it comes to $digest cycles.

## 9.2. Karma

Once integration tests have been written using Protractor, the tests need to run. Waiting for tests to run, especially integration tests, can be frustrating for developers. The AngularJS core team also felt this pain and developed Karma.

Karma is a Javascript test runner which helps close the feedback loop. Karma does this by running the tests any time specified files are changed. Karma will also run tests in parallel across different browsers. Different devices can also be pointed to the Karma server to get better coverage of real world usage scenarios.

# 10 Using jQuery

jQuery is an amazing library. It has standardized cross-platform development and is almost a requirement in modern web development. While jQuery has many great features, its philosophy does not align with AngularJS.

AngularJS is a framework for building applications; jQuery is a library for simplifying "HTML document traversal and manipulation, event handling, animation, and Ajax". This is the fundamental difference between the two. AngularJS is about architecture of applications, not augmenting HTML pages.

In order to really understand how to build an AngularJS application, stop using jQuery. jQuery keeps the developer thinking of existing HTML standards, but as the docs say "AngularJS lets you extend HTML vocabulary for your application."

DOM manipulation should only be done in directives, but this doesn't mean they have to be jQuery wrappers. Always consider what features AngularJS already provides before reaching for jQuery. Directives work really well when they build upon each other to create powerful tools.

The day may come when a very nice jQuery library is necessary, but including it from the beginning is an all too common mistake.

# Conclusion

AngularJS is a great framework that continues to evolve with the community. Idiomatic AngularJS is still an evolving concept, but hopefully by following these conventions some of the major pitfalls of scaling an AngularJS app can be avoided.
