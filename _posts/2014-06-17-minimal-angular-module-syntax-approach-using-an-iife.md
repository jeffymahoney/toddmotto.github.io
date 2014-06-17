---
layout: post
permalink: /minimal-angular-module-syntax-approach-using-an-iife
title: Minimal Angular module/syntax approach using an IIFE
path: 2014-06-17-minimal-angular-module-syntax-approach-using-an-iife.md
---

Since day one using Angular, I've always debated on _how_ I should be extending modules. This post talks through some common Angular patterns, from variable to chaining and a variant.

The options are via a variable, such as `var myApp = angular.module('myApp', []);`, or using the "getter" method of the module from Angular via `angular.module('myApp')` and chaining our methods. I've decided on the latter, but with a nice twist to smarten up our JavaScript inside Angular using an IIFE and minimising Angular footprint.

### Using a variable
The variable approach isn't the preferred way of dealing with modules according to [Brian Ford](https://github.com/btford/ngmin#references) on the Angular team, so I've tried to break away from it. The variable approach is simple to declare and use:

{% highlight javascript %}
var app = angular.module('app', []);

app.controller('MainCtrl', function MainCtrl ($scope, SomeFactory) {
  this.doSometing = function () {
    SomeFactory.doSomething();
  };
});
{% endhighlight %}

I'm not entirely sure what the implications are from Brian's advice, so far I've not experienced any drawbacks with the applications themselves or writing unit tests.

Initially I opted for those approach as our workflow changed and we were using Grunt (now Gulp) file concats and minifications - of which we immediately split things into fresh files for everything to keep things modular and clean. The easiest way to grab the module we needed was using a variable, because chaining our module then method calls were a little unclean.

### Using chains

Chaining is really smart, it looks a lot more modern as well and very clean:

{% highlight javascript %}
// set the module
angular.module('app', []);

// get the module
angular.module('app').controller('MainCtrl', function MainCtrl ($scope, SomeFactory) {
  this.doSometing = function () {
    SomeFactory.doSomething();
  };
});
{% endhighlight %}

You could argue this isn't that different from the above, but variables are also minified whereas the `angular.module('app')` gets aren't - though that isn't a huge concern. We can also do this though and chain, chain, chain:

{% highlight javascript %}
angular.module('app', []);

angular.module('app')
.controller('MainCtrl', function MainCtrl ($scope, SomeFactory) {
  this.doSometing = function () {
    SomeFactory.doSomething();
  };
})
.directive('someDirective', function someDirective ($scope, SomeFactory) {
  this.doSometing = function () {
    SomeFactory.doSomething();
  };
});
{% endhighlight %}

Nice and very concise.

### Anonymous functions
I've [written about anonymous functions](http://toddmotto.com/avoiding-anonymous-javascript-functions) before and how they can clean up our code, aid debugging (through better stack traces and function naming) and more - so for now I'd been doing this and naming my anonymous functions to counter some of the arguments:

{% highlight javascript %}
// yes, shame on me for the variable
var app = angular.module('app', []);

// indent the anonymous function and name it `MainCtrl` for visibility
app.controller('MainCtrl',
  function MainCtrl ($scope, SomeFactory) {
  this.doSometing = function () {
    SomeFactory.doSomething();
  };
});
{% endhighlight %}

So what about moving it outside into it's own function to make Angular less visible and my JavaScript more "standalone":

{% highlight javascript %}
function MainCtrl ($scope, SomeFactory) {
  this.doSometing = function () {
    SomeFactory.doSomething();
  };
}
app.controller('MainCtrl', MainCtrl);
{% endhighlight %}

### Introducing an IIFE

In the global scope the above could cause some confusion for Angular as it picks up functions as Controllers if the names match, so to avoid this we can hide it inside an IIFE.

{% highlight javascript %}
(function () {
  function MainCtrl ($scope, SomeFactory) {
    this.doSometing = function () {
      SomeFactory.doSomething();
    };
  }
  app.controller('MainCtrl', MainCtrl);
})();
{% endhighlight %}

Better! Now we integrate into Angular and nothing is public. But what about the `app` variable referencing my Module? I think `app` is pretty naff anyway, but it's simple and I don't need to change it per project as it's pretty generic. But maybe it's _too_ generic and needs a change. So a module name in the way Brian recommends would be best.

