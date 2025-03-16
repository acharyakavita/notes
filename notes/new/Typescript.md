# Typescript

## Type Inference

TypeScript knows the JavaScript language and will generate types for you in many cases. For example in creating a variable and assigning it to a particular value, TypeScript will use the value as its type.

```typescript
let x = 3;
```

## Basic Types

### Primitive Types

- `number`: All numbers, no differentiation between integers or floats
- `string`: All text values
- `boolean`: `true` or `false`
- `null`: Represents an intentional absence of an object value
- `undefined`: Denotes the absence of a value

### Arrays

You can define arrays using the `[]` syntax:

```typescript
let list: number[] = [1, 2, 3];
```

### Any

The `any` type is used to represent any JavaScript value:

```typescript
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false;
```

### Tuple

Tuples allow you to express an array with a fixed number of elements whose types are known:

```typescript
let x: [string, number];
x = ["hello", 10]; // OK
x = [10, "hello"]; // Error
```

### Enum

Enums allow you to define a set of named constants:

```typescript
enum Color {
  Red,
  Green,
  Blue,
}
let c: Color = Color.Green;
```

### Void

The `void` type is used when a function does not return a value:

```typescript
function warnUser(): void {
  console.log("This is my warning message");
}
```

### Functions

#### Parameters

You can define the types of parameters in a function:

```typescript
function greet(name: string) {
  console.log("Hello, " + name.toUpperCase() + "!!");
}
```

#### Optional Parameters

You can make parameters optional by adding a `?`:

```typescript
function greet(name?: string) {
  console.log("Hello, " + name?.toUpperCase() + "!!");
}
```

#### Rest Parameters

You can define a function that takes a variable number of arguments:

```typescript
function multiply(n: number, ...m: number[]) {
  return m.map((x) => n * x);
}
// 'a' gets value [10, 20, 30, 40]
const a = multiply(10, 1, 2, 3, 4);
```

#### Parameter Destructuring

The type annotation for the object goes after the destructuring syntax:

```typescript
function printProfile({ name, age }: { name: string; age: number }) {
  console.log(name, age);
}
```

#### Return Types

You can define the return type of a function:

```typescript
function greet(name: string): string {
  return "Hello, " + name.toUpperCase() + "!!";
}
```

#### Functions that return promises

You can define a function that returns a promise:

```typescript
function fetchUsers(): Promise<User[]> {
  return fetch("/users").then((res) => res.json());
}
```

Or using `async`/`await`:

```typescript
async function fetchUsers(): Promise<User[]> {
  const res = await fetch("/users");
  return res.json();
}
```

#### Anonymous Functions

When a function appears in a place where TypeScript can determine how it’s going to be called, the parameters of that function are automatically given types:

```typescript
const names = ["Alice", "Bob", "Eve"];

// Contextual typing for function - parameter s inferred to have type string
names.forEach(function (s) {
  console.log(s.toUpperCase());
});

// Contextual typing also applies to arrow functions
names.forEach((s) => {
  console.log(s.toUpperCase());
});
```

#### Overloads

You can define multiple function signatures for a function:

```typescript
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
```

#### Declaring `this` in a function

TypeScript will infer what the `this` should be in a function via code flow analysis. There are a lot of cases where you need more control over what object `this` represents and TypeScript lets you declare the type for this in the function body:

```typescript
function print(this: { x: number; y: number }) {
  console.log(this.x, this.y);
}
```

### Object Types

You can define the types of properties in an object:

```typescript
function printLabel(labeledObj: { label: string }) {
  console.log(labeledObj.label);
}

let myObj = { size: 10, label: "Size 10 Object" };
printLabel(myObj);
```

## Defining Types

### Type Aliases

Type aliases allow you to give a type a name:

```typescript
type Point = {
  x: number;
  y: number;
};

function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
```

### Interfaces

Interfaces are a way to define the shape of an object. They can be used to define the types of properties an object must have.

```typescript
interface Person {
  name: string;
  age: number;
}

let person: Person = {
  name: "John Doe",
  age: 30,
};
```

#### Types VS Interfaces

Type aliases and interfaces are very similar, but there are some subtle differences:

- Interfaces can be extended, but type aliases can't
- Interfaces can be implemented, but type aliases can't

##### Extending

