# Closures

- Closure is the most esoteric of JavaScript concepts
- Enables powerful pro-level functions like ‘once’ and ‘memoize’
- Many JavaScript design patterns including the module pattern use closure
- Build iterators, handle partial application and maintain state in an asynchronous world

## Functions with memories
- When our functions get called, we create a live store of data (local memory/variable environment/state) for that function’s execution context.
- When the function finishes executing, its local memory is deleted (except the returned value)
- But what if our functions could hold on to live data between executions?
- This would let our function definitions have an associated cache/persistent memory
- But it all starts with us returning a function from another function

### Functions can be returned from other functions in JavaScript
```javascript
function createFunction() {
  function multiplyBy2(num) {
    return num * 2;
  }
  return multiplyBy2;
}
const generatedFunc = createFunction();
const result = generatedFunc(3); // 6
```

### Calling a function in the same function call as it was defined (where you define your functions determines what data it has access to when you call it)
```javascript
function outer() {
  let counter = 0;
  function incrementCounter() {
    counter++;
  }
  incrementCounter();
}
outer();
```

### Calling a function outside of the function call in which it was defined
```javascript
function outer (){
 let counter = 0;
 function incrementCounter (){ counter ++; }
 return incrementCounter;
}
const myNewFunction = outer();
myNewFunction();
myNewFunction();
```

## The bond
When a function is defined, it gets a bond to the surrounding Local Memory (“Variable Environment”) in which it has been defined

## The "backpack"

- We return incrementCounter’s code (function definition) out of outer into global and give it a new name - myNewFunction
- We maintain the bond to outer’s live local memory - it gets ‘returned out’attached on the back of incrementCounter’s function definition.
- So outer’s local memory is now stored attached to myNewFunction - even though outer’s execution context is long gone
- When we run myNewFunction in global, it will first look in its own local memory first (as we’d expect), but then in myNewFunction’s ‘backpack’

## What can we call this "backpack"?

- Closed over ‘Variable Environment’ (C.O.V.E.)
- Persistent Lexical Scope Referenced Data (P.L.S.R.D.)
- ‘Backpack’
- ‘Closure’

The ‘backpack’ (or ‘closure’) of live data is attached incrementCounter (then to myNewFunction) through a hidden property known as [[scope]] which persists
when the inner function is returned out

## Individual backpacks

If we run 'outer' again and store the returned 'incrementCounter' function definition in 'anotherFunction', this new incrementCounter function was created in
a new execution context and therefore has a brand new independent backpack
```javascript
function outer() {
  let counter = 0;
  function incrementCounter() {
    counter++;
  }
  return incrementCounter;
}

const myNewFunction = outer();
myNewFunction();
myNewFunction();

const anotherFunction = outer();
anotherFunction();
anotherFunction();
```

## Closure gives our functions persistent memories and entirely new toolkit for writing professional code

- Helper functions: Everyday professional helper functions like ‘once’ and ‘memoize’
- Iterators and generators: Which use lexical scoping and closure to achieve the most contemporary patterns for handling data in JavaScript
- Module pattern: Preserve state for the life of an application without polluting the global namespace
- Asynchronous JavaScript: Callbacks and Promises rely on closure to persist state in an asynchronous environment


# Promises, Async & the Event Loop

- Promises - the most signficant ES6 feature
- Asynchronicity - the feature that makes dynamic web applications possible
- The event loop - JavaScript’s triage
- Microtask queue, Callback queue and Web Browser features (APIs)

## JavaScript is not enough - We need new pieces (some of which aren’t JavaScript at all)

Our core JavaScript engine has 3 main parts:
- Thread of execution
- Memory/variable environment
- Call stack

We need to add some new components:
- Web Browser APIs/Node background APIs
- Promises
- Event loop, Callback/Task queue and micro task queue


## ES6+ Solution (Promises)

Using two-pronged ‘facade’ functions that both:
- Initiate background web browser work and
- Return a placeholder object (promise) immediately in JavaScript

Special objects built into JavaScript that get returned immediately when we make a call to a web browser API/feature (e.g. fetch) that’s set up to return promises (not all are)
Promises act as a placeholder for the data we expect to get back from the web browser feature’s background work

### then method and functionality to call on completion

Any code we want to run on the returned data must also be saved on the promise object
Added using .then method to the hidden property ‘onFulfilment’
Promise objects will automatically trigger the attached function to run (with its input being the returned data)

### But we need to know how our promise-deferred functionality gets back into JavaScript to be run
```javascript
function display(data) {
  console.log(data);
}
function printHello() {
  console.log('Hello');
}
function blockFor300ms() {}

setTimeout(printHello, 0);

const futureData = fetch('https://twitter.com/will/tweets/1');
futureData.then(display);

blockFor300ms();

console.log('Me first!');

// me first
// twitter data
// Hello
```

### We have rules for the execution of our asynchronously delayed code

Hold promise-deferred functions in a microtask queue and callback function in a task queue (Callback queue) when the Web Browser Feature (API) finishes
Add the function to the Call stack (i.e. run the function) when:
    - Call stack is empty & all global code run (Have the Event Loop check this condition)
Prioritize functions in the microtask queue over the Callback queue

