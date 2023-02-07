# Tricky JS Questions

- [Tricky JS Questions](#tricky-js-questions)
  - [JavaScript Core](#javascript-core)
    - [Closures under the hood](#closures-under-the-hood)
    - [Prototype inheritance](#prototype-inheritance)
    - [`this` keyword](#this-keyword)
      - [Classic function declaration](#classic-function-declaration)
      - [arrow functions](#arrow-functions)
      - [Constructor-functions](#constructor-functions)
      - [Object method](#object-method)
      - [call, bind, apply usages](#call-bind-apply-usages)
    - [Event Loop](#event-loop)
      - [Sync code](#sync-code)
      - [Async code](#async-code)
        - [Micro-tasks and macro-tasks](#micro-tasks-and-macro-tasks)
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

### Event Loop

JavaScript is syncronous or one-threaded programming language. It means that it can execute one comand after another. Pretty simple concept, right? But what if we need to execute come code with delay? Here comes the Event Loop. In JavaScript runtime we have a call stack and a task queue, where our JS runtime put our function calls. Let's take a look how it works for a syncronous code

#### Sync code

For example, we have this lines of code:

```javascript
function hello(name) {
  console.log(`Hello, ${name}!`);
}

hello('Alex');
hello('John');
hello('Jane');
```

And our call stack looks like this:
`[hello('Alex'), hello('John'), hello('Jane')]`
The first call will be `hello('Jane')`, because stack is LIFO-structure and when it's called our stack will look like this:
`[hello('Alex'), hello('John')]`
Pretty simple concept, right? Now, let's take a look at asyncronous code.

#### Async code

And there goes task queue. Our runtime declares asyncronous code in call stack, but let's say, implementation of it put in task queue, so stack knows, that there is an instruction to execute, but we cant execute it right now, because it is in a queue.

We can define Event Loop, very primitive, like this:

```json
Event Loop
{
  callStack: [],
  taskQueue: []
}
```

Here's simple example:

```javascript
function sayHelloAndBye(name) {
  setTimeout(function hello() {
    console.log(`Hello, ${name}!`);
  }, 2000);

  console.log(`Bye, ${name}`);
}

sayHelloAndBye('Alex');
```

At the first step of our little program interpretetion, loop will look like this:

```json
Event Loop
{
  callStack: [sayHelloAndBye('Alex')],
  taskQueue: [],
}
```

Then, it will see, that we have `setTimeout` call and push it to queue:

```json
Event Loop
{
  callStack: [sayHelloAndBye('Alex')],
  taskQueue: [setTimeout(hello)],
}
```

His next step will be pushing our "bye" `console.log` to callStack:

```json
Event Loop
{
  callStack: [sayHelloAndBye('Alex'), console.log('Bye, Alex!')],
  taskQueue: [setTimeout(hello)],
}
```

Then, our "bye" log will be executed and popped-out of stack, but here's the thing - `sayHelloAndBye('Alex')` will be popped out as well, cause function body has no more instructions and there's no need to keep this function in stack:

```json
Event Loop
{
  callStack: [],
  taskQueue: [setTimeout(hello)],
}
```

When callStack will be empty, our taskQueue will be executed, but there's a nuance - we have synchronous code in setTimeout, so here's how it'll be look like(and yeah, we assumed, that 2 seconds have passed):

```json
Event Loop
{
  callStack: [hello(), console.log('Hello, Alex!')],
  taskQueue: [],
}
```

Then it runs `console.log`, pop-out `hello()` and our Event Loop will be empty again. Of course, it's a very simple explanation of such big mechanism as Event Loop, but it works just like that.

##### Micro-tasks and macro-tasks

But why it's working like that? Well, we have tasks and they are pushed into taskqueue, but how define task? Here's a short answer. Things like `setTimeout`, `setInterval`, `requestAnimationFrame`, etc. are APIs(Web API) that browser gives us. They are called macro-tasks.

And there are micro-tasks. They are a part of JavaScript and completely controlled from our code(`Promises` for example). Let's take a look at example. Here we have a couple synchronous commands, a Promise and a bunch of Web APIs usage. Try to order `console.log` in order of execution.

```javascript
console.log('log 1');
setTimeout(() => console.log('timeout 1'));
setTimeout(() => console.log('timeout 2'), 0);
Promise.resolve()
  .then(() => console.log('promise 1'))
  .then(() => console.log('promise 2'))
  .then(() => console.log('promise 3'));
console.log('log 2');
setTimeout(() => console.log('timeout 3'), 2000);
requestAnimationFrame(() => console.log('requestAnimationFrame'));
```

<details>
  <summary>Answer</summary>
  <ol>
    <li>log 1</li>
    <li>log 2</li>
    <li>promise 1</li>
    <li>promise 2</li>
    <li>promise 3</li>
    <li>timeout 1</li>
    <li>timeout 2</li>
    <li>requestAnimationFrame</li>
    <li>timeout3</li>
  </ol>
  <p>
    That's because Event Loop have strict order of execution. The first one is syncronous code, then goes async micro-tasks(Promises), and the last ones are macro-tasks(Web APIs).
  </p>
</details>

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
