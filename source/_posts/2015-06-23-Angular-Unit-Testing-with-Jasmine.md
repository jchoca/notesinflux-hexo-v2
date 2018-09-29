title: Angular 1.x Unit Testing with Jasmine
tags:
  - jasmine
  - angularjs
comments: true
date: 2015-06-23 19:31:24
---

Testing in Angular seems to be a sticking point for a lot of developers and rightly so.  I haven't been able to find a real solid reference or comprehensive guide for testing in Angular with Jasmine in conjunction with the [John Papa Angular Style Guide](https://github.com/johnpapa/angular-styleguide), so I decided to start creating one.  This guide assumes no previous experience with unit testing, but it does assume familiarity/experience with Angular JS.

## Background
Wikipedia defines a unit test as follows:
>In computer programming, unit testing is a software testing method by which individual units of source code, sets of one or more computer program modules together with associated control data, usage procedures, and operating procedures, are tested to determine whether they are fit for use.

We essentially test a block of code (e.g., a module, controller, service, etc.) in a vacuum.  We set the stage by mocking dependencies and simulating the conditions we'd like to test, then we make assertions about how our code should behave.  Let's take a look at a simple example.  Here is a simple multiplication module:

```javascript
// multiplication.js
var Multiplication = (function () {
  function multiplyBy2(val) {
    return val*2;
  }
  
  function multiplyBy3(val) {
    return val*4; // typo here will cause this function to behave incorrectly
  }
  
  return {
    multiplyBy2: multiplyBy2,
    multiplyBy3: multiplyBy3
  }
})();
```

and here is a corresponding Jasmine test file:

```javascript
// multiplication.spec.js
describe("multiplication", function() {
  var testNum;
  
  beforeEach(function () {
    // put setup code here
    testNum = 3;
  })
  
  describe('method multiplyBy2', function() {
    it('should multiply its argument by 2', function() {
      expect(Multiplication.multiplyBy2(testNum)).toEqual(6);
    });
  })

  describe('method multiplyBy3', function() {
    it('should multiply its argument by 3', function() {
      // fails due to typo above
      expect(Multiplication.multiplyBy3(testNum)).toEqual(9);
    });
  });
});
```
http://plnkr.co/edit/FO47r4yXfzM5lWQGWoEG?

Check out the plunk to see how it's set up.  We have two different test cases, marked by the 'it' statement.  You can nest describe and it statements, and the descriptions will concatenate.  For example, the first test case would read "multiplication multiplyBy2 should multiply its argument by 2."

Testing in Angular is similar to the example above, except that we'll need to use the [https://docs.angularjs.org/api/ngMock](ngMock) module so that we can inject and mock angular services and modules.  When I write Angular code, I mostly follow the [John Papa Style Guide](https://github.com/johnpapa/angular-styleguide), which says you should use controllerAs syntax (with bindToController, if needed).

## Controllers

Testing controllers is probably the most straight-forward in Angular.  We load the module that our controller is associated with, then we mock any dependencies our controller has, and finally we instantiate the controller.  I've created another plunk (here: http://plnkr.co/edit/RpPAl5oRaXFB96XNB4KD) that I'll be using examples from.  Here's the controller and test file:

```javascript
// myCtrl.controller.js

(function() {
  'use strict';
  
  // Load Angular module and add controller
  angular
    .module('myApp')
    .controller('MyCtrl', MyCtrl);
  
  // Simple controller that uses myService
  MyCtrl.$inject = ['myService'];
  function MyCtrl(myService) {
    var myCtrl = this;
    
    // Declare bindable members here
    myCtrl.hello = "Hello World";
    myCtrl.anotherHello = myService.hello();
  }
})();
```
&nbsp;
```javascript
// myCtrl.controller.spec.js

describe("myCtrl", function() {
  var myCtrl, mockService;
  
  // Load the module
  beforeEach(module('myApp'));
  
  beforeEach(inject(function ($controller) {
    // Create a mocked version of myService
    mockService = {
      // Using a spy here to track calls to mockService.hello()
      hello: jasmine.createSpy('myService')
    };
    // Instantiate the controller using $controller service,
    // stubbing in mockService for myService
    myCtrl = $controller('MyCtrl', { myService: mockService });
  }));
  
  it('should set the hello property', function () {
    expect(myCtrl.hello).toEqual('Hello World');
  });
  
  it('should call myService.hello', function () {
    expect(mockService.hello).toHaveBeenCalled();
  });
});
```

## Services

Services are similar to controllers, except there's no helper function for instantiating them, which means we have to mock dependencies a little differently.  Here's my simple example service with test file:

```javascript
// myService.service.js

(function() {
  'use strict';

  angular
    .module('myApp')
    .service('myService', myService);
  
  myService.$inject = ['myOtherService'];
  function myService(myOtherService) {
    function hello() {
      return "Hello";
    }

    return {
      hello: hello
    };
  }
})();
```
&nbsp;
```javascript
// myService.service.spec.js

describe('myService', function () {
  var myService, mockMyOtherService = {};
  
  beforeEach(module('myApp'));
  
  beforeEach(module(function($provide) {
    $provide.value('myOtherService', mockMyOtherService);
  }));
  
  beforeEach(inject(function(_myService_) {
    // Use injector to get the service
    myService = _myService_;
  }));
  
  describe('hello', function () {
    it('should return hello', function () {
      expect(myService.hello()).toEqual("Hello");
    });
  });
});
```

Here we're using $provide to "register" a provider in Angular world.  We tell it the name of the dependency we are providing (myOtherService in this case) along with the mocked service (in this case just an empty object).  It's important to understand here that we're just registering the provider so Angular knows about it.  We can even change it after the fact like this:

```javascript
describe('myService', function () {
  var myService, mockMyOtherService = {};
  
  beforeEach(module('myApp'));
  
  beforeEach(module(function($provide) {
    $provide.value('myOtherService', mockMyOtherService);
  }));
  
  beforeEach(inject(function(_myService_) {
    // Use injector to get the service
    myService = _myService_;
  }));
  
  describe('hello', function () {
    it('should return hello and call myOtherService.someMethod', function () {
      mockMyOtherService.someMethod = jasmine.createSpy('someMethod');
      expect(myService.hello()).toEqual("Hello");
      expect(mockMyOtherService.someMethod).toHaveBeenCalled();
    });
  });
});
```

## Directives

Directives are a little trickier.  Since directives are essentially reusable components with custom behavior, they inherently mix presentation (template) with display logic (controller or linking function).  Things get interesting when you're using controllerAs syntax, where the controller is a property of the directive/element scope.  Here's my directive, using both a post-link function and controller (http://plnkr.co/edit/IeVdEwmNWcfF8wnnx9i2):

```javascript
// myDirective.directive.js

(function() {
  'use strict';
  
  angular
    .module('myApp')
    .directive('myDirective', myDirective);
    
  myDirective.$inject = ['myService'];
  function myDirective(myService) {
    var directive = {
      restrict: 'E',
      template: '<div>Hello</div>',
      controller: MyDirectiveCtrl,
      controllerAs: 'myDirectiveCtrl',
      scope: {},
      bindToController: true,
      link: linkFn
    }
    
    return directive;
    
    function linkFn(scope) {
      scope.myDirectiveCtrl.goodbye = myService.goodbye(); // Goodbye
      console.log(scope.myDirectiveCtrl.hello); // Hello
    }
  }
  
  MyDirectiveCtrl.$inject = ['myService'];
  function MyDirectiveCtrl(myService) {
    var myDirectiveCtrl = this;
    
    // bindables go here
    myDirectiveCtrl.hello = null;
    
    activate();
    
    function activate() {
      myDirectiveCtrl.hello = myService.hello(); // Hello
    }
  }
})();
```
The controller is accessible in the linking function as a scope property as shown above.  Likewise, $scope could be accessible in the controller by injecting it.

```javascript
// myDirective.directive.spec.js

describe("myDirective", function() {
  var myDirectiveCtrl, mockService = {}, $compile, $rootScope, element;
  
  // Load the module
  beforeEach(module('myApp'));
  
  // Register providers
  beforeEach(function () {
    module('myApp', function ($provide) {
      $provide.value('myService', mockService);
    })
  })
  
  // Add content to providers (mock the dependencies)
  beforeEach(function () {
    mockService = {
      hello: jasmine.createSpy('myServiceHello').and.returnValue("Hello"),
      goodbye: jasmine.createSpy('myServiceGoodbye').and.returnValue("Goodbye")
    };
  });
  
  // instantiate the directive
  beforeEach(inject(function (_$compile_, _$rootScope_) {
    $compile = _$compile_;
    $rootScope = _$rootScope_;

    // wrap the directive in an angular element, then compile it
    element = angular.element('<my-directive></my-directive>');
    $compile(element)($rootScope);
    $rootScope.$digest();

    // retrieve the controller for the directive
    // see "Extras" section here https://docs.angularjs.org/api/ng/function/angular.element
    myDirectiveCtrl = element.controller('myDirective');
  }));
  
  it('consist of a div', function () {
    expect(element.html()).toEqual('<div>Hello</div>');
  });
  
  describe('link function', function () {
    it('should call my service goodbye', function () {
      expect(mockService.goodbye).toHaveBeenCalled();
    })
    
    it('should set the goodbye property', function () {
      expect(myDirectiveCtrl.goodbye).toEqual("Goodbye");
    })
  });
  
  describe('myDirectiveCtrl', function () {
    it('should invoke myService.hello', function () {
      expect(mockService.hello).toHaveBeenCalled();
    });
    
    it('should set the hello property', function () {
      expect(myDirectiveCtrl.hello).toEqual("Hello");
    })
  });
});
```