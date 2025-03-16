# Javascript

## Primitive Types

`String`, `Boolean`, `Number`, `BigInt`, `Null`, `Undefined`, `Symbol`

## Object Reference Types

`Array`, `Function`

## Protoype

When it comes to inheritance, JavaScript only has one construct: objects. Each object has a private property which holds a link to another object called its prototype. That prototype object has a prototype of its own, and so on until an object is reached with null as its prototype. By definition, null has no prototype, and acts as the final link in this prototype chain. It is possible to mutate any member of the prototype chain or even swap out the prototype at runtime.

### Inheritance

When trying to access a property of an object, the property will not only be sought on the object but on the prototype of the object, the prototype of the prototype, and so on until either a property with a matching name is found or the end of the prototype chain is reached.

### Constructor

```javascript
// A constructor function
function Box(value) {
  this.value = value;
}

// Properties all boxes created from the Box() constructor
// will have
Box.prototype.getValue = function () {
  return this.value;
};

const boxes = [new Box(1), new Box(2), new Box(3)];
```

or as a `class`:

```javascript
class Box {
  constructor(value) {
    this.value = value;
  }

  // Methods are created on Box.prototype
  getValue() {
    return this.value;
  }
}
```

## Call/Apply/Bind and this

### this

Refers to the object where the function was called

### call

Calls the function with `this` bound and the provided arguments list

```javascript
fn.call(this, arg1, .., argN);
```

### apply

Calls the function with `this` bound and the provided arguments list in an array

```javascript
fn.apply(this, [arg1, .., argN]);
```

### bind

Returns a new function with `this` bound and the provided arguments list

```javascript
const newFn = fn.bind(this, arg1, .., argN);
```

## const

Defines a constant reference to a value, so a value type cannot be changed by a reference type can.

Good:

```javascript
const a = {};
a.foo = 2;
```

Bad:

```javascript
const a = 1;
a = 3; // throws!
```

## Functions

### Declaration

The function declaration creates a binding of a new function to a given name. These are hoisted to the top of the enclosing function or global scope.

```javascript
function calcRectArea(width, height) {
  return width * height;
}
```

### Expression

The function keyword can be used to define a function inside an expression. These are not hoisted.

```javascript
const getRectArea = function (width, height) {
  return width * height;
};
```

or as an Immediately Invoked Function Expression (IIFE):

```javascript
(function () {
  console.log("Code runs!");
})();
```

### Object

The `Function` object provides methods for functions

```javascript
return new Function("return x;");
```

## Promises

The Promise object represents the eventual completion (or failure) of an asynchronous operation and its resulting value.

```javascript
const myPromise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("foo");
  }, 300);
});

myPromise
  .then((value) => `${value} and bar`)
  .then((value) => `${value} and bar again`)
  .then((value) => `${value} and again`)
  .then((value) => `${value} and again`)
  .then((value) => {
    console.log(value);
  })
  .catch((err) => {
    console.error(err);
  });
```

### Concurrency

The Promise class offers four static methods to facilitate async task concurrency:

`Promise.all()`
Fulfills when all of the promises fulfill; rejects when any of the promises rejects.

`Promise.allSettled()`
Fulfills when all promises settle.

`Promise.any()`
Fulfills when any of the promises fulfills; rejects when all of the promises reject.

`Promise.race()`
Settles when any of the promises settles. In other words, fulfills when any of the promises fulfills; rejects when any of the promises rejects.

## Async/Await

### Async

The async function declaration creates a binding of a new async function to a given name. The await keyword is permitted within the function body

```javascript
async function foo() {
  return 1;
}
```

### Await

```javascript
async function foo() {
  await 1;
}
```

## Arrays

### Concat

The `concat()` method of Array instances is used to merge two or more arrays. This method does not change the existing arrays, but instead returns a new array.

```javascript
const array1 = ["a", "b", "c"];
const array2 = ["d", "e", "f"];
const array3 = array1.concat(array2);

console.log(array3); // Expected output: Array ["a", "b", "c", "d", "e", "f"]
```

### Fill

The `fill()` method of Array instances changes all elements within a range of indices in an array to a static value. It returns the modified array.

