# __:simple-javascript: JavaScript__

## Variables

| Kind | Scope | Redeclare | Reassign | Hoisted | Binds `this` |
| --- | --- | --- | --- | --- | --- |
| `var` | No | Yes | Yes | Yes | Yes |
| `let` | Yes | No | Yes | No | No |
| `const` | Yes | No | No | No | No |

- __Hoisting__: Move the declarations of functions, variables, and classes to the top of their respective scopes before the code is executed. ==i.e.: You can use the variable before it is declared==
- `this`: Refers to the context in which a function is executed. ==It's value is not determined by where the function is defined, but rather by how the function is called.==

=== "`var`"

    ``` javascript title="var"
    // Re-Declaring
    var carName = "Volvo";
    var carName;

    // Global scope
    { 
      var x = 2; 
    }
    // x CAN be used here
    ```

=== "`let`"

    ``` javascript title="let"
    { 
      let x = 2;
    }
    // x can NOT be used here
    ```

=== "`const`"

    - Must be assigned a value when they are declared
    - Use `const` when you declare:
        - A new Array
        - A new Object
        - A new Function
        - A new RegExp
    - ==It does not define a constant value. It defines a constant reference to a value.==
        
        __CAN NOT__:

        - Reassign a constant value
        - Reassign a constant array
        - Reassign a constant object

        __CAN__:

        - Change the elements of constant array
        - Change the properties of constant object

    ``` javascript title="const"
    const PI = 3.141592653589793;
    PI = 3.14;      // This will give an error

    const PI;
    PI = 3.14159265359; // This will also give an error

    var x = 2;     // Allowed
    const x = 2;   // Not allowed

    // You can create a constant array:
    const cars = ["Saab", "Volvo", "BMW"];

    // You can change an element:
    cars[0] = "Toyota";

    // You can add an element:
    cars.push("Audi");

    // You can create a const object:
    const car = {type:"Fiat", model:"500", color:"white"};

    // You can change a property:
    car.color = "red";

    // You can add a property:
    car.owner = "Johnson";

    typeof 3.14           // Returns "number"
    typeof (3 + 4)        // Returns "number"
    ```
 

## Data Types

- JavaScript has 8 Datatypes:
    - String
    - Number: All JavaScript numbers are stored in a 64-bit floating-point format.
    - Bigint: new datatype (ES2020) that can be used to store integer values bigger than regular Number
    - Boolean
    - Undefined: A variable without a value, has the value and type as undefined.
    - Null: Intentional absence of any object value.
    - Symbol
    - Object: 2-types
        - Built-in: objects, arrays, dates, maps, sets, intarrays, floatarrays, promises, and more.
        - User-defined
- ==JavaScript has dynamic types. i.e. same variable can be used to hold different data types.==
- `typeof` operator returns the type of a variable or an expression.

``` javascript title="types"
let x;       // Now x is undefined
x = 5;       // Now x is a Number
x = "John";  // Now x is a String

// Numbers:
let length = 16;
let weight = 7.5;
let y = 123e5;    // 12300000
let z = 123e-5;   // 0.00123

// Strings:
let color = "Yellow";

// Booleans
let x = `true`;
let y = `false`;

// Object:
const person = {firstName:"John", lastName:"Doe"};

// Array object:
const cars = ["Saab", "Volvo", "BMW"];

// Date object:
const date = new Date("2022-03-25");

// Type casting
let x = 16 + 4 + "Volvo"; // 20Volvo
let x = "Volvo" + 16 + 4; // Volvo164
```


## Data Structures

``` javascript
// Array can hold a collection of items
let array = [];
array.push(1);
array.push(1);

// Set is a collection without duplicates
let noDuplicate = new Set();
noDuplicate.add(1);
noDuplicate.add(1);

// Map is a collection of key value pairs
let map = new Map();
map.set('first', 1);
map.set('second', 2);
```

## Class

