# Angular 1 Style Guide

## Sources / Inspiration

  - [@toddmotto's styleguide](https://github.com/toddmotto/angular-styleguide)
  - [John Papa's styleguide](https://github.com/johnpapa/angular-styleguide)
  - [A better way to $scope, angular.extend, no more “vm = this”](https://toddmotto.com/a-better-way-to-scope-angular-extend-no-more-vm-this/)

## Rule of all rules

  - Use your judgement to determine if the given rule makes sense in your situation – don't use these rules just because they're rules, use them to make your and other developers' life easier!

## Basics
### Rule of 1

  - Define 1 component per file, recommended to be less than 500 lines of code.

  *Why?*: One component per file promotes easier unit testing and mocking.

  *Why?*: One component per file makes it far easier to read, maintain, and avoid collisions with teams in source control.

  *Why?*: One component per file avoids hidden bugs that often arise when combining components in a file where they may share variables, create unwanted closures, or unwanted coupling with dependencies.

  The following example defines the `app` module and its dependencies, defines a controller, and defines a factory all in the same file.

  ```js
  /* avoid */
  angular
      .module('app', ['ngRoute'])
      .controller('SomeController', SomeController)
      .factory('someFactory', someFactory)

  function SomeController() { }

  function someFactory() { }
  ```

  The same components are now separated into their own files.

  ```js
  /* recommended */

  // app.module.js
  angular
      .module('app', ['ngRoute'])
  ```

  ```js
  /* recommended */

  // some.controller.js
  angular
      .module('app')
      .controller('SomeController', SomeController)

  function SomeController() { }
  ```

  ```js
  /* recommended */

  // someFactory.js
  angular
      .module('app')
      .factory('someFactory', someFactory)

  function someFactory() { }
  ```


### Small Functions

  - Define small functions, no more than 75 LOC (less is better).

  *Why?*: Small functions are easier to test, especially when they do one thing and serve one purpose.

  *Why?*: Small functions promote reuse.

  *Why?*: Small functions are easier to read.

  *Why?*: Small functions are easier to maintain.

  *Why?*: Small functions help avoid hidden bugs that come with large functions that share variables with external scope, create unwanted closures, or unwanted coupling with dependencies.

---

## Modules

### Definition
- Declare modules without a variable using the setter and getter syntax

  ```js
  // avoid
  var app = angular.module('app', [])
  app.controller()
  app.factory()

  // recommended
  angular
    .module('app', [])
    .controller()
    .factory()
  ```

- Note: Using `angular.module('app', [])` sets a module, whereas `angular.module('app')` gets the module. Only set once and get for all other instances.


### Methods

  - Pass functions into module methods rather than assign as a callback

  ```js
  // avoid
  angular
    .module('app', [])
    .controller('MainCtrl', function MainCtrl () {

    })
    .service('SomeService', function SomeService () {

    })

  // recommended
  function MainCtrl () {

  }
  function SomeService () {

  }
  angular
    .module('app', [])
    .controller('MainCtrl', MainCtrl)
    .service('SomeService', SomeService)
  ```

  - ES6 Classes are not hoisted, which will break your code if you rely on hoisting

  - This aids with readability and reduces the volume of code "wrapped" inside the Angular framework


### IIFE scoping

  - To avoid polluting the global scope with our function declarations that get passed into Angular, ensure build tasks wrap the concatenated files inside an IIFE

  ```js
  (function () {

    angular
      .module('app', [])

    // MainCtrl.js
    function MainCtrl () {

    }

    angular
      .module('app')
      .controller('MainCtrl', MainCtrl)

    // SomeService.js
    function SomeService () {

    }

    angular
      .module('app')
      .service('SomeService', SomeService)

    // ...

  })()
  ```


### Avoid Naming Collisions

  - Use unique naming conventions with separators for sub-modules.

  *Why?*: Unique names help avoid module name collisions. Separators help define modules and their submodule hierarchy. For example `app` may be your root module while `app.dashboard` and `app.users` may be modules that are used as dependencies of `app`.

---

## Controllers

### controllerAs syntax

  - There are a lots of discussion about using `controllerAs` syntax. Use it when there are more then 1 controller in the DOM (besides the main app controller), otherwise it can be optional
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

  - *Note:* Upon using `controllerAs` syntax, `$scope` gets bound to `this`, and in this case you can do stuff like:

  ```js
  function MainCtrl () {
    this.someVar = {
      name: 'Todd'
    }
    this.anotherVar = []
    this.doSomething = function doSomething() {

    }
  }

  angular
    .module('app')
    .controller('MainCtrl', MainCtrl)
  ```
### Using angular.extend

- You can drop using this or $scope, check these articles by [@toddmotto](https://toddmotto.com/a-better-way-to-scope-angular-extend-no-more-vm-this/) and [Modus Create](http://moduscreate.com/angularjs-tricks-with-angular-extend/)
  ```js
  /**
   * avoid
   */
  function MainCtrl () {
    this.someVar = {
      name: 'Todd'
    }
    this.anotherVar = []
    this.doSomething = function doSomething() {

    }
  }

  /**
   * recommended
   */
  function MainCtrl ($scope) {

    // private
    function someMethod() {

    }

    // public
    var someVar = { name: 'Todd' }
    var anotherVar = []
    function doSomething() {
      someMethod()
    }

    // exports
    angular.extend($scope, {
      someVar: someVar,
      anotherVar: anotherVar,
      doSomething: doSomething
    })
  }
  ```

### Presentational logic only (MVVM)

- Presentational logic only inside a controller, avoid Business logic (delegate to Services)

    ```js
    // avoid
    function MainCtrl ($scope) {

      var users

      $http
        .get('/users')
        .success(function (response) {
          var users = response
        })

      function removeUser (user, index) {
        $http
          .delete('/user/' + user.id)
          .then(function (response) {
            users.splice(index, 1)
          })
      }

      angular.extend($scope, {
        users : users,
        removeUser : removeUser
      })

    }

    // recommended
    function MainCtrl (UserService, $scope) {

      var users

      UserService
        .getUsers()
        .then(function (response) {
          var users = response
        })

      function removeUser (user, index) {
        UserService
          .removeUser(user)
          .then(function (response) {
            users.splice(index, 1)
          })
      }

      angular.extend($scope, {
        users : users,
        removeUser : removeUser
      })

    }
    ```

    *Why?* : Controllers should fetch Model data from Services, avoiding any Business logic. Controllers should act as a ViewModel and control the data flowing between the Model and the View presentational layer. Business logic in Controllers makes testing Services impossible.

---

## Resolving Promises

### Route Resolve Promises

  - **Promises**: Resolve Controller dependencies in the `$routeProvider` (or `$stateProvider` for `ui-router`), not the Controller itself

    ```js
    // avoid
    function MainCtrl (SomeService) {
      var _this = this
      // unresolved
      _this.something
      // resolved asynchronously
      SomeService.doSomething().then(function (response) {
        _this.something = response
      })
    }
    angular
      .module('app')
      .controller('MainCtrl', MainCtrl)

    // recommended
    function config ($routeProvider) {
      $routeProvider
      .when('/', {
        templateUrl: 'views/main.html',
        resolve: {
          // resolve here
        }
      })
    }
    angular
      .module('app')
      .config(config)
    ```

  - **Controller.resolve property**: Never bind logic to the router itself. Reference a `resolve` property for each Controller to couple the logic

    ```js
    // avoid
    function MainCtrl (SomeService) {
      this.something = SomeService.something
    }

    function config ($routeProvider) {
      $routeProvider
      .when('/', {
        templateUrl: 'views/main.html',
        controller: 'MainCtrl'
        resolve: {
          doSomething: function () {
            return SomeService.doSomething()
          }
        }
      })
    }

    // recommended
    function MainCtrl (SomeService) {
      this.something = SomeService.something
    }

    MainCtrl.resolve = {
      doSomething: function (SomeService) {
        return SomeService.doSomething()
      }
    }

    function config ($routeProvider) {
      $routeProvider
      .when('/', {
        templateUrl: 'views/main.html',
        controller: 'MainCtrl'
        resolve: MainCtrl.resolve
      })
    }
    ```

  - This keeps resolve dependencies inside the same file as the Controller and the router free from logic

---

## Dircetives and Components

 - TBD, needs to migrate current stuff to [components](https://toddmotto.com/exploring-the-angular-1-5-component-method/)

---

## Services and Factory

  - All Angular Services are singletons, using `.service()` or `.factory()` differs the way Objects are created.

  - Refactor logic for making data operations and interacting with data to a factory. Make data services responsible for XHR calls, local storage, stashing in memory, or any other data operations.

    *Why?*: The controller's responsibility is for the presentation and gathering of information for the view. It should not care how it gets the data, just that it knows who to ask for it. Separating the data services moves the logic on how to get it to the data service, and lets the controller be simpler and more focused on the view.

    *Why?*: This makes it easier to test (mock or real) the data calls when testing a controller that uses a data service.

    *Why?*: Data service implementation may have very specific code to handle the data repository. This may include headers, how to talk to the data, or other services such as `$http`. Separating the logic into a data service encapsulates this logic in a single place hiding the implementation from the outside consumers (perhaps a controller), also making it easier to change the implementation.

### Services

  - Services act as a `constructor` function and are instantiated with the `new` keyword. Use `this` for public methods and variables

      ```js
      function SomeService () {
        this.someMethod = function () {

        }
      }
      angular
        .module('app')
        .service('SomeService', SomeService)
      ```

### Factory

  - Business logic or provider modules, return an Object or closure

  - Factories should have a [single responsibility](https://en.wikipedia.org/wiki/Single_responsibility_principle), that is encapsulated by its context. Once a factory begins to exceed that singular purpose, a new factory should be created.

  - Always return a host Object instead of the revealing Module pattern due to the way Object references are bound and updated

  ```js
  function AnotherService () {
    var AnotherService = {}
    AnotherService.someValue = ''
    AnotherService.someMethod = function () {

    }
    return AnotherService
  }
  angular
    .module('app')
    .factory('AnotherService', AnotherService)
  ```

  *Why?* : Primitive values cannot update alone using the revealing module pattern

### Function Declarations to Hide Implementation Details

  - Use function declarations to hide implementation details. Keep your accessible members of the factory up top. Point those to function declarations that appears later in the file. For more details see [this post](http://www.johnpapa.net/angular-function-declarations-function-expressions-and-readable-code).

    *Why?*: Placing accessible members at the top makes it easy to read and helps you instantly identify which functions of the factory you can access externally.

    *Why?*: Placing the implementation details of a function later in the file moves that complexity out of view so you can see the important stuff up top.

    *Why?*: Function declaration are hoisted so there are no concerns over using a function before it is defined (as there would be with function expressions).

    *Why?*: You never have to worry with function declarations that moving `var a` before `var b` will break your code because `a` depends on `b`.

    *Why?*: Order is critical with function expressions

  ```javascript
  /**
   * avoid
   * Using function expressions
   */
   function dataservice($http, $location, $q, exception, logger) {
      var isPrimed = false
      var primePromise

      var getAvengers = function() {
          // implementation details go here
      }

      var getAvengerCount = function() {
          // implementation details go here
      }

      var getAvengersCast = function() {
         // implementation details go here
      }

      var prime = function() {
         // implementation details go here
      }

      var ready = function(nextPromises) {
          // implementation details go here
      }

      var service = {
          getAvengersCast: getAvengersCast,
          getAvengerCount: getAvengerCount,
          getAvengers: getAvengers,
          ready: ready
      }

      return service
  }
  ```

  ```javascript
  /**
   * recommended
   * Using function declarations
   * and accessible members up top.
   */
  function dataservice($http, $location, $q, exception, logger) {
      var isPrimed = false
      var primePromise

      var service = {
          getAvengersCast: getAvengersCast,
          getAvengerCount: getAvengerCount,
          getAvengers: getAvengers,
          ready: ready
      }

      return service

      ////////////

      function getAvengers() {
          // implementation details go here
      }

      function getAvengerCount() {
          // implementation details go here
      }

      function getAvengersCast() {
          // implementation details go here
      }

      function prime() {
          // implementation details go here
      }

      function ready(nextPromises) {
          // implementation details go here
      }
  }
  ```

---

## Miscellaneous

### Publish and subscribe events

  - **$scope**: Use the `$emit` and `$broadcast` methods to trigger events to direct relationship scopes only

    ```js
    // up the $scope
    $scope.$emit('customEvent', data)

    // down the $scope
    $scope.$broadcast('customEvent', data)
    ```

  - **$rootScope**: Use only `$emit` as an application-wide event bus and remember to unbind listeners

    ```js
    // all $rootScope.$on listeners
    $rootScope.$emit('customEvent', data)
    ```

  - Hint: Because the `$rootScope` is never destroyed, `$rootScope.$on` listeners aren't either, unlike `$scope.$on` listeners and will always persist, so they need destroying when the relevant `$scope` fires the `$destroy` event

    ```js
    // call the closure
    var unbind = $rootScope.$on('customEvent'[, callback])
    $scope.$on('$destroy', unbind)
    ```

  - For multiple `$rootScope` listeners, use an Object literal and loop each one on the `$destroy` event to unbind all automatically

    ```js
    var unbind = [
      $rootScope.$on('customEvent1'[, callback]),
      $rootScope.$on('customEvent2'[, callback]),
      $rootScope.$on('customEvent3'[, callback])
    ]
    $scope.$on('$destroy', function () {
      unbind.forEach(function (fn) {
        fn()
      })
    })
    ```

### Angular wrapper references

  - **$document and $window**: Use `$document` and `$window` at all times to aid testing and Angular references

    ```js
    // avoid
    function dragUpload () {
      return {
        link: function ($scope, $element, $attrs) {
          document.addEventListener('click', function () {

          })
        }
      }
    }

    // recommended
    function dragUpload ($document) {
      return {
        link: function ($scope, $element, $attrs) {
          $document.addEventListener('click', function () {

          })
        }
      }
    }
    ```

  - **$timeout and $interval**: Use `$timeout` and `$interval` over their native counterparts to keep Angular's two-way data binding up to date

    ```js
    // avoid
    function dragUpload () {
      return {
        link: function ($scope, $element, $attrs) {
          setTimeout(function () {
            //
          }, 1000)
        }
      }
    }

    // recommended
    function dragUpload ($timeout) {
      return {
        link: function ($scope, $element, $attrs) {
          $timeout(function () {
            //
          }, 1000)
        }
      }
    }
    ```

### Comment standards

  - **jsDoc**: Use jsDoc syntax to document function names, description, params and returns

    ```js
    /**
     * @name SomeService
     * @desc Main application Controller
     */
    function SomeService (SomeService) {

      /**
       * @name doSomething
       * @desc Does something awesome
       * @param {Number} x - First number to do something with
       * @param {Number} y - Second number to do something with
       * @returns {Number}
       */
      this.doSomething = function (x, y) {
        return x * y
      }

    }
    angular
      .module('app')
      .service('SomeService', SomeService)
    ```

### Use Gulp or Grunt for ng-annotate

  - Use [gulp-ng-annotate](https://www.npmjs.com/package/gulp-ng-annotate)  in an automated build task. Inject `/* @ngInject */` prior to any function that has dependencies.

    *Why?*: ng-annotate will catch most dependencies, but it sometimes requires hints using the `/* @ngInject */` syntax.

    The following code is an example of a gulp task using ngAnnotate

    ```javascript
    gulp.task('js', ['jshint'], function() {
        var source = pkg.paths.js

        return gulp.src(source)
            .pipe(sourcemaps.init())
            .pipe(concat('all.min.js', {newLine: ''}))
            // Annotate before uglify so the code get's min'd properly.
            .pipe(ngAnnotate({
                // true helps add where @ngInject is not used. It infers.
                // Doesn't work with resolve, so we must be explicit there
                add: true
            }))
            .pipe(bytediff.start())
            .pipe(uglify({mangle: true}))
            .pipe(bytediff.stop())
            .pipe(sourcemaps.write('./'))
            .pipe(gulp.dest(pkg.paths.dev))
    })

    ```
### Structuring the app

  - [Angular App Structuring Guidelines](http://www.johnpapa.net/angular-app-structuring-guidelines/)