```javascript
const array1 = [1, 2, 3, 4];

// Fill with 0 from position 2 until position 4
console.log(array1.fill(0, 2, 4)); // Expected output: Array [1, 2, 0, 0]

// Fill with 5 from position 1
console.log(array1.fill(5, 1)); // Expected output: Array [1, 5, 5, 5]

console.log(array1.fill(6)); // Expected output: Array [6, 6, 6, 6]
```

### Filter

The `filter()` method of Array instances creates a shallow copy of a portion of a given array, filtered down to just the elements from the given array that pass the test implemented by the provided function.

```javascript
const words = ["spray", "elite", "exuberant", "destruction", "present"];

const result = words.filter((word) => word.length > 6);
console.log(result); // Expected output: Array ["exuberant", "destruction", "present"]
```

### Find

The `find()` method of Array instances returns the first element in the provided array that satisfies the provided testing function. If no values satisfy the testing function, undefined is returned.

```javascript
const array1 = [5, 12, 8, 130, 44];

const found = array1.find((element) => element > 10);
console.log(found); // Expected output: 12
```

### ForEach

The `forEach()` method of Array instances executes a provided function once for each array element.

```javascript
const array1 = ["a", "b", "c"];

array1.forEach((element) => console.log(element));
// Expected output: "a"
// Expected output: "b"
// Expected output: "c"
```

### Includes

The `includes()` method of Array instances determines whether an array includes a certain value among its entries, returning true or false.

```javascript
const array1 = [1, 2, 3];
console.log(array1.includes(2)); // Expected output: true

const pets = ["cat", "dog", "bat"];
console.log(pets.includes("cat")); // Expected output: true
console.log(pets.includes("at")); // Expected output: false
```

### Join

```javascript
const elements = ["Fire", "Air", "Water"];
elements.join(","); // returns Fire,Air,Water
```

### Map

The `map()` method of Array instances creates a new array populated with the results of calling a provided function on every element in the calling array.

```javascript
const array1 = [1, 4, 9, 16];

const map1 = array1.map((x) => x * 2);
console.log(map1); // Expected output: Array [2, 8, 18, 32]
```

### Pop

The `pop()` method of Array instances removes the last element from an array and returns that element. This method changes the length of the array.

```javascript
const plants = ["broccoli", "cauliflower", "cabbage", "kale", "tomato"];

console.log(plants.pop()); // Expected output: "tomato"
console.log(plants); // Expected output: Array ["broccoli", "cauliflower", "cabbage", "kale"]
```

### Push

The `push()` method of Array instances adds the specified elements to the end of an array and returns the new length of the array.

```javascript
const animals = ["pigs", "goats", "sheep"];

const count = animals.push("cows");
console.log(count); // Expected output: 4
console.log(animals); // Expected output: Array ["pigs", "goats", "sheep", "cows"]
```

### Reduce

The `reduce()` method of Array instances executes a user-supplied "reducer" callback function on each element of the array, in order, passing in the return value from the calculation on the preceding element. The final result of running the reducer across all elements of the array is a single value.

The first time that the callback is run there is no "return value of the previous calculation". If supplied, an initial value may be used in its place. Otherwise the array element at index 0 is used as the initial value and iteration starts from the next element (index 1 instead of index 0).

```javascript
const array1 = [1, 2, 3, 4];

// 0 + 1 + 2 + 3 + 4
const initialValue = 0;
const sumWithInitial = array1.reduce(
  (accumulator, currentValue) => accumulator + currentValue,
  initialValue
);

console.log(sumWithInitial); // Expected output: 10
```

### Reverse

The `reverse()` method of Array instances reverses an array in place and returns the reference to the same array, the first array element now becoming the last, and the last array element becoming the first. In other words, elements order in the array will be turned towards the direction opposite to that previously stated.

```javascript
const array1 = ["one", "two", "three"];
const reversed = array1.reverse();

console.log("reversed:", reversed); // Expected output: "reversed:" Array ["three", "two", "one"]
```

### Shift

The shift() method of Array instances removes the first element from an array and returns that removed element. This method changes the length of the array.

```javascript
const array1 = [1, 2, 3];
const firstElement = array1.shift();

console.log(array1); // Expected output: Array [2, 3]
console.log(firstElement); // Expected output: 1
```

### Slice

The `slice()` method of Array instances returns a shallow copy of a portion of an array into a new array object selected from `start` to `end` (end not included) where start and end represent the index of items in that array. The original array will not be modified.