``` javascript
class Person {
    firstName = 'John';
    lastName = 'Doe';
    #ssn;           // private field
    static constantField = "static value";

    constructor(firstName, lastName, ssn) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.#ssn = ssn;
    }

    getName() {
        return `Person: ${this.firstName} ${this.lastName}`
    }
}

class Employee extends Person {
    constructor(firstName, lastName, ssn) {
        super(firstName, lastName, ssn);         // super() must be invoked before the use of this
    }

    getName() {
        return `Employee: ${this.firstName} ${this.lastName}`
    }
}

console.log(`Static field value ${Employee.constantField}`);
let person = new Person('John', 'Adams');
console.log(person.getName());
let employee = new Employee('John', 'Adams');
console.log(employee.getName());
```


## Comparisons

### Comparison Operators

| Operator | Description | Comparing	| Return |
|---|---|---|---|
| `==` | equal value	| `5 == 8` | `false` |
| | | `5 == 5` | `true` |
| | | `5 == "5"` | `true` |
| `===` | equal value ==and== equal type | `5 === 5` | `true` |
| | | `5 === "5"`	| `false` |
| `!=` | not equal | `5 != 8` | `true` |
| `!==` | not equal value ==or== not equal type | `5 !== 5` | `false` |
| | | `5 !== "5"` | `true` |
| | | `5 !== 8` | `true` |
| `>` | greater than | `5 > 8`	| `false` |
| `<`	| less than | `5 < 8` | `true` |
| `>=` | greater than or equal to	| `5 >= 8` | `false` |
| `<=` | less than or equal to | `5 <= 8` | `true` |

### Logical Operators

| Operator | Description | Example |
| --- | --- | --- |
| `&&` | and | `(6 < 10 && 3 > 1)` is `true` |	
| `||` | or	| `(6 == 5 || 3 == 5)` is `false` |
| `!` | not	| `!(6 == 3)` is `true` |

### Other Operators

- The `??` operator returns the first argument if it is not nullish (null or undefined).

``` javascript
// Ternary Operator
let voteable = (age < 18) ? "Too young":"Old enough";

let name = null;
let text = "missing";
let result = name ?? text;      // result == missing
```

## Conditions, Loops

``` javascript title="conditions"
if (time < 10) {
  greeting = "Good morning";
} else if (time < 20) {
  greeting = "Good day";
} else {
  greeting = "Good evening";
}

switch (new Date().getDay()) {
  case 4:
  case 5:       // case 4 and 5 executes same block
    text = "Soon it is Weekend";
    break;      // it breaks out of the switch block.
  case 0:
  case 6:
    text = "It is Weekend";
    break;
  default:
    text = "Looking forward to the Weekend";
}
```

``` javascript title="loops"
// For Loop
for (let i = 0; i < 5; i++) {
  text += "The number is " + i + "<br>";
}

// For-In
const person = {fname:"John", lname:"Doe", age:25};
// or
// const person = ["Alex", "Luke", "Hailey"]
for (let x in person) {
  console.log(person[x]);
}

const numbers = [45, 4, 9, 16, 25];
numbers.forEach(myFunction);

function myFunction(value, index, array) {
  console.log(value);
}

// For-Of
// Variable can be const, let, or var.
// It lets you loop over iterable data structures such as Arrays, Strings, Maps, NodeLists, and more.
const cars = ["BMW", "Volvo", "Mini"];
for (let x of cars) {
  text += x;
}

// While
while (i < 10) {
  text += "The number is " + i;
  i++;
}

// Do-While
do {
  text += "The number is " + i;
  i++;
}
while (i < 10);

// Break and continue
const nums = [1,2,5,3];
for (let i = 0; i < len(nums); i++) {
  if (i === 2) { continue; }
  if(nums[i]==5) break;
}

// Scopes
var i = 5;
for (var i = 0; i < 10; i++) {
  // some code
}
// Here i is 10

let i = 5;
for (let i = 0; i < 10; i++) {
  // some code
}
// Here i is 5
```

## Functions

