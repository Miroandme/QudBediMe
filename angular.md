# Unit-testing Restangular clients with Jasmine and Karma

## Technology stack

* [AngularJS](http://www.angularjs.org)
* [Restangular](https://github.com/mgonto/restangular)
* [Jasmine](http://jasmine.github.io)
* [Karma](http://karma-runner.github.io)
* [Lo-Dash](https://lodash.com)

## Approach

Let's face it, unit testing with Angular `$q` promises is a little tricky at times, especially when working on large scale applications. `undefined` results trickle through for obscure reasons, requests must be flushed at the appropriate time, and synchronization of what should be straightforward events, such as nested promises, can be especially painful.

The following is a proposed method of `$q` promise unit testing in Jasmine that leverages the ideals of AngularJS and also helps to alleviate the difficulties of unit testing asynchronous responses of API resources.

The primary concepts of the approach are:

* Separate AngularjS module for test fixtures
* Fixtures as services
* Immutable responses (minimize mocking)
* `describe()` block composition structure of type over behavior (will make more sense later)
* Minimize the amount of setup code at the assertion level

The code examples outline the "front back end" of a basic cooking recipe application and uses JSON fixtures for testing. Fixtures can be an appropriate solution for responses of all sizes, but if the responses are consistently small then a simple assertion-level variable will suffice.

```javascript
'use strict';

// pretend configuration
var app = angular.module('recipeApp', [])

app.service('User', function(Restangular, Recipes) {

  return Restangular.one('user', function(user) {
    return {
      getId: function() {
        return user.id
      }.
      getName: function() {
        return user.givenName + ' ' + user.familyName
      },

      getRecipes: function() {
        return Recipes.forUser(this.id).get()
      }
    }
  })

})

app.service('Recipes', function(Restangular) {
  
  Restangular.extendModel('recipe', function(model) {
    model.getName = function() {
      return model.name
    }

    model.getIngredients = function() {
      return model.ingredients
    }

    model.getTotalTime = function() {
      return model.totalTime
    }

    ...
  })
  
  this.forUser = function(userId) {
    return Restangular.one('recipe').one('user', userId)
  }

  this.all = function() {
    return Restangular.all('recipe')
  }

})
```

### Testing module

The testing module is important because it separates testing components from production ones. All of the resource fixtures and stubs should stem from this module. Defining the module is straightforward:

```javascript
'use strict';

angular.module('recipeAppMocks', [])
```

Next we will need to define some fixture services. We are keeping it simple and defining a fixture service for each response, but there is certainly room for improvement.  A notable advantage to the service fixture approach is that your tests won't have an internal dependence on server based tools such as Node which can sacrifice the transparency of the file system.

```javascript
angular.module('recipeAppMocks')
.service('UserStub', function() {
return {
   id: 'abc123',
   givenName: 'Master',
   familyName: 'Chef'
})
```

```javascript
angular.module('recipeAppMocks')
.service('User2Stub', function() {
return {
   id: 'xyz890',
   givenName: 'Fail',
   familyName: 'Chef'
})
```

```javascript
angular.module('recipeAppMocks')
.service('RecipeStub', function() {
return {
  id: 1,
  userId: 'abc123',
  name: 'Grilled cheese'
  ingredients: [
    'Bread',
    'Butter',
    'Cheese'
  ],
  totalTime: 5
})
```

To make the examples cleaner and easier to understand, we have also provided a small test utility for resolving Restangular resource promises.  Feel free to utilize this in your own unit tests:

```javascript
'use strict';

angular.module('recipeAppMocks')
.service('resourceTest', function($log) {
  return function(q, scope) {
    return {
      resolveResponse: function(promise) {
        var defer = q.defer()
        var promisedValue

        promise.then(function(value) {
          promisedValue = value
        })
        .catch(function(error) {
          $log.error('[restTest::resolvePromise] Failed to resolve promise', error)
        })

        defer.resolve()
        scope.$apply()

        return promisedValue
      },

      restResponse: function(resource) {
         // .plain() makes it way easier to compare responses
         // as they won't be extended with Restangular specific methods
        return this.resolveResponse(resource).plain()
      }
    }
  }
})
```

Once a stub module and some fixture services are defined, the relevant files must be included in the `files` block of your Karma configuration (snippet taken from Karma website)

_karma.conf.js_

```javascript
module.exports = function(config) {
  config.set({
    basePath: '../..',
    frameworks: ['jasmine'],
    //...
  });
};
```

### Jasmine Unit Tests

Fortunately it is possible to inject multiple Angular modules into our Jasmine testing suite at a time. Naming conflicts are possible although easily avoidable through conventions such as a simple postfix (`SomeService*Stub*`).

It's a lot to take in, but here is how it all comes together.  More details will follow after the code.

```javascript
'use strict';

describe('Recipe app', function() {
  
  // Angular core ($q is Angular's promise library inspired by q)
  var $q, $rootScope, $httpBackend

  // Service stubs
  var UserStub, User2Stub, RecipeStub

  // Test utility
  var resourceTest

  // Load production module and testing module into testing suite
  beforeEach(function() {
    angular.mock.module('recipeApp')
    angular.mock.module('recipeAppMocks') 
  })

  beforeEach(
    angular.mock.inject(function(_$q_, _$rootScope_, _$httpBackend_ ,_UserStub_, _User2Stub_,  _RecipeStub_, _resourceTest_) {
      // Angular Core
      $q           = _$q_
      $rootScope   = _$rootScope_
      $httpBackend = _$httpBackend_

      // Service stubs
      UserStub     = _UserStub_
      User2Stub    = _USer2Stub_
      RecipeStub   = _RecipeStub_

      // Test utility, note that we are passing in a mocked instance of $q and $scope
      resourceTest = _resourceTest_($q, $rootScope)
    })
  )

  // Helps identify some mysterious undefineds (e.g. when a request doesn't get flushed properly)
  afterEach(function() {
    $httpBackend.verifyNoOutstandingExpectation()
    $httpBackend.verifyNoOutstandingRequest()
  })

  describe('User service', function() {
    var user
    var prepareUserStub

    beforeEach(function() {
      // This should be called before your suite of tests (based on the state of a specific resource) execute
      prepareUserStub = function(userId, userStub) {
        // Insert any stub related configuration before the app loads
        angular.module('recipeApp').config(function(_AuthProvider_) {
          _AuthProvider_.setOAuthToken('1234456780')
        })

        $httpBackend.whenGET('http://foo.bar/api/user' + userId).respond(UserStub)
      }
    })

    describe('for any type of user', function() {
      var user

      beforeEach(
        prepareUserStub('abc123', UserStub)
      )

      beforeEach(
        // Note that the _actual_ service is injected here.  The responses will be intercepted through prepareUserStub
        angular.mock.inject(function(_User_) {
          user = _User_.get()

          // The request must be immediately flushed. It's beneficial that it is defined here because it is guaranteed to have a result by the time an assertion is reached (also, we shouldn't muddle up that logic with our assertion)
          $httpBackend.flush()
        })
      )

      describe('getId()', function() {
        it('should exist', function() {
          // resourceTest is the glue that ties everything together. The user resource response will be flushed by the time we reach this assertion block because of the beforeEach block above
          var actualUser = resourceTest.restResponse(user)

          expect(angular.isFunction(actualUser.getId)).toBe(true)
        })

        it('should return the property id', function() {
          // Once this assertion is reached, the user stub has been re-instantiated and has no recollection of what happened to it in the previous assertion. Immutability is adhered.
          var actualUser = resourceTest.restResponse(user)

          expect(actualUser.getId()).toEqual(actualUser.id)
        })
      })

      describe('getName()', function() {
        it('should exist', function() {
          var actualUser = resourceTest.restResponse(user)

          expect(angular.isFunction(actualUser.getName)).toBe(true)
        })

        it('should return the user's givenName appended to the user's familyName', function() {
          var actualUser = resourceTest.restResponse(user)

          expect(actualUser.getName()).toEqual(actualUser.givenName + ' ' + actualUser.familyName)
        })
      })

      describe('getRecipes()', function() {
        it('should exist', function() {
          var actualUser = resourceTest.restResponse(user)
          
          expect(angular.isFunction(actualUser.getRecipes)).toBe(true)  
        })
      })

    })

    // Note how the following describe blocks are organized such that the type of the user determines what assertions are relevant in this block.  Method stubs used at the assertion level are contextually relevant to the state of the user.

    describe('for users with recipes', function() {
      desribe('getRecipes()', function() {
        it('should provide a list of recipes created by the user', function() {
          var actualUser    = resourceTest.restResopnse(user)
          var actualRecipes = actualUser.getRecipes()

          expect(_.isEmpty(actualRecipes)).toBe(false)
          expect(_.every(actualRecipes, {id: 'abc123'})).toBe(true)
        })
      })
    })

    describe('for users without recipes', function() {
      describe('getRecipes()', function() {
        it('should provide an empty list of recipes', function() {
          var actualUser = resourceTest.restResopnse(user)
          var actualRecipes = actualUser.getRecipes()

          expect(_.isEmpty(actualRecipes)).toBe(true)
        })
      })
    })

  })

})
```

Let's break down the previous example and look at some of the high-level ideas and designs:

#### HTTP Stubs

The angular mock module prevents the `$http` service from making any external requests. In my opinion, this limitation is apt because external dependencies in client unit tests are never appropriate.

So how do we work with this? Angular's dependency injection model makes it very easy to provide mock implementations of nearly any service.  Angular Mock provides `$httpBackend` out-of-the-box for the purpose of stubbing HTTP resources.  Anywhere the `$http` service is used in your application (yes, even with Restangular), `$httpBackend` will stub out the responses to your specifications.

Note that in the example, the fixtures and stubs are created in the `prepareUserStub` function, which is in a `beforeEach` block.  Here, fixture loading and `$http` stubbing are tightly coupled. `prepareUserStub` accepts arguments for `userId` and a `userStub` override to improve flexibility. The primary advantage to this approach is that the fixtures have the freedom to change dynamically per `describe()` block, which, of course, allows `$http` stubs to change easier, too.

In summary, the main concepts are:

* `$http` fixtures are easily accessible and independent of one another
* Use stub constructor functions (like `prepareQuoteStub`) with parameters to improve contextual and functional control of fixtures

#### Test Organization

When stubbing `$http`, the organization of unit tests can inhibit flexibility.  The primary reason is that you essentially have one stub per assertion context, and often times the context of one stub can conflict with another.  An extreme example of this problem is defining the `prepareFooBar()` method as closely to the top of the test suite as possible, essentially locking all of the suites to a single stub.  To following organizational guidelines help to mitigate this issue:

* Create a fixture for each resource state you need to test (e.g., `UserWithRecipes`, `UserWithoutRecipes`, etc.)
* Define ‘prepareFooStub’ type methods often, preferably at every resource abstraction level
* Utilize customization / context definition parameters in your 'prepareFooStub' methods
* Push your ‘prepareFooStub’ methods as close to your assertions as possible:

  ```javascript
  …
  var foo

  beforeEach(
    prepareFooStub(‘fixtureId’, CustomResponseStub)
  )

  // This flushing method is called directly after the $httpBackend stubs are prepared
  // It doesn't have to be in it's own describe block, but I think it outlines a clear separation of concerns
  beforeEach(
    angular.mock.inject(function(_Foo_) {
      foo = _Foo_.get()

      $httpBackend.flush()
    })
  )

  // Prevents undefines from trickling through, but most importantly ensures that...
  afterEach(function() {
    $httpBackend.verifyNoOutstandingExpectation()
    $httpBackend.verifyNoOutstandingRequest()
  })

  // ... the request promise is always resolved by this point and is accessible as `foo`
  describe(‘someFunction’, function() {
    var actualFoo = resourceTest.restResponse(foo)

    expect(_.isFunction(actualFoo.someFunction)).toBe(true)
  })

  …
  ```

* Organize method assertion suites by the types of responses they can return in a specific resource state.  Generally, a hierarchy of "Resource -> State -> Model methods" outlines this structure.

  This structure helps limit the amount of state and context switching at or near the assertion level by 

  From a readability standpoint, an additional benefit is that it outlines only the relevant methods in a given resource state.

  ```javascript
  // Resource
  describe('Users', function() {
    // State
    describe('for users with recipes', function() {
      beforeEach(
        prepareRecipeStub(‘fixtureId’, UserWithRecipesStub)
      )

      // Model method
      desribe('getRecipes()', function() {
        it('should provide a list of recipes created by the user', function() {
          var actualUser    = resourceTest.restResopnse(user)
          var actualRecipes = actualUser.getRecipes()

          expect(_.isEmpty(actualRecipes)).toBe(false)
          expect(_.every(actualRecipes, {id: 'abc123'})).toBe(true)
        })
      })
    })

    // State
    describe('for users without recipes', function() {
      beforeEach(
        prepareRecipeStub(‘fixtureId’, UserWithoutRecipesStub)
      )

      // Model method
      describe('getRecipes()', function() {
        it('should provide an empty list of recipes', function() {
          var actualUser = resourceTest.restResopnse(user)
          var actualRecipes = actualUser.getRecipes()

          expect(_.isEmpty(actualRecipes)).toBe(true)
        })
      })
    })
  })
  ```

  Interestingly enough, the reverse structure (Model methods -> State -> Resource) provides the same benefit:

  ```javascript
  // Model method
  describe('getRecipes()', function() {

    // State
    describe('for users with recipes', function() {

      beforeEach(
        prepareRecipeStub(‘fixtureId’, UserWithRecipesStub)
      )

      // Model method
      it('should provide a list of recipes created by the user', function() {
        var actualUser    = resourceTest.restResopnse(user)
        var actualRecipes = actualUser.getRecipes()

        expect(_.isEmpty(actualRecipes)).toBe(false)
        expect(_.every(actualRecipes, {id: 'abc123'})).toBe(true)
      })

    })

    // State
    desribe('for users without recipes', function() {

      beforeEach(
        prepareRecipeStub(‘fixtureId’, UserWithoutRecipesStub)
      )

      // Model method
      it('should provide an empty list of recipes', function() {
        var actulUserWithRecipes = resourceTest.restResponse(user)

        var actualUser = resourceTest.restResopnse(user)
        var actualRecipes = actualUser.getRecipes()

        expect(_.isEmpty(actualRecipes)).toBe(true)
      })
    })

  })
  ```

Although these examples are simplified, we have found that, with the admitted occasional quirk, these concepts scale successfully in large applications. This approach has notable limitations when it comes to working with multiple stubs in the same assertion context, but we will attempt to address this issue the next article. We will also break down these design patterns even further to obtain a deep understanding of the combined Jasmine / AngularJS execution flow and how it relates to $http, $httpBackend, request flushing, and block scopes.

