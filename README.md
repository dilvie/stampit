# Stampit

Create objects from reusable, composable behaviors.

## Features

 * Create functions (called factories) which stamp out new objects. All of the new objects inherit all of the prescribed behavior.

 * Compose factories together to create new factories.

 * Inherit methods and default state.

 * Supports composable private state and privileged methods.

 * State is cloned for each instance, so it won't be accidentally shared.

 * For the curious - it's great for [learning about prototypal OO](http://ericleads.com/2013/02/fluent-javascript-three-different-kinds-of-prototypal-oo/). It mixes three major types of prototypes:
   1. differential inheritance, aka delegation (for methods),
   2. cloning, aka concatenation/exemplar prototypes (for state),
   3. functional / closure inheritance (for privacy / encapsulation)

## What's the Point?

Prototypal OO is great, and JavaScript's capabilities give us some really powerful tools to explore it, but it could be easier to use.

Basic questions like "how do I inherit privileged methods and private data?" and "what are some good alternatives to inheritance hierarchies?" are stumpers for many JavaScript users.

Let's answer both of these questions at the same time. First, we'll use a closure to create data privacy:

```js
var a = stampit().enclose(function () {
  var a = 'a';
  this.getA = function () {
    return a;
  };
});
```

It uses function scope to encapsulate private data. Note that the getter must be defined inside the function in order to access the closure variables.

Let's see if that worked:

```js
a(); // Object -- so far so good.
a().getA(); // "a"
```

Yes. Got it. In both of these instances, we actually created a brand new object, and then immediately threw it away, because we didn't assign it to anything. Don't worry about that.

Here's another:

```js
var b = stampit().enclose(function () {
  var a = 'b';
  this.getB = function () {
    return a;
  };
});
```

Those `a`'s are not a typo. The point is to demonstrate that `a` and `b`'s private variables won't clash.

But here's the real treat:

```js
var c = stampit.compose(a, b);

var foo = c(); // we won't throw this one away...

foo.getA(); // "a"
foo.getB(); // "b"
```

WAT? Yeah. You just inherited privileged methods and private data from two sources at the same time.

But that's boring. Let's see what else is on tap:

```js
// Some more privileged methods, with some private data.
// Use stampit.mixin() to make this feel declarative:
var availability = stampit().enclose(function () {
  var isOpen = false; // private

  return stampit.mixin(this, {
    open: function open() {
      isOpen = true;
      return this;
    },
    close: function close() {
      isOpen = false;
      return this;
    },
    isOpen: function isOpenMethod() {
      return isOpen;
    }
  });
});

// Here's a mixin with public methods, and some state:
var membership = stampit({
    add: function (member) {
      this.members[member.name] = member;
      return this;
    },
    getMember: function (name) {
      return this.members[name];
    }
  },
  {
    members: {}
  });

// Let's set some defaults:
var defaults = stampit().state({
      name: 'The Saloon',
      specials: 'Whisky, Gin, Tequila'
    });

// Classical inheritance has nothing on this. No parent/child coupling. No deep inheritance hierarchies.
// Just good, clean code reusability.
var bar = stampit.compose(defaults, availability, membership);

// Note that you can override state on instantiation:
var myBar = bar({name: 'Moe\'s'});

// Silly, but proves that everything is as it should be.
myBar.add({name: 'Homer' }).open().getMember('Homer');
```

## More chaining

You can chain `.methods()` ...

```js
var obj = stampit().methods({
  foo: function () {
    return 'foo';
  },
  methodOverride: function () {
    return false;
  }
}).methods({
  bar: function () {
    return 'bar'
  },
  methodOverride: function () {
    return true;
  }
}).create();
```

And `.state()` ...

```js
var obj = stampit().state({
  foo: {bar: 'bar'},
  stateOverride: false
}).state({
  bar: 'bar',
  stateOverride: true
}).create();
```

## Pass multiple objects into .methods() and .state()

Stampit mimics the behavior of `_.extend()`, `$.extend()`, and ES6 `Object.mixIn()` when you pass multiple objects into `.methods()`. In other words, it will copy all of the properties from those objects to the `.methods` or `.state` prototype for the factory. The properties from later arguments in the list will override the same named properties of previously passed in objects.

```js
  var obj = stampit().methods({
    a: function () { return 'a'; }
  }, {
    b: function () { return 'b'; }
  }).create();
```

Or `.state()` ...

```js
  var obj = stampit().state({
    a: 'a'
  }, {
    b: 'b'
  }).create();
```

# Stampit API #

**Source: stampit.js**

### stampit ###

Return a factory function that will produce new objects using the
prototypes that are passed in or composed.

* `@param {Object} [methods]` A map of method names and bodies for delegation.
* `@param {Object} [state]` A map of property names and values to clone for each new object.
* `@param {Function} [enclose]` A closure (function) used to create private data and privileged methods.
* `@return {Function} factory` A factory to produce objects using the given prototypes.
* `@return {Function} factory.create` Just like calling the factory function.
* `@return {Object} factory.fixed` An object map containing the fixed prototypes.
* `@return {Function} factory.methods` Add methods to the methods prototype. Chainable.
* `@return {Function} factory.state` Add properties to the state prototype. Chainable.
* `@return {Function} factory.enclose` Add or replace the closure prototype. Not chainable.


### compose ###

Take two or more factories produced from stampit() and
combine them to produce a new factory. Combining overrides
properties with last-in priority.

* `@param {...Function} factory` A factory produced by stampit().
* `@return {Function}` A new stampit factory composed from arguments.


### mixIn ###

Take a destination object followed by one or more source objects,
and copy the source object properties to the destination object,
with last in priority overrides.

* `@param {Object} destination` An object to copy properties to.
* `@param {...Object} source` An object to copy properties from.
* `@returns {Object}`

### extend ###

Alias for `mixIn`.
