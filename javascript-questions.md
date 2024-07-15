# JavaScript Interview questions

## 1. What is the difference between named and arrow functions?

### Named Functions
Named functions are traditional functions that have their own name and can be called anywhere in their scope. They are defined using the `function` keyword.

```javascript
function namedFunction() {
  console.log('This is a named function');
}
namedFunction(); // This is a named function
```

### Arrow Functions
Arrow functions are a shorter syntax for writing functions in ES6. They do not have their own `this` context and are defined using the `=>` syntax.

```javascript
const arrowFunction = () => {
  console.log('This is an arrow function');
}
arrowFunction(); // This is an arrow function
```

**Key Differences:**
- **Syntax**: Arrow functions have a more concise syntax.
- **`this` Context**: Arrow functions do not have their own `this` context; they inherit it from the surrounding scope.
- **Usage**: Arrow functions are often used for simple functions or callbacks, while named functions are preferred for more complex operations.

## 2. What are generators?

Generators are functions that can be paused and resumed. They are defined using the `function*` syntax and use the `yield` keyword to pause execution.

```javascript
function* generatorFunction() {
  yield 1;
  yield 2;
  yield 3;
}

const generator = generatorFunction();
console.log(generator.next().value); // 1
console.log(generator.next().value); // 2
console.log(generator.next().value); // 3
```

**Key Features:**
- **Pause and Resume**: Generators can pause execution at any point using `yield`.
- **Iterators**: They implement the iterator protocol and can be used in loops.
- **State Management**: Useful for managing asynchronous operations or handling state.

## 3. What is the difference between function declaration and function expression?

### Function Declaration
A function declaration defines a named function and is hoisted, meaning it can be called before it is defined in the code.

```javascript
function declaredFunction() {
  console.log('This is a function declaration');
}
declaredFunction(); // This is a function declaration
```

### Function Expression
A function expression defines a function inside an expression and is not hoisted, meaning it cannot be called before it is defined.

```javascript
const expressedFunction = function() {
  console.log('This is a function expression');
};
expressedFunction(); // This is a function expression
```

**Key Differences:**
- **Hoisting**: Function declarations are hoisted, while function expressions are not.
- **Usage**: Declarations are used for defining named functions, while expressions are often used for anonymous functions or passing functions as arguments.

## 4. What is scope chaining?

Scope chaining refers to the process by which JavaScript resolves variable names. When a variable is referenced, JavaScript starts looking for it in the current scope. If it's not found, it moves up to the next outer scope, and so on, until it reaches the global scope.

```javascript
function outerFunction() {
  let outerVar = 'outer';
  function innerFunction() {
    let innerVar = 'inner';
    console.log(outerVar); // outer
  }
  innerFunction();
}
outerFunction();
```

**Key Points:**
- **Nested Scopes**: Functions can access variables from their own scope and outer scopes.
- **Global Scope**: If a variable is not found in any scope, it leads to a `ReferenceError`.

## 5. What is `this` object in functions?

The `this` object refers to the context in which a function is called. Its value depends on how the function is invoked.

```javascript
const obj = {
  value: 42,
  getValue: function() {
    console.log(this.value);
  }
};
obj.getValue(); // 42
```

**Key Points:**
- **Method Call**: When called as a method, `this` refers to the object.
- **Global Context**: In the global context or in a regular function, `this` refers to the global object (or `undefined` in strict mode).
- **Arrow Functions**: Arrow functions do not have their own `this`; they inherit it from the parent scope.

## 6. How to shallow copy from one object to another object?

Shallow copying creates a new object with the same properties as the original, but nested objects are still referenced.

```javascript
const original = { a: 1, b: { c: 2 } };
const shallowCopy = { ...original };
console.log(shallowCopy); // { a: 1, b: { c: 2 } }
```

## 7. How to deep copy from one object to another object?

Deep copying creates a new object with all properties deeply copied, so nested objects are also duplicated.

```javascript
const original = { a: 1, b: { c: 2 } };
const deepCopy = JSON.parse(JSON.stringify(original));
console.log(deepCopy); // { a: 1, b: { c: 2 } }
```

For more complex objects (functions, dates, etc.), consider using libraries like Lodash.

## 8. What is execution context?

Execution context is the environment in which JavaScript code is executed. It consists of three main components:
- **Variable Object**: Contains function arguments, inner variable declarations, and function declarations.
- **Scope Chain**: Contains the current scope and its outer scopes.
- **`this` Keyword**: References the object associated with the execution context.