```javascript
const animals = ["ant", "bison", "camel", "duck", "elephant"];

console.log(animals.slice(2)); // Expected output: Array ["camel", "duck", "elephant"]

console.log(animals.slice(2, 4)); // Expected output: Array ["camel", "duck"]

console.log(animals.slice(1, 5)); // Expected output: Array ["bison", "camel", "duck", "elephant"]

console.log(animals.slice(-2)); // Expected output: Array ["duck", "elephant"]

console.log(animals.slice(2, -1)); // Expected output: Array ["camel", "duck"]

console.log(animals.slice()); // Expected output: Array ["ant", "bison", "camel", "duck", "elephant"]
```

### Some

The `some()` method of Array instances tests whether at least one element in the array passes the test implemented by the provided function. It returns true if, in the array, it finds an element for which the provided function returns true; otherwise it returns false. It doesn't modify the array.

```javascript
const array = [1, 2, 3, 4, 5];

// Checks whether an element is even
const even = (element) => element % 2 === 0;

console.log(array.some(even)); // Expected output: true
```

### Sort

The `sort()` method of Array instances sorts the elements of an array in place and returns the reference to the same array, now sorted. The default sort order is ascending, built upon converting the elements into strings, then comparing their sequences of UTF-16 code units values.

It can take in a compare function with values (a, b) to compare, it should return a value where:

- A negative value indicates that `a` should come before `b`
- A positive value indicates that `a` should come after `b`
- Zero or `NaN` indicates that `a` and `b` are considered equal

```javascript
const months = ["March", "Jan", "Feb", "Dec"];
months.sort();
console.log(months); // Expected output: Array ["Dec", "Feb", "Jan", "March"]

const array1 = [1, 30, 4, 21, 100000];
array1.sort();
console.log(array1); // Expected output: Array [1, 100000, 21, 30, 4]
```

### Splice

The `splice()` method of Array instances changes the contents of an array by removing or replacing existing elements and/or adding new elements in place. Returns an array of deleted elements.

```javascript
const months = ["Jan", "March", "April", "June"];
months.splice(1, 0, "Feb"); // Inserts at index 1
console.log(months); // Expected output: Array ["Jan", "Feb", "March", "April", "June"]

months.splice(4, 1, "May"); // Replaces 1 element at index 4
console.log(months); // Expected output: Array ["Jan", "Feb", "March", "April", "May"]
```

#### Syntax

```javascript
splice(start, deleteCount, item1, item2, /* …, */ itemN);
```

### Unshift

The `unshift()` method of Array instances adds the specified elements to the beginning of an array and returns the new length of the array.

```javascript
const array1 = [1, 2, 3];

console.log(array1.unshift(4, 5)); // Expected output: 5
console.log(array1); // Expected output: Array [4, 5, 1, 2, 3]
```

## Fetch

The `Fetch API` provides a JavaScript interface for making HTTP requests and processing the responses.

Fetch is the modern replacement for `XMLHttpRequest`: unlike `XMLHttpRequest`, which uses callbacks, Fetch is promise-based and is integrated with features of the modern web such as service workers and Cross-Origin Resource Sharing (CORS).

With the Fetch API, you make a request by calling `fetch()`, which is available as a global function in both `window` and `worker` contexts. You pass it a Request object or a string containing the URL to fetch, along with an optional argument to configure the request.

The `fetch()` function returns a `Promise` which is fulfilled with a Response object representing the server's response. You can then check the request status and extract the body of the response in various formats, including text and JSON, by calling the appropriate method on the response.

```javascript
async function getData() {
  const url = "https://example.org/products.json";
  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error(`Response status: ${response.status}`);
    }

    const json = await response.json();
    console.log(json);
  } catch (error) {
    console.error(error.message);
  }
}
```

The `fetch()` function will reject the promise on some errors, but not if the server responds with an error status like `404`: so we also check the response status and throw if it is not OK.

Otherwise, we fetch the response body content as `JSON` by calling the `json()` method of Response, and log one of its values. Note that like `fetch()` itself, `json()` is asynchronous, as are all the other methods to access the response body content.

### Options

#### Method

By default, `fetch()` makes a `GET` request, but you can use the method option to use a different request method.

```javascript
const response = await fetch("https://example.org/post", {
  method: "POST",
  // ...
});
```

#### Body