Let's assume a module is setup, I can then bolt into it:

{% highlight javascript %}
(function () {
  function MainCtrl ($scope, SomeFactory) {
    this.doSometing = function () {
      SomeFactory.doSomething();
    };
  }
  angular.module('app').controller('MainCtrl', MainCtrl);
})();
{% endhighlight %}

Looking really nice. If I want to convert my scripts at some point into a bigger module, add more methods and extend:

{% highlight javascript %}
(function () {
  function MainCtrl ($scope, SomeFactory) {
    this.doSometing = function () {
      SomeFactory.doSomething();
    };
  }
  function someDirective () {
    return {
      restrict: 'EA',
      replace: true,
      scope: {},
      template: [
        '<div class="someDirective"></div>'
      ].join(''),
      controllerAs: 'someDirectiveCtrl',
      controller: function () {}
      link: function () {}
    };
  }
  angular.module('app').controller('MainCtrl', MainCtrl).directive('someDirective', someDirective);
})();
{% endhighlight %}

Chaining gets quite long though, let's neaten things up, add a `use strict` statement and we're done:

{% highlight javascript %}
(function () {

  'use strict';

  function MainCtrl ($scope, SomeFactory) {
    this.doSometing = function () {
      SomeFactory.doSomething();
    };
  }

  function someDirective () {
    return {
      restrict: 'EA',
      replace: true,
      scope: {},
      template: [
        '<div class="someDirective"></div>'
      ].join(''),
      controllerAs: 'someDirectiveCtrl',
      controller: function () {}
      link: function () {}
    };
  }

  angular
    .module('app')
    .controller('MainCtrl', MainCtrl)
    .directive('someDirective', someDirective);

})();
{% endhighlight %}

Very clean, shows my _intent_ whilst being very visible. I know exactly what this file contains and how to lookup the functions powering the module methods. I can scroll to the bottom of any file and see this. With function hoisting you could even add them to the top, but I prefer not too. Generally name the method you're passing into as similar as possible as your Angular's extension name to avoid confusion, things like `.controller('UserCtrl', myCoolUserFunc);` will not help anybody.

The other thing I like about this is it makes my Angular application seem more like _my_ application too, rather than it being fully encapsulated inside a tonne of Angular's syntax as we've seen before.

Unless creating a (fairly) small module, you might want to keep all files separate if you're automating file concats:

{% highlight javascript %}
// MainCtrl.js
(function () {

  'use strict';

  function MainCtrl ($scope, SomeFactory) {
    this.doSometing = function () {
      SomeFactory.doSomething();
    };
  }

  angular
    .module('app')
    .controller('MainCtrl', MainCtrl);

})();
// someDirective.js
(function () {

  'use strict';

  function someDirective () {
    return {
      restrict: 'EA',
      replace: true,
      scope: {},
      template: [
        '<div class="someDirective"></div>'
      ].join(''),
      controllerAs: 'someDirectiveCtrl',
      controller: function () {}
      link: function () {}
    };
  }

  angular
    .module('app')
    .directive('someDirective', someDirective);

})();
{% endhighlight %}

If you're globally wrapping your code with an IIFE automatically with something like Grunt/Gulp, then extra points for you. This means each file could look like this:

{% highlight javascript %}
// MainCtrl.js
function MainCtrl ($scope, SomeFactory) {
  this.doSometing = function () {
    SomeFactory.doSomething();
  };
}
angular
  .module('app')
  .controller('MainCtrl', MainCtrl);
{% endhighlight %}

...and be concatenated into one huge file encapsulated in an IIFE. Though beware of scoping/variable issues by doing that as any variables used inside the closure will suddenly be in the same scope _if_ declared outside of your Angular stuff. Inside individual IIFE's that hold each file, we can create very generic and basic variables which would be available in the lexical scope (quick example):

{% highlight javascript %}
// removing the IIFE might break any variables needing to be inside this scope
(function () {

  'use strict';

  /**
   * I can have stuff here and no other scope can see it!
   * though you can't refer to any Angular stuff like $scope
   * but for Objects or Arrays you could get clever wiht
   */

  function MainCtrl ($scope, SomeFactory) {
    this.doSometing = function () {
      SomeFactory.doSomething();
    };
  }

  angular
    .module('app')
    .controller('MainCtrl', MainCtrl);

})();
{% endhighlight %}