Interface:

```typescript
interface Animal {
  name: string;
}

interface Bear extends Animal {
  honey: boolean;
}
```

Type Alias:

```typescript
type Animal = {
  name: string;
};

type Bear = Animal & { honey: boolean };
```

This creates an intersection type, which is a type that combines multiple types.

##### Implementing

Interface:

```typescript
interface Animal {
  name: string;
}

class Bear implements Animal {
  name: string;
}
```

#### Readonly Properties

You can make properties readonly by using the `readonly` keyword:

```typescript
interface Point {
  readonly x: number;
  readonly y: number;
}

let p1: Point = { x: 10, y: 20 };
p1.x = 5; // Error
```

Using the `readonly` modifier doesn’t necessarily imply that a value is totally immutable - or in other words, that its internal contents can’t be changed. It just means the property itself can’t be re-written to.

#### Index Signatures

You can define an index signature in an interface:

```typescript
interface StringArray {
  [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```

### Classes

Classes are a way to define a blueprint for an object. They can have properties and methods.

```typescript
interface User {
  name: string;
  id: number;
}

class UserAccount {
  name: string;
  id: number;

  constructor(name: string, id: number) {
    this.name = name;
    this.id = id;
  }
}

const user: User = new UserAccount("Murphy", 1);
```

## Composing Types

### Unions

You can combine types using the `|` operator.

```typescript
type MyBool = true | false;
```

A popular use-case for union types is to describe the set of `string` or `number` literals that a value is allowed to be:

```typescript
type WindowStates = "open" | "closed" | "minimized";
type LockStates = "locked" | "unlocked";
type PositiveOddNumbersUnderTen = 1 | 3 | 5 | 7 | 9;
```

Unions provide a way to handle different types too. For example, you may have a function that takes an `array` or a `string`:

```typescript
function getLength(obj: string | string[]) {
  return obj.length;
}
```

For example, you can make a function return different values depending on whether it is passed a string or an array:

```typescript
function wrapInArray(obj: string | string[]) {
  if (typeof obj === "string") {
    return [obj];
  }
  return obj;
}
```

### Generics

Generics allow you to define the type of a function or class at a later point in time.

```typescript
function identity<T>(arg: T): T {
  return arg;
}
```

A common example is an array:

```typescript
type StringArray = Array<string>;
type NumberArray = Array<number>;
type ObjectWithNameArray = Array<{ name: string }>;
```

You can declare your own types that use generics:

```typescript
interface Backpack<Type> {
  add: (obj: Type) => void;
  get: () => Type;
}

// This line is a shortcut to tell TypeScript there is a
// constant called `backpack`, and to not worry about where it came from.
declare const backpack: Backpack<string>;

// object is a string, because we declared it above as the variable part of Backpack.
const object = backpack.get();

// Since the backpack variable is a string, you can't pass a number to the add function.
backpack.add(23); // Error
```

#### Inferring Generic Types

Note that we didn’t have to specify Type in this sample. The type was inferred - chosen automatically - by TypeScript.

We can use multiple type parameters as well. For example, a standalone version of map would look like this:

```typescript
function map<Input, Output>(
  arr: Input[],
  func: (arg: Input) => Output
): Output[] {
  return arr.map(func);
}

// Parameter 'n' is of type 'number'
// 'parsed' is of type 'number[]'
const parsed = map(["1", "2", "3"], (n) => parseInt(n));
```

Note that in this example, TypeScript could infer both the type of the `Input` type parameter (from the given `string` array), as well as the `Output` type parameter based on the return value of the function expression `(number)`.

#### Generic Classes

You can also create classes with generic types:

```typescript
class Queue<T> {
  private data = [];

  push(item: T) {
    this.data.push(item);
  }

  pop(): T | undefined {
    return this.data.shift();
  }
}

const queue = new Queue<number>();
queue.push(0);
queue.push(1);
console.log(queue.pop().toFixed());
console.log(queue.pop().toFixed());
```

#### Generic Constraints

Sometimes we want to relate two values, but can only operate on a certain subset of values. In this case, we can use a constraint to limit the kinds of types that a type parameter can accept.

Let’s write a function that returns the longer of two values. To do this, we need a `length` property that’s a number. We constrain the type parameter to that type by writing an `extends` clause:

```typescript
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}

// longerArray is of type 'number[]'
const longerArray = longest([1, 2], [1, 2, 3]);
// longerString is of type 'alice' | 'bob'
const longerString = longest("alice", "bob");
// Error! Numbers don't have a 'length' property
const notOK = longest(10, 100);
// Error: Argument of type 'number' is not assignable to parameter of type '{ length: number; }'.
```

### Type Assertions

Sometimes you will have information about the type of a value that TypeScript can’t know about.

For example, if you’re using `document.getElementById`, TypeScript only knows that this will return some kind of `HTMLElement`, but you might know that your page will always have an `HTMLCanvasElement` with a given ID.

In this situation, you can use a type assertion to specify a more specific type:

```typescript
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
```

You can also use the angle-bracket syntax (except if the code is in a `.tsx` file), which is equivalent:

```typescript
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```

#### Non-null assertion operator

If you’re sure that a value isn’t `null` or `undefined`, you can use the non-null assertion operator `!`:

```typescript
function liveDangerously(x?: number | null) {
  // Error if x is null or undefined
  console.log(x!.toFixed());
}
```

## Narrowing

### Typeof type guards

You can use the `typeof` operator to narrow down the type of a variable:

```typescript
function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
    return Array(padding + 1).join(" ") + value;
  }
  if (typeof padding === "string") {
    return padding + value;
  }
  throw new Error(`Expected string or number, got '${padding}'.`);
}
```

### Truthiness type guards

You can use truthiness checks to narrow down the type of a variable:

```typescript
function getUsersOnlineMessage(numUsersOnline: number) {
  if (numUsersOnline) {
    return `There are ${numUsersOnline} online now!`;
  }
  return "Nobody's here. :(";
}
```

### Equality type guards

You can use equality checks to narrow down the type of a variable:

```typescript
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // We can now call any 'string' method on 'x' or 'y'.
    x.toUpperCase(); // (method) String.toUpperCase(): string
    y.toLowerCase(); // (method) String.toLowerCase(): string
  } else {
    console.log(x); // (parameter) x: string | number
    console.log(y); // (parameter) y: string | boolean
  }
}
```

When we checked that `x` and `y` are both equal in the above example, TypeScript knew their types also had to be equal. Since `string` is the only common type that both `x` and `y` could take on, TypeScript knows that x and y must be `string`s in the first branch.

Checking against specific literal values (as opposed to variables) works also:

```typescript
function printAll(strs: string | string[] | null) {
  if (strs !== null) {
    if (typeof strs === "object") {
      for (const s of strs) {
        // (parameter) strs: string[]
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs); // (parameter) strs: string
    }
  }
}
```

### `in` Operator Narrowing

You can use the `in` operator to narrow down the type of a variable:

```typescript
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    return animal.swim(); // (parameter) animal: Fish
  }
  return animal.fly(); // (parameter) animal: Bird
}
```

### `instanceof` Narrowing

You can use the `instanceof` operator to narrow down the type of a variable:

```typescript
function logValue(x: Date | string) {
  if (x instanceof Date) {
    console.log(x.toUTCString()); // (parameter) x: Date
  } else {
    console.log(x.toUpperCase()); // (parameter) x: string
  }
}
```

### Assignments

TypeScript looks at the right side of the assignment and narrows the left side appropriately:

```typescript
let x = Math.random() < 0.5 ? 10 : "hello world!"; // let x: string | number

x = 1;
console.log(x); // let x: number

x = "goodbye!";
console.log(x); // let x: string
```

### Control flow analysis

TypeScript can narrow types based on the control flow:

```typescript
function example() {
  let x: string | number | boolean;

  x = Math.random() < 0.5;

  if (Math.random() < 0.5) {
    x = "hello";
    console.log(x); // let x: string
  } else {
    console.log(x); // let x: number | boolean
  }

  console.log(x); // let x: string | number | boolean
}
```

### Type predicates

A type predicate is a function that returns a boolean and asserts the type of its argument:

```typescript
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

`pet is Fish` is our type predicate in this example. A predicate takes the form `parameterName is Type`, where `parameterName` must be the name of a parameter from the current function signature.

```typescript
// Both calls to 'swim' and 'fly' are now okay.
let pet = getSmallPet();