``` javascript title="functions"
//Using variable
const addVariable = function(x, y) {
  const sum = x + y;
  return sum;
};
addVariable(1, 2);      // 3

//Named functions
function addNamed(x, y) {
  return x + y;
}
addNamed(5, 5);         // 10

//Anonymous Functions
const sum = (function(x, y) {
  return x + y;
})(1, 2);
sum;        // 3

function addAndExcute(x, y, callback) {
  const sum = x + y;
  if (typeof callback === 'function') {
    return callback(sum);
  }
}

function multiply(value) {
  return value * 10;
};

function divide(value) {
  return value / 10;
};

addAndExcute(2, 3, multiply);       // 50
addAndExcute(2, 3, divide);         // 0

// Arrow functions
const addArrow = (x, y) => {
  return x + y;
};
```


## Pass by value or reference

- In passed by value, a copy of the actual value is passed to the function. primitive data types are passed by value.
-  In passed by reference, a reference to the actual data is passed.
- Javascript is always pass by value. Changing the value of a variable never changes the underlying primitive or object, it just points the variable to a new primitive or object.

``` javascript
// Pass by value
let n = 10;

function modify(x) {
    x = 20;
    console.log(x);     // 20
}

modify(n);
console.log(n);         // 10

// Pass by reference
let obj = { name: "Ravi" };

function modify(o) {
    o.name = "Arun";
    console.log(o.name);    // Arun
}

modify(obj);
console.log(obj.name);      // Ravi
```


## Asynchronous operations

### Timeout and Interval

- `setTimeout(callback, delay)`:
    - Runs the callback once, after delay milliseconds.
    - Runs only once.
    - Can be cleared with clearTimeout(timerId).
- `setInterval(callback, delay)`:
    - Runs the callback repeatedly, every delay milliseconds.
    - Runs repeatedly.
    - Can be cleared with clearTimeout(timerId).

```javascript
let i = 0;
let intervalId = setInterval(() => {
  console.log(i);
  i++;
}, 1000);

// let interval run for 10 seconds
setTimeout(() => {
  clearInterval(intervalId);
  console.log("Interval cleared");
}, 10000);

console.log("Program completed");
```

### Promise

- Promises were introduced to solve asynchronous programming challenges in a cleaner, more manageable way.
- ==A common need is to execute two or more asynchronous operations back to back, where each subsequent operation starts when the previous operation succeeds, with the result from the previous step. Resulting in callback hell.==

=== "Callback Hell"

    ``` javascript title="Callback hell"
    // defining async operations
    function getUser(id, callback, err) {
        try {
            let user = fetch(`http://example.com/users/${id}`);    // asynchronous
            callback(user);
        } catch(e) {
            err();
        }
    }

    function getPosts(userId, callback, err) {
        try {
            let posts = fetch(`http://example.com/users/${id}/posts`);    // asynchronous
            callback(posts);
        } catch(e) {
            err();
        }
    }

    function getComments(postId, callback, err) {
        try {
            let comments = fetch(`http://example.com/users/${id}/posts/${postId}/comments`);    // asynchronous
            callback(comments);
        } catch(e) {
            err();
        }
    }

    // invoking asynch operations in desired sequence
    getUser(id, (user) => {
        getPosts(user.id, (posts) => {
            getComments(posts[0].id, (comments) => {
                // ... deeply nested
            });
        });
    });
    ```

=== "Solution"

    With this pattern, you can create longer chains of processing, where each promise represents the completion of one asynchronous step in the chain.

    ``` javascript
    function getUser(id) {
        return new Promise((resolve, reject) => {
            try {
                let user = fetch(`http://example.com/users/${id}`);    // asynchronous
                resolve(user);  // (1)!
            } catch(e) {
                reject(e);
            }
        });
    }

    function getPosts(userId) {
        return new Promise((resolve, reject) => {
            try {
                let posts = fetch(`http://example.com/users/${id}/posts`);    // asynchronous
                resolve(posts);
            } catch(e) {
                err();
            }
        });
    }

    function getComments(posts) {
        return new Promise((resolve, reject) => {
            try {
                let comments = fetch(`http://example.com/users/${id}/posts/${posts[0]}/comments`);    // asynchronous
                resolve(comments);
            } catch(e) {
                err();
            }
        });
    }

    // (2)!
    getUser(1)
        .then((user) => getPosts(user))
        .then((posts) => getComments())
        .then((comments) => doSomething())
        .catch(e => {
            console.error("Something went wrong:", err);
        });
    ```

    1. resolve() is equivalent to pressing the DONE button on a task.

    2. When you write promise.then(callback). Javascript stores the callback in a queue. And, when resolve(...params) is called, the promise immediately calls all .then(callback). This ensures sequence of async operations are maintained without nested callbacks.


### async/await

- it's similar to promise, except the promise appears to return an output mimicing a synchronous call.
- `await` only works in a `async` function.

```javascript
let promise = new Promise((resolve, reject) => {
    let i = 6;
    setTimeout(() => {
        if (resolve && i < 5) {
            resolve(i);
        } else {
            reject("Reject");
        }
    }, 1000);
});
console.log("Invoking promise object");