There are two types of execution contexts:
- **Global Execution Context**: Default context; creates the global object and sets up the global scope.
- **Function Execution Context**: Created whenever a function is invoked.

## 9. What is event loop?

The event loop is a mechanism that allows JavaScript to perform non-blocking operations by offloading operations to the system kernel whenever possible.

**Key Points:**
- **Call Stack**: Where functions are executed.
- **Task Queue**: Where asynchronous tasks (like setTimeout) are queued.
- **Event Loop**: Continuously checks the call stack and task queue. If the call stack is empty, it pushes the first task from the task queue to the call stack.

## 10. What is the difference between map and foreach?

Both `map` and `forEach` are used to iterate over arrays, but they have different purposes and behaviors.

### `map`
- **Returns a New Array**: Transforms each element and returns a new array with the results.
- **Chainable**: Since it returns a new array, you can chain other array methods.

```javascript
const numbers = [1, 2, 3];
const doubled = numbers.map(n => n * 2);
console.log(doubled); // [2, 4, 6]
```

### `forEach`
- **No Return Value**: Executes a provided function for each array element but does not return a new array.
- **Not Chainable**: Since it returns `undefined`, it cannot be chained with other array methods.

```javascript
const numbers = [1, 2, 3];
numbers.forEach(n => console.log(n * 2)); // 2, 4, 6
```

## 11. What is a Promise?

A Promise is an object representing the eventual completion or failure of an asynchronous operation.

**Key States:**
- **Pending**: Initial state, neither fulfilled nor rejected.
- **Fulfilled**: Operation completed successfully.
- **Rejected**: Operation failed.

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => resolve('Success!'), 1000);
});
promise.then(result => console.log(result)); // Success!
```

## 12. What is lexical scoping?

Lexical scoping refers to the scope of a variable being determined by its position in the source code. Inner functions have access to variables defined in their outer scopes.

```javascript
function outerFunction() {
  let outerVar = 'outer';
  function innerFunction() {
    console.log(outerVar); // outer
  }
  innerFunction();
}
outerFunction();
```

**Key Points:**
- **Static Scope**: The scope is determined at compile time.
- **Scope Chain**: Inner functions can access variables from their parent functions.

These concepts are fundamental in JavaScript and are often explored in technical interviews for understanding a candidate's depth of knowledge and problem-solving abilities.

Here are detailed explanations for each of the additional interview preparation questions:

## 13. What are string literals?

String literals are sequences of characters enclosed in quotes. JavaScript supports three types of quotes for string literals:
- **Single quotes** (`'...'`)
- **Double quotes** (`"..."`)
- **Template literals** (`` `...` ``)

**Examples:**
```javascript
// Single quotes
const singleQuoteString = 'Hello, world!';

// Double quotes
const doubleQuoteString = "Hello, world!";

// Template literals (introduced in ES6)
const name = 'Alice';
const templateLiteralString = `Hello, ${name}!`; // Hello, Alice!
```

**Key Features of Template Literals:**
- **Multi-line Strings**: Allow strings to span multiple lines without concatenation.
- **String Interpolation**: Embed expressions within the string using `${}`.

## 14. What is the difference between ES5 and ES6?

ECMAScript 5 (ES5) and ECMAScript 6 (ES6) are versions of the ECMAScript standard, which defines JavaScript.

**Key Differences:**

### ES5
- **Introduced in 2009**.
- **New Features**: `strict mode`, `JSON` object, `Array` methods (`forEach`, `map`, `filter`, etc.), `Function.prototype.bind`, `Object.create`.

### ES6 (also known as ECMAScript 2015)
- **Introduced in 2015**.
- **New Features**:
  - **let and const**: Block-scoped variables.
  - **Arrow Functions**: Concise syntax and lexical `this`.
  - **Template Literals**: Enhanced string interpolation.
  - **Classes**: Syntactical sugar over prototype-based inheritance.
  - **Modules**: `import` and `export` statements.
  - **Default Parameters**: Function parameters with default values.
  - **Rest and Spread Operators**: `...` for variable arguments and array/object spread.
  - **Destructuring**: Extracting values from arrays or properties from objects into variables.
  - **Promises**: Improved handling of asynchronous operations.
  - **Map and Set**: New collection types.

## 15. What is the difference between document and window?

### `window`
- **Global Object**: Represents the browser window or frame containing the DOM document.
- **Global Scope**: All global JavaScript objects, functions, and variables automatically become members of the `window` object.
- **Methods and Properties**: Includes methods like `alert()`, `setTimeout()`, `setInterval()`, and properties like `innerHeight`, `innerWidth`.

### `document`
- **Document Object**: Represents the HTML or XML document loaded in the window.
- **DOM**: Provides access to the content of the document, including elements, attributes, and text.
- **Methods and Properties**: Includes methods like `getElementById()`, `querySelector()`, `createElement()`, and properties like `title`, `body`.

**Example:**
```javascript
// Accessing the window object
console.log(window.innerHeight);