if (isFish(pet)) {
  pet.swim();
} else {
  pet.fly();
}
```

### Discriminated Unions

A common pattern in TypeScript is to have a type that has a string literal property — a discriminant — which is used to narrow types in a `switch` or `if` statement:

```typescript
interface Square {
  kind: "square";
  size: number;
}

interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}

interface Circle {
  kind: "circle";
  radius: number;
}

type Shape = Square | Rectangle | Circle;

function area(s: Shape) {
  switch (s.kind) {
    case "square":
      return s.size * s.size;
    case "rectangle":
      return s.width * s.height;
    case "circle":
      return Math.PI * s.radius ** 2;
  }
}
```

### Exhaustiveness checking and the `never` type

When narrowing, you can reduce the options of a union to a point where you have removed all possibilities and have nothing left. In those cases, TypeScript will use a `never` type to represent a state which shouldn’t exist.

The `never` type is assignable to every type; however, no type is assignable to `never` (except `never` itself). This means you can use narrowing and rely on `never` turning up to do exhaustive checking in a `switch` statement.

For example, adding a `default` to our `getArea` function which tries to assign the shape to `never` will not raise an error when every possible case has been handled.

```typescript
type Shape = Circle | Square;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

## KeyOf Type Operator

The `keyof` operator takes an object type and produces a string or numeric literal union of its keys:

```typescript
type Point = { x: number; y: number };
type P = keyof Point; // "x" | "y"
```

## Typeof Type Operator

The `typeof` operator takes a JavaScript value and produces a type representing the type of that value:

```typescript
const s = "hello";
type T = typeof s; // string
```

## Indexed Access Types

The indexed access operator `T[K]` is used to access the type of the property `K` on the type `T`:

```typescript
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"]; // number
```

The indexing type is itself a type, so we can use unions, keyof, or other types entirely:

```typescript
type I1 = Person["age" | "name"]; // string | number
type I2 = Person[keyof Person]; // string | number | boolean
type AliveOrName = "alive" | "name";
type I3 = Person[AliveOrName]; // string | boolean
```

## Conditional Types

Conditional types select one of two possible types based on a condition expressed as a type relationship test:

```typescript
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never;

interface Email {
  message: string;
}

type EmailMessageContents = MessageOf<Email>; // string

interface Dog {
  bark(): void;
}

type DogMessageContents = MessageOf<Dog>; // never
```

## Mapped Types

Mapped types transform properties in one type to another type:

```typescript
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};

type FeatureFlags = {
  darkMode: () => void;
};

type FeatureOptions = OptionsFlags<FeatureFlags>;
// type FeatureOptions = {
//   darkMode: boolean;
// }
```

## Mapping Modifiers

There are two additional modifiers which can be applied during mapping: `readonly` and `?` which affect mutability and optionality respectively.

You can remove or add these modifiers by prefixing with `-` or `+`. If you don’t add a prefix, then `+` is assumed.

```typescript
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};

type LockedAccount = {
  readonly id: string;
  readonly name: string;
};

type UnlockedAccount = CreateMutable<LockedAccount>;
// type UnlockedAccount = {
//   id: string;
//   name: string;
// }
```

## Key remapping via `as`

You can remap keys in mapped types using the `as` keyword:

```typescript
type MappedTypeWithNewProperties<Type> = {
  [Properties in keyof Type as NewKeyType]: Type[Properties];
};
```

You can leverage features like template literal types to create new property names from prior ones:

```typescript
type Getters<Type> = {
  [Property in keyof Type as `get${Capitalize<
    string & Property
  >}`]: () => Type[Property];
};

interface Person {
  name: string;
  age: number;
  location: string;
}

type LazyPerson = Getters<Person>;
// type LazyPerson = {
//   getName: () => string;
//   getAge: () => number;
//   getLocation: () => string;
// }
```

## Template Literal Types

Template literal types allow you to create new types by concatenating strings:

```typescript
type World = "world";
type Greeting = `hello ${World}`; // "hello world"
```

When a union is used in the interpolated position, the type is the set of every possible string literal that could be represented by each union member:

```typescript
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";

type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
// type AllLocaleIDs = "welcome_email_id" | "email_heading_id" | "footer_title_id" | "footer_sendoff_id"
```

## Modules

TBD
