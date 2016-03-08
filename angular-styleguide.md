# Angular 1 Style Guide

## Sources / Inspiration

  - [@toddmotto's styleguide](https://github.com/toddmotto/angular-styleguide)
  - [John Papa's styleguide](https://github.com/johnpapa/angular-styleguide)
  - [A better way to $scope, angular.extend, no more “vm = this”](https://toddmotto.com/a-better-way-to-scope-angular-extend-no-more-vm-this/)

## Rule of all rules

  - Use your judgement to determine if the given rule makes sense in your situation – don't use these rules just because they're rules, use them to make your and other developers' life easier!

## Single Responsibility
### Rule of 1

  - Define 1 component per file, recommended to be less than 500 lines of code.

  *Why?*: One component per file promotes easier unit testing and mocking.

  *Why?*: One component per file makes it far easier to read, maintain, and avoid collisions with teams in source control.

  *Why?*: One component per file avoids hidden bugs that often arise when combining components in a file where they may share variables, create unwanted closures, or unwanted coupling with dependencies.

  The following example defines the `app` module and its dependencies, defines a controller, and defines a factory all in the same file.

  ```javascript
  /* avoid */
  angular
      .module('app', ['ngRoute'])
      .controller('SomeController', SomeController)
      .factory('someFactory', someFactory);

  function SomeController() { }

  function someFactory() { }
  ```

  The same components are now separated into their own files.

  ```javascript
  /* recommended */

  // app.module.js
  angular
      .module('app', ['ngRoute']);
  ```

  ```javascript
  /* recommended */

  // some.controller.js
  angular
      .module('app')
      .controller('SomeController', SomeController);

  function SomeController() { }
  ```

  ```javascript
  /* recommended */

  // someFactory.js
  angular
      .module('app')
      .factory('someFactory', someFactory);

  function someFactory() { }
  ```


### Small Functions

  - Define small functions, no more than 75 LOC (less is better).

  *Why?*: Small functions are easier to test, especially when they do one thing and serve one purpose.

  *Why?*: Small functions promote reuse.

  *Why?*: Small functions are easier to read.

  *Why?*: Small functions are easier to maintain.

  *Why?*: Small functions help avoid hidden bugs that come with large functions that share variables with external scope, create unwanted closures, or unwanted coupling with dependencies.


## Modules

### Definition
- Declare modules without a variable using the setter and getter syntax

  ```javascript
  // avoid
  var app = angular.module('app', []);
  app.controller();
  app.factory();

  // recommended
  angular
    .module('app', [])
    .controller()
    .factory();
  ```

- Note: Using `angular.module('app', []);` sets a module, whereas `angular.module('app');` gets the module. Only set once and get for all other instances.


### Methods

  - Pass functions into module methods rather than assign as a callback

  ```javascript
  // avoid
  angular
    .module('app', [])
    .controller('MainCtrl', function MainCtrl () {

    })
    .service('SomeService', function SomeService () {

    });

  // recommended
  function MainCtrl () {

  }
  function SomeService () {

  }
  angular
    .module('app', [])
    .controller('MainCtrl', MainCtrl)
    .service('SomeService', SomeService);
  ```

  - ES6 Classes are not hoisted, which will break your code if you rely on hoisting

  - This aids with readability and reduces the volume of code "wrapped" inside the Angular framework


### IIFE scoping

  - To avoid polluting the global scope with our function declarations that get passed into Angular, ensure build tasks wrap the concatenated files inside an IIFE

  ```javascript
  (function () {

    angular
      .module('app', []);

    // MainCtrl.js
    function MainCtrl () {

    }

    angular
      .module('app')
      .controller('MainCtrl', MainCtrl);

    // SomeService.js
    function SomeService () {

    }

    angular
      .module('app')
      .service('SomeService', SomeService);

    // ...

  })();
  ```


### Avoid Naming Collisions

  - Use unique naming conventions with separators for sub-modules.

  *Why?*: Unique names help avoid module name collisions. Separators help define modules and their submodule hierarchy. For example `app` may be your root module while `app.dashboard` and `app.users` may be modules that are used as dependencies of `app`.


## Controllers

### controllerAs syntax

- There are a lots of discussion about using `controllerAs` syntax. Use it when there are more then 1 controller in the DOM (besides the main app controller).
- In the DOM we get a variable per controller, which aids nested controller methods, avoiding any `$parent` calls

  ```html
  <!-- avoid -->
  <div ng-controller="MainCtrl">
    {{ someObject }}
  </div>

  <!-- recommended -->
  <div ng-controller="MainCtrl as vm">
    {{ vm.someObject }}
  </div>
  ```