// Accessing the document object
console.log(document.title);
```

## 16. How do you calculate Fibonacci numbers in JavaScript?

**Recursive Approach:**
```javascript
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

console.log(fibonacci(10)); // 55
```

**Iterative Approach:**
```javascript
function fibonacci(n) {
  let a = 0, b = 1, temp;
  for (let i = 1; i < n; i++) {
    temp = a;
    a = b;
    b = temp + b;
  }
  return b;
}

console.log(fibonacci(10)); // 55
```

## 17. How to add and remove object properties dynamically in JavaScript?

**Adding Properties:**
```javascript
const obj = {};
obj.newProperty = 'value'; // Using dot notation
obj['anotherProperty'] = 'another value'; // Using bracket notation
```

**Removing Properties:**
```javascript
delete obj.newProperty;
delete obj['anotherProperty'];
```

## 18. Explain the difference between `Object.freeze()` and `const`.

### `Object.freeze()`
- **Immutable Object**: Prevents adding, removing, or modifying properties of an object.
- **Shallow Freeze**: Only the top-level properties are frozen, not nested objects.

```javascript
const obj = { prop: 'value' };
Object.freeze(obj);
obj.prop = 'new value'; // No effect
console.log(obj.prop); // 'value'
```

### `const`
- **Immutable Reference**: Prevents reassigning the variable to a new value, but does not make the object itself immutable.

```javascript
const obj = { prop: 'value' };
obj.prop = 'new value'; // Allowed
console.log(obj.prop); // 'new value'

const obj = {}; // Not allowed, throws an error
```

## 19. Explain JavaScript cookies.

Cookies are small pieces of data stored by the browser, associated with a specific website. They are used to remember information between requests.

**Setting a Cookie:**
```javascript
document.cookie = "username=JohnDoe; expires=Fri, 31 Dec 2024 12:00:00 UTC; path=/";
```

**Getting Cookies:**
```javascript
console.log(document.cookie); // username=JohnDoe
```

**Deleting a Cookie:**
```javascript
document.cookie = "username=JohnDoe; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/";
```

## 20. Explain the role of `deferred` scripts in JavaScript.

The `defer` attribute is used with the `<script>` tag to indicate that the script should be executed after the document has been parsed but before the `DOMContentLoaded` event.

**Example:**
```html
<script src="script.js" defer></script>
```

**Benefits:**
- **Non-blocking**: Does not block the parsing of HTML.
- **Execution Order**: Deferred scripts are executed in the order they are encountered in the document.

## 21. Why wrap the content of JS source file in a function block?

Wrapping the content of a JS file in an Immediately Invoked Function Expression (IIFE) helps avoid polluting the global namespace and creates a local scope for variables and functions.

**Example:**
```javascript
(function() {
  var localVariable = 'This is local';
  console.log(localVariable); // This is local
})();
console.log(localVariable); // ReferenceError: localVariable is not defined
```

## 22. What are browser events?

Browser events are actions or occurrences that happen in the browser, which JavaScript can respond to. Examples include user interactions, browser lifecycle events, and DOM events.

**Common Events:**
- **Mouse Events**: `click`, `dblclick`, `mouseover`, `mouseout`, `mousedown`, `mouseup`
- **Keyboard Events**: `keydown`, `keyup`, `keypress`
- **Form Events**: `submit`, `change`, `focus`, `blur`
- **Document Events**: `DOMContentLoaded`, `load`, `unload`, `resize`

**Event Handling Example:**
```javascript
document.getElementById('myButton').addEventListener('click', function() {
  alert('Button clicked!');
});
```

## 23. What is vanilla JavaScript?

Vanilla JavaScript refers to plain JavaScript, without any libraries or frameworks. It emphasizes using the core language features and built-in APIs to accomplish tasks.

**Example:**
```javascript
document.getElementById('myElement').innerText = 'Hello, world!';
```

Using vanilla JavaScript ensures compatibility and avoids dependencies on third-party libraries, making the code more lightweight and maintainable.