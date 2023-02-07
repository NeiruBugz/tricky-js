# Tricky JS Questions

- [Tricky JS Questions](#tricky-js-questions)
  - [JavaScript Core](#javascript-core)
    - [Event loop](#event-loop)
      - [Sync code](#sync-code)
      - [Async code](#async-code)
        - [Micro-tasks](#micro-tasks)
        - [Macro-tasks](#macro-tasks)
    - [Closures under the hood](#closures-under-the-hood)
    - [Prototype inheritance](#prototype-inheritance)
    - [`this` keyword](#this-keyword)
      - [Classic function declaration](#classic-function-declaration)
      - [arrow functions](#arrow-functions)
      - [Constructor-functions](#constructor-functions)
      - [Object method](#object-method)
      - [call, bind, apply usages](#call-bind-apply-usages)
    - [Promise API](#promise-api)
      - [States](#states)
      - [Methods](#methods)
  - [React](#react)
    - [React patterns](#react-patterns)
    - [Cons of memoizaion in React apps](#cons-of-memoizaion-in-react-apps)
  - [TypeScript](#typescript)
    - [TypeScript disadvantages](#typescript-disadvantages)
    - [TypeScript Utility types](#typescript-utility-types)

## JavaScript Core

### Event loop

#### Sync code

#### Async code

##### Micro-tasks

##### Macro-tasks

### Closures under the hood

### Prototype inheritance

### `this` keyword

 `this` - pointer to some object. Pretty tricky thing cause its value depends on invocation context. Contexts are:

- classic function declaration
- constructor-functions
- object method
- arrow functions
- call, bind and apply usages

#### Classic function declaration

 It's pretty simple. `this` equals global object: window in browser and global in Node environment. Also, if function body annotated with `'use strict'` – `this` equals `undefined`. And it cascades into inner functions, even if inner function is an arrow one

 ```javascript
 function f() {
  console.log(this);
 }

 f(); // window or global

 function s() {
  'use strict'
  console.log(this);
 }

 s(); // undefined
 ```

#### arrow functions

Arrow function hasn't its own context, instead they take closest `this` value from outer hirerachy.

```javascript
const f = () => {
  console.log(this);
};

f(); // logs Window in browser

const user = { user: '123' };

f.call(user);

f(); // still Window
```

#### Constructor-functions

 `this` equals instance of a class/new object. In-depth, when constructor called `this` is an empty object, but when we assign some values to new object instance `this` will have them too, cause it is an instance of this object/class

 ```javascript
 function A(name) {
  console.log(this);
  this.name = name;
  console.log(this);
 }

 const a = new A('John');
 // first log: Object {}
 // second log: Object { name: 'John' }
 ```

#### Object method

In objects that contains some method the value of `this` is an object itself. But, there's a trick. If we, let's say set an object method to a variable, then `this` will be same as in classic function declaration.

#### call, bind, apply usages

- `call`, `apply`

  `call` and `apply` require `this` as first argument to call some function with some scope. The only difference between them is how they take other arguments: `call` takes arguments comma-separated, `apply` takes them as an array

- `bind`
  As it comes from the name, `bind` binds `this` value and some method or function. It doesn't executes function, but creates a new one with some context

### Promise API

#### States

- `Pending` - at operation start
- `Fulfilled` - promise resolved and we can process result of its execution
- `Rejected` - error while executing a promise

#### Methods

- `.resolve()` - fired when promise is fulfilled and processes resolve value next
- `.reject()` - fired when promise is rejected and throws an error
- `.then()` - works, when promise is not pending and we have to do something with execution result, inherit from prototype
- `.catch()` - catches error when promise got rejected
- `.finally()` - works every time, when promise fulfilled/rejected, inherit from prototype
- `.all()` - handles array of promises, but when catches rejected promise interrupts and results an error, returns a Promise
- `.allSettled()` - handles array of promises and returns a Promise results array, even when one of promises is rejected
- `.any()` - handles array of promises and returns first fulfilled promise or returns an error, when every promise got rejected
- `.race()` - handles array of promises and returns first executed promise, either its fulfilled or rejected

## React

### React patterns

### Cons of memoizaion in React apps

## TypeScript

### TypeScript disadvantages

### TypeScript Utility types