let asynFn = async () => {
  try {
    const result = await promise;   // returns input passed to resolve func in the promise
    console.log(`Result: ${result}`);
  } catch (error) {
    console.log(`Error: ${error}`);
  }
};
asynFn();

```

## `this`

- `this` keyword refers to an object.
- It refers to different objects depending on how it is used:
    - In an object method, `this` refers to the object.
    - Alone, `this` refers to the global object.
    - In a function, `this` refers to default binding (alone: global object, object method: object).
    - In a function, in `strict` mode, `this` is `undefined`.
    - In an event, `this` refers to the element that received the event.
    - Arrow functions don't bind their own scope, but inherit it from the parent one.
- Methods like `call()`, `apply()`, and `bind()` can bind `this` to any object.

```javascript
let x = this;               // global object

"use strict";
let x = this;               // global object

function myFunction() {
  return this;              // the default binding. i.e. global object   
}

"use strict";
function myFunction() {
  return this;              // strict mode does not allow default binding. this == undefined
}

const person = {
    firstName  : "John",
    lastName   : "Doe",
    id         : 5566,
    myFunction : function() {
        return this;
    },
    arrowFunction: () => {
        return this;        // inherits this from person. i.e. this = global
    }
};

let self = this;
person.myFunction()         // returns person
person.arrowFunction();     // returns self;

class Employee {
    constructor() {}
    myFunction() {
        return this;        // this == instance(Employee)
    }
}
```

Eplicit binding

=== "`call()`"

    Immediately invokes the function with a given `this` and comma-separated arguments.

    ```javascript
    function greet(greeting, punctuation) {
        console.log(`${greeting}, ${this.name}${punctuation}`);
    }
    const person = { name: "Ron" };

    greet.call(person, "Hello", "!");     // Hello, Ron!
    ```

=== "`apply()`"

    Just like call(), but arguments are passed as an array instead of separately.

    ```javascript
    function greet(greeting, punctuation) {
        console.log(`${greeting}, ${this.name}${punctuation}`);
    }
    const person = { name: "Ron" };

    greet.apply(person, ["Hello", "!"]);     // Hello, Ron!
    ```

=== "`bind()`"

    Does not call the function right away — it returns a new function with the `this` value pre-set.

    ```javascript
    function greet(greeting, punctuation) {
        console.log(`${greeting}, ${this.name}${punctuation}`);
    }
    const person = { name: "Ron" };

    const greetCyril = greet.bind(person);
    greetCyril("Hey", "!");                 // Hello, Ron!
    ```


## Prototype

- A prototype is just a blueprint object from which other objects can inherit properties and methods.
- Every JavaScript object has an internal link to a prototype — which is another object.

```javascript
let numbers = [1, 2, 3, 4];

numbers.forEach((i, val) => { console.log(`${i} -> ${val}`)});      // val -> idx

Array.prototype.forEach = function (callback) {
    for(let i=0; i<this.length; i++) {
        callback(i, this[i]);
    }
}
numbers.forEach((i, val) => { console.log(`${i} -> ${val}`)});      // idx -> val
```


## Modules

- Each file can have only one default export.

``` javascript title="person.js"
// utils.js
export const name = 'Ron';              // named export

export function greet(name) {           // named export
  return `Hello, ${name}`;
}

const val = 99;
export default val;                     // default export
```

``` javascript title="index.js"
import defultExport, { name, greet } from './person.js';
```