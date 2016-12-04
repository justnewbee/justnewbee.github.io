---
layout: post
title: 使用 Angular 时常犯的十大错误「译」
date: 2015-01-05 20:19:41
categories: translation
tags: angular
---

原文地址：<https://www.airpair.com/angularjs/posts/top-10-mistakes-angularjs-developers-make>

# 序

[AngularJS](https://angularjs.org/) 是时下最流行的 JavaScript 框架之一，它的其中一个目标是简化前端开发，这使得它十分适合快速搭建小项目的原型，但它更多是用来完成全功能的客户端应用。开发流程的简化，丰富的功能以及性能的卓越表现，使得 AngularJS 得到广泛的运用，随之而来的是在运用过程中所犯的错误。本文列举了常犯的错误，尤其是在写大型项目的时候。

# 1 MVC 目录结构

AngularJS （在没有更恰当的词汇的情况下）的确是一个 MVC 框架。然而它并不像 [backbone.js](http://backbonejs.org/) 那样有清晰的定义。我们在使用 MVC 框架的时候，一个常用的做法就是把文件以类型归类，如下：

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

这个结构非常明确，尤其当你有 Rails 背景，应该会感到很熟悉。因为相同功能的代码被安置在不同的目录，当项目规模变大，你不得不同时打开不同的多个目录来修改一个功能。无论你是用 Sublime，Visual Studio 或者装了 Nerd Tree 插件的 Vim，在文件树上滚来滚去找文件都需要花费大量时间。

所以，我们有另外的一个选择，按功能（用途）对文件进行分组：

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

关联功能的文件放在同一个目录下，修改代码就很简单了，虽然把 `.html` 和 `.js` 文件放一起比较有争议，但相对来说节省时间更为重要。

# 2 模块

我们一开始往往把所有的东西都挂在一个主模块下面，在项目的初期阶段，这样做的问题不大，但慢慢地就会发现，继续这样下去的话项目就会变得不可维护。

```js
var app = angular.module('app', []);
app.service('MyService', function() {
    // service code
});
app.controller('MyCtrl', function($scope, MyService) {
    // controller code
});
```

常用的策略是把相近的对象组合在一起：

```js
var services = angular.module('services', []);
services.service('MyService', function() {
    // service code
});

var controllers = angular.module('controllers', ['services']);
controllers.controller('MyCtrl', function($scope, MyService) {
    // controller code
});

var app = angular.module('app', ['controllers', 'services']);
```

跟前面提到的按文件类型编排目录结构一样，这样的策略有着同样的问题；按功能分模块将会有更好的扩展性。

```js
var sharedServicesModule = angular.module('sharedServices', []);
sharedServices.service('NetworkService', function($http) {});

var loginModule = angular.module('login', ['sharedServices']);
loginModule.service('loginService', function(NetworkService) {});
loginModule.controller('loginCtrl', function($scope, loginService) {});

var app = angular.module('app', ['sharedServices', 'login']);
```

大型项目，一个页面可能不会包含所有的代码，把功能整合到模块可以让代码的复用更简单。

# 3 依赖注入

依赖注入是 AngularJS 最令人激动的设计模式之一。它使得代码测试起来十分简单，同时在 Angular 体系中的任何对象的依赖关系也一清二楚。Angular 提供了灵活的依赖注入的形式，最简单的就是把依赖的名字传给定义模块的方法：

```js
var app = angular.module('app', []);

app.controller('MainCtrl', function($scope, $timeout) {
    $timeout(function() {
        console.log($scope);
    }, 1000);
});
```

如此，`MainCtrl` 依赖 `$scope` 和 `$timeout` 就十分明显了。

一切都运行完好..直到，你的项目准备上线，而你需要把代码后在进行发布，比如我们使用 [UglifyJS](http://lisperator.net/uglifyjs/)，前面的例子就变成下面的这个样子：

```js
var app=angular.module("app",[])
app.controller("MainCtrl",function(n,o){o(function(){console.log(n)},1e3)})
```

这下好了，叫 AngularJS 可怎么知道 `MainCtrl` 依赖了什么呢？AngularJS 提供的解决方案很简单，把依赖以字符串数组的形式进行传递，数组的最后一个元素是定义用的方法，这个方法接收的参数即它前面字符串对应的依赖。

```js
app.controller('MainCtrl', ['$scope', '$timeout', function($scope, $timeout) {
    $timeout(function() {
        console.log($scope);
    }, 1000);
}]);
```

这样，代码即使经过压缩，AngularJS 还是能够找到正确的依赖：

```js
app.controller("MainCtrl",["$scope","$timeout",function(e,t){t(function(){console.log(e)},1e3)}])
```

## 3.1 全局依赖

在写 Angular 应用的时候，经常会有某个依赖把自己放到全局的 `scope` 里，这样它就能在该应用的所有 Angular 代码中都可见，然而，这样的行为打破了 Angular 的依赖注入的模式，可能带来一些问题，尤其是单元测试的时候。

Angular 的处理方式十分简单，把它们以一个个独立的功能模块的形式封装起来，这样就可以像别的标准模块一样被当成依赖注入到别的模块中去了。

[Underscore.js](http://underscorejs.org) 是一个很棒的库，它提供的功能函数能够极大简化我们的代码。在 Angular 中，我们只需要这么做，即可将它转化为一个模块：

```js
var underscore = angular.module('underscore', []);
underscore.factory('_', function() {
    return window._; // Underscore must already be loaded on the page
});
var app = angular.module('app', ['underscore']);

app.controller('MainCtrl', ['$scope', '_', function($scope, _) {
    var init = function() {
        _.keys($scope);
    }
    
    init();
}]);
```

这样做，应用能继续遵循 Angular 的依赖注入模式，可以在单元测试的时候方便地将 underscore 给替换掉。

这额做看上去很无聊，没什么必要，但如果你的代码运行在严格模式下（就应该严格模式），这么做就是必要的了。


# 4 臃肿的 Controller

`controller` 在 Angular 应用中扮演着至关重要的角色。但我们经常不经意间就在 `controller` 里写了大量的逻辑代码，尤其是在项目的初期阶段。`controller` 不应该有任何的 DOM 操作，或 DOM 选择器，这些脏活累活应该交给 `directive` 来干；同样的，业务逻辑，应该放到 `service` 中。

数据应该存在 `service` 中，除非是需要绑定到的 `$scope` 上的数据。`service` 是单例的，在应用的生命周期都不会变；而 `controller` 的生命周期却很短，跟应用当前的状态有关。如果数据存在 `controller` 中，那么当 `controller` 被重新实例化的时候，就需要再从哪里获取一份数据，即使是 `localStorage`，跟从 JavaScript 的对象中获取数据相比，这个时间也是相当慢的。

遵循 SRP（Single Responsibility Principle「单一责任原则」），能让 Angular 运作地更好。如果 `controller` 仅仅只是做视图（view）和模型（model）之间的协调者，那么它的代码应该可以是极简的。

# 5 Service vs Factory

几乎每个 Angular 的开发这对这些名词都产生过疑惑，其实大可不必，它们只是用来完成的同样（几乎同样）任务的语法糖而已。

以下是 Angular 源代码中关于两者的定义：

```js
function factory(name, factoryFn) { 
    return provider(name, { $get: factoryFn }); 
}

function service(name, constructor) {
    return factory(name, ['$injector', function($injector) {
      return $injector.instantiate(constructor);
    }]);
}
```

从源代码中，可以看到 `service` 调用了 `factory`，而 `factory` 又调用了 `provider`，事实上，Angular 还提供了几个 `provider` 的封装：`value`、`constant`、`decorator`，这些封装不会像 `service` 和 `factory` 那样让人容易产生误解，毕竟从字面上也好，Angular 的官方文档也好，都很清楚地能知道它们的应用场景。

那么 `service` 和 `factory` 究竟有什么区别呢？线索就在 `$injector.instantiate`，通过它 `$injector` 为 `service` 的构造函数创建了一个新的事例。

下面是一个例子，`service` 和 `factory` 做的是同一个事情：

```js
var app = angular.module('app', []);

app.service('helloWorldService', function() {
    this.hello = function() {
        return "Hello World";
    };
});

app.factory('helloWorldFactory', function() {
    return {
        hello: function() {
            return "Hello World";
        }
    }
});
```

When either helloWorldService or helloWorldFactory are injected into a controller, they will both have a hello method that returns "Hello World". The service constructor function is instantiated once at declaration and the factory object is passed around every time it is injected, but there is still just one instance of the factory. All providers are singletons.

`helloWorldService` `helloWorldFactory` 两者被注入到 `controller`，它们都会有一个 `hello` 的方法，这个方法的作用很简单，就是单纯地返回字符串 `"Hello World"`。`service` 的构造方法在其声明的时候就被实例化，而 `factory` 则是一个对象，在依赖注入的时候被传来传去，永远只有一个实例对象。所有的 `provider` 都是单例的。

那么为什么做同样的事情要两种形式呢？`factory` 相对 `service` 而言，可以提供多那么一点点灵活性，因为它可以返回一个 `function`，你可以当成类来 `new`。下面的例子，是一个面向对象形式的 `factory`，`factory` 可以是一个用来创建别的对象的对象。

```js
app.factory('helloFactory', function() {
    return function(name) {
        this.name = name;

        this.hello = function() {
            return "Hello " + this.name;
        };
    };
});
```

接下来的例子是一个 `controller`，它依赖上面提到的一个 `service` 和 两个 `factory`，`name` 的值是在 `new` 赋予的。

```js
app.controller('helloCtrl', function($scope, helloWorldService, helloWorldFactory, helloFactory) {
    init = function() {
      helloWorldService.hello(); // 'Hello World'
      helloWorldFactory.hello(); // 'Hello World'
      new helloFactory('Readers').hello(); // 'Hello Readers'
    }

    init();
});
```

如果你是新手，最好先用 `service`。

如果你需要一个类，而这个类又需要较多的私有方法的时候，`factory` 的作用更加明显。

```js
app.factory('privateFactory', function() {
    var privateFunc = function(name) {
        return name.split("").reverse().join(""); // reverses the name
    };

    return {
        hello: function(name){
          return "Hello " + privateFunc(name);
        }
    };
});
```

With this example it's possible to have the privateFunc not accessible to the public API of privateFactory. This pattern is achievable in services, but factories make it more clear.

这个例子中，`privateFunc` 可以做到完全不被外部所感知；虽然用 `service` 也可以做到这一点，在 `factory` 中这么做更清楚一些。

「译者注：看不出来哪里清楚了...」

# 6 怎么，还没用上 Batarang

Batarang 是 Chrome 上的一个插件，开发和调试 Angular 应用必不可少。

Batarang 提供了 Model 浏览方法，在作用于 `scope` 上绑定的 `model` 可以看得一清二楚。在写 `directive` 的看数据是如何绑定的，这个功能特别有用。

Batarang 还有个依赖图谱的功能，你可以通过它看出那些 `service` 的重要程度最高。

最后，Batarang 提供了性能分析工具。Angular 本身其实运行效率很高，但伴随这应用的扩大，大量的自定义 `directive` 和复杂逻辑的加入，可能就导致应用会慢慢变得不那么顺畅。这个功能，可以找到究竟是哪些方法在 `digest` 周期中消耗的时间最多。同时，该工具提供了一个完整的 `watch` 树，这在你的项目又很多的 `watcher` 的时候很有用。

# 7 过多的 watcher

如前所述，Angular 自身的性能已经非常好了。Angular 在一个 digest cyble 中做脏数据检查，因此，当 watcher 的数量达到约 2000 个的时候，性能问题就会显现出来，2000 只是一个经验值，不代表到了这个点就会又非常明显的差别。Angular 1.3 将允许控制 digest cycle，以期解决类似的问题，[Aaron Gray 的这篇文章很好地研究了这个问题](http://www.aaron-gray.com/delaying-the-digest-cycle-in-angularjs/)。

下面这个 IIFE（立即执行函数表达式）可以打印出当前页面上的 `watcher` 数，你只要在 `console` 中执行该段代码即可，此 IIFE 的出处在 [StackOverflow](http://stackoverflow.com/questions/18499909/how-to-count-total-number-of-watches-on-a-page) 上：

```js
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

用上面的这段 IIFE 判断有多少的 `watcher` 在页面上，加上 Batarang 的 watch 树功能，我们可以找到是否有冗余的 `watcher`，或者是否有 `watcher` 在并不会改变的数据上。

如果有数据并不会改变，而你又需要它用于模板，可以考虑使用 [bindonce](https://github.com/Pasvaz/bindonce)，它使得我们在把数据绑定到模板的时候，不会增加多余的 `watcher`。

8 Scoping $scope's

JavaScript 基于原型（`prototype`）的继承方式与基于类的继承方式实际差别比较细微，通常这不会带来什么问题，然而，当我们跟 `$scope` 打交道的时候，这些差别就会显$现出来。在 Angular 中，每个 `$scope` 都会从它的父 `$scope` 中进行继承，一直到最上层的 `$rootScope`（在 `derective` 中，`$scope` 的行为略有不同，若是孤立的 `$scope`，只会继承指定的属性）。

通过原型继承的方式从父到子共享数据其实并不是特别有用，反而会带来一些问题，诸如子 `$scope` 会把父亲的某个属性给不小心覆盖了的情况。

比如我们希望在导航条上显示用户名，而它是由一个登录表单输入的。下面的代码是一个好的开端，获取是可以工作的：

```html
<div ng-controller="navCtrl">
   <span>{{user}}</span>
   <div ng-controller="loginCtrl">
        <span>{{user}}</span>
        <input ng-model="user"></input>
   </div>
</div>
```

问：当用户在输入框（它的 `ng-model` 是 `user`）中输入的时候，哪个模板会被更新呢，`navCtrl`，`loginCtrl` 还是两个都会？

如果你的选择是 `loginCtrl`，你应该已经对原型继承的工作机理比较了解了。

当查找简单类型的数据，原型链并不会被查阅。所以，如果要让 `navCtrl` 能够感知到数据的变化，就需要让被查的数据属于一个引用类型的对象（`function`、数组或对象）。

所以，我们需要在 `navCtrl` 上创建一个对象，`loginCtrl` 也就能访问到该对象，从而达到预期的效果。

```html
<div ng-controller="navCtrl">
   <span>{{user.name}}</span>
   <div ng-controller="loginCtrl">
        <span>{{user.name}}</span>
        <input ng-model="user.name"></input>
   </div>
</div>
```

如此一来，当输入被改变的时候，`navCtrl` 会跟着 `loginCtrl` 一起变化，因为在查找对象的时候，原型链会被逐层往上查阅。

这个例子有点牵强，看起来像故意制造问题一样，但使用 `directive` 的时候需要当心，因为它可能会创建子的 `$scope`，这个时候就容易发生类似的问题。

# 9 人工测试

TDD 也许不是每个开发者喜欢的开发方式，很多人喜欢手工的方式来验证代码是否能够正常工作。

但对于 Angular 应用来说，你没有借口不进行测试，Angular 在出生的那一刻起，就是设计成可测试的，依赖注入和 `ndMock` 模块就是最好的证明。Angular 核心组开发了一些工具，可以帮助我们把测试提升到一个新的层次。

## 9.1 Protractor

单元测试是测试套件的基石，但随着项目的增长，只有集成测试才能帮我们理顺更多的实际情况，幸运的是，Angular 核心开发组为我们提供了这样的工具。

> 我们开发了 [Protractor](https://github.com/angular/protractor)，这是一个端到端的测试运行器，它能够模拟用户的交互行为，可以帮助你了解项目的健康程度。

Protractor 使用的是 [Jasmine](http://jasmine.github.io/1.3/introduction.html) 测试框架来定义测试用例，同时具有完善的 API 来定义不同的页面交互。

虽然还有其他的端测试工具可以选择，但 Protractor 胜在它对 Angular 的运行机制了如指掌，尤其是 `$digest` cycle。

## 9.2. Karma

我们用 Protractor 写完集成测试用例后，需要让它跑起来。等待测试用例的运行完成，尤其是继承测试的测试用例，对于开发者来说是一件比较痛苦的事情，Angular 核心组的成员也感受到了这种痛苦，所以他们又开发了 [Karma](http://karma-runner.github.io/0.12/index.html)。

Karma 是 JavaScript 的测试运行器，它可以在制定文件被修改的任意时刻跑测试用例。Karma 还能并行地对不同的浏览器进行测试，你可以为 Karma 服务器指定不同的机型以期更全面的场景覆盖率。

# 10 还在用 jQuery？

不得不说，jQuery 是一个太棒了的库，它提供了夸浏览器的前端开发解决方案，几乎是目前网络开发的必备，即便它如此强大，如此好用，它跟 Angular 的哲学却是不搭调的。

Angular 是用来搭建应用的框架；jQuery 是简化「HTML 遍历与操作，事件处理，动画以及 AJAX」的库。这是两者最核心的区别，Angular 关注的是应用的架构，并不是 HTML 页面。

为了能够真正理解如何用 Angular 来搭建应用，那么请不要用 jQuery，因为 jQuery 让开发者以现有的 HTML 标准的思维考量问题；Angular 则让你「可以为你的应用对 HTML 进行扩展」。

DOM 操作只应该在 directive 中发生，但这并不意味着它们必须是 jQuery 包裹后的对象。如果你有想用 jQuery 的冲动，请先考虑 Angular 提供的功能是否满足你的需求。一般情况下，directive 就够了，我们可以用它做很多事情。

也许，随着项目的增长，有一天可能不得不用 jQuery，但项目的一开始就考虑把 jQuery 引进来却是不可取的。

# 总结

Angular 是一个超级棒的框架，而且还在不停地进化。希望这些意见可以避免项目开发过程中的陷阱。