The request body is the payload of the request: it's the thing the client is sending to the server. You cannot include a body with `GET` requests, but it's useful for requests that send content to the server, such as `POST` or `PUT` requests. For example, if you want to upload a file to the server, you might make a `POST` request and include the file as the request body.

```javascript
const response = await fetch("https://example.org/post", {
  body: JSON.stringify({ username: "example" }),
  // ...
});
```

#### Headers

Request headers give the server information about the request: for example, the `Content-Type` header tells the server the format of the request's body. Many headers are set automatically by the browser and can't be set by a script: these are called `Forbidden header names`.

```javascript
const response = await fetch("https://example.org/post", {
  headers: {
    "Content-Type": "application/json",
  },
  // .,.
});
```

Or:

```javascript
const myHeaders = new Headers();
myHeaders.append("Content-Type", "application/json");

const response = await fetch("https://example.org/post", {
  headers: myHeaders,
  // .,.
});
```

## Looping

### For statement

```javascript
function countSelected(selectObject) {
  let numberSelected = 0;
  for (let i = 0; i < selectObject.options.length; i++) {
    if (selectObject.options[i].selected) {
      numberSelected++;
    }
  }
  return numberSelected;
}
```

### Do...while statement

```javascript
let i = 0;

do {
  i += 1;
  console.log(i);
} while (i < 5);
```

### While statement

```javascript
let n = 0;
let x = 0;

while (n < 3) {
  n++;
  x += n;
}
```

### Labeled statement

```javascript
label: statement;
```

### Break statement

```javascript
break;
break label;
```

Or:

```javascript
for (let i = 0; i < a.length; i++) {
  if (a[i] === theValue) {
    break;
  }
}
```

### Continue statement

```javascript
let i = 0;
let n = 0;
while (i < 5) {
  i++;
  if (i === 3) {
    continue;
  }
  n += i;
  console.log(n);
}
// Logs:
// 1 3 7 12
```

### For...in statement

```javascript
function dumpProps(obj, objName) {
  let result = "";

  for (const i in obj) {
    result += `${objName}.${i} = ${obj[i]}<br>`;
  }

  result += "<hr>";
  return result;
}
```

### For...of statement

```javascript
const arr = [3, 5, 7];

for (const i of arr) {
  console.log(i);
}
// Logs: 3 5 7
```

## Switch

The `switch` statement evaluates an expression, matching the expression's value against a series of `case` clauses, and executes statements after the first `case` clause with a matching value, until a `break` statement is encountered. The `default` clause of a `switch` statement will be jumped to if no case matches the expression's value.

```javascript
const expr = "Papayas";

switch (expr) {
  case "Oranges":
    console.log("Oranges are $0.59 a pound.");
    break;
  case "Mangoes":
  case "Papayas":
    console.log("Mangoes and papayas are $2.79 a pound.");
    // Expected output: "Mangoes and papayas are $2.79 a pound."
    break;
  default:
    console.log(`Sorry, we are out of ${expr}.`);
}
```

## Map Object

The `Map` object holds key-value pairs and remembers the original insertion order of the keys. Any value (both objects and primitive values) may be used as either a key or a value.

```javascript
const myMap = new Map();

const keyString = "a string";
const keyObj = {};
const keyFunc = function () {};

// setting the values
myMap.set(keyString, "value associated with 'a string'");
myMap.set(keyObj, "value associated with keyObj");
myMap.set(keyFunc, "value associated with keyFunc");

myMap.size; // 3

// getting the values
myMap.get(keyString); // "value associated with 'a string'"
myMap.get(keyObj); // "value associated with keyObj"
myMap.get(keyFunc); // "value associated with keyFunc"
```

## Set Object

The `Set` object lets you store unique values of any type, whether primitive values or object references.

```javascript
const mySet = new Set();

mySet.add(1);
mySet.add(5);
mySet.add(5); // 5 is already in the set
mySet.add("some text");
const o = { a: 1, b: 2 };
mySet.add(o);

mySet.has(1); // true
mySet.has(3); // false, 3 has not been added to the set
mySet.has(5); // true
mySet.has(Math.sqrt(25)); // true
mySet.has("Some Text".toLowerCase()); // true
mySet.has(o); // true

mySet.size; // 4

mySet.delete(5); // removes 5 from the set
mySet.has(5); // false, 5 has been removed

mySet.size; // 3, we just removed one value
```