### Promises, Web APIs, the Callback & Microtask Queues and Event loop enable:

- Non-blocking applications: This means we don’t have to wait in the single thread and don’t block further code from running
- However long it takes: We cannot predict when our Browser feature’s work will finish so we let JS handle automatically running the function on its completion
- Web applications: Asynchronous JavaScript is the backbone of the modern web - letting us build fast ‘non-blocking’ applications



### Summary of Call Stack - Microtask Queue - Callback Queue, all managed by Event Loop

- Call Stack (global()): order / where functions run
- Event Loop: constantly checks if Call stack is empty, then checks the Microtask Queue, then the Callback Queue 
- Microtask Queue: any function that is attached to a Promise object by the .then method and then auto-triggered by JS when the value property in that promise object gets updated
automatically as a result of the background work (e.g fetch), that function goes to the Microtask Queue
- Callback Queue: any function that is thrown out of JS (e.g to the browser, example: setTimeout) those functions when the background work is complete, they come back to the Callback Queue


# Classes, Prototypes - Object Oriented JavaScript

- An enormously popular paradigm for structuring our complex code
- Prototype chain - the feature behind-the-scenes that enables emulation of OOP but is a compelling tool in itself
- Understanding the difference between __proto__ and prototype
- The new and class keywords as tools to automate our object & method creation

## That is, I want my code to be:

1. Easy to reason about
2. Easy to add features to (new functionality)
3. Nevertheless efficient and performant

The Object-oriented paradigm aims is to let us achieve these three goals


## So if I’m storing each user in my app with their respective data (let’s simplify)

user1:
- name: ‘Tim’
- score: 3

user2:
- name: ‘Stephanie’
- score: 5

And the functionality I need to have for each user (again simplifying!)
- increment functionality (there’d be a ton of functions in practice)
How could I store my data and functionality together in one place?



### Solution 1. Generate objects using a function
```javascript
function userCreator(name, score) {
  const newUser = {};
  newUser.name = name;
  newUser.score = score;
  newUser.increment = function () {
    newUser.score++;
  };
  return newUser;
}
const user1 = userCreator('Will', 3);
const user2 = userCreator('Tim', 5);
user1.increment();
```

Problems: Each time we create a new user we make space in our computer's memory for all our data and functions. But our functions are just copies
Is there a better way?


### Solution 2: Using the prototype chain

Store the increment function in just one object and have the interpreter, if it doesn't find the function on user1, look up to that object to check if it's there
Link user1 and functionStore so the interpreter, on not finding .increment, makes sure to check up in functionStore where it would find it
Make the link with Object.create() technique
```javascript
function userCreator(name, score) {
  const newUser = Object.create(userFunctionStore);
  newUser.name = name;
  newUser.score = score;
  return newUser;
}
const userFunctionStore = {
  increment: function () {
    this.score++;
  },
  login: function () {
    console.log('Logged in');
  },
};
const user1 = userCreator('Will', 3);
const user2 = userCreator('Tim', 5);
user1.increment();
```

What if we want to confirm our user1 has the property score?
We can use the hasOwnProperty method - but where is it? Is it on user1?

All objects have a __proto__ property by default which defaults to linking to a big object - Object.prototype full of (somewhat) useful functions
We get access to it via userFunctionStore’s __proto__ property - the chain
```javascript
function userCreator(name, score) {
  const newUser = Object.create(userFunctionStore);
  newUser.name = name;
  newUser.score = score;
  return newUser;
}
const userFunctionStore = {
  increment: function () {
    this.score++;
  },
  login: function () {
    console.log('Logged in');
  },
};
const user1 = userCreator('Will', 3);
const user2 = userCreator('Tim', 5);
user1.hasOwnProperty('score');
```

### Arrow functions override the normal this rules
```javascript
function userCreator(name, score) {
  const newUser = Object.create(userFunctionStore);
  newUser.name = name;
  newUser.score = score;
  return newUser;
}
const userFunctionStore = {
  increment: function () {
    const add1 = () => {
      this.score++;
    };
    add1();
  },
};
const user1 = userCreator('Will', 3);
const user2 = userCreator('Tim', 5);
user1.increment();
```

Now our inner function gets its this set by where it was saved - it’s a ‘lexically scoped this


### Solution 3 - Introducing the keyword that automates the hard work: new

When we call the function that returns an object with new in front we automate 2 things
1. Create a new user object
2. Return the new user object

But now we need to adjust how we write the body of userCreator - how can we:
- Refer to the auto-created object?
- Know where to put our single copies of functions?

```javascript
function userCreator(name, score){
  this.name = name;
  this.score = score;
 }
 userCreator.prototype.increment = function(){ this.score++; };
 userCreator.prototype.login = function(){ console.log("login"); };
 
 const user1 = new userCreator(“Eva”, 9)
 user1.increment()
```

 ### Solution 4: The class ‘syntactic sugar’
 
 We’re writing our shared methods separately from our object ‘constructor’ itself (off in the userCreator.prototype object)
```javascript
 class UserCreator {
  constructor(name, score) {
    this.name = name;
    this.score = score;
  }
  increment() {
    this.score++;
  }
  login() {
    console.log('login');
  }
}
const user1 = new UserCreator('Eva', 9);
user1.increment();
```