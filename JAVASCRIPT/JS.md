# mr-frontend-interview

Useful JavaScript interview questions that cover various aspects of the language, ranging from basic to advanced topics

1. What is JavaScript? <br>
   JavaScript is a high-level, interpreted programming language primarily used for web development to add interactivity and dynamic elements to websites. It runs on the client-side (in the browser) and server-side (using Node.js).

2. What are the data types in JavaScript? <br>
   JavaScript has six primitive data types: string, number, boolean, null, undefined, and symbol (introduced in ES6). Additionally, it has one complex data type: object.

3. Explain the difference between null and undefined.<br>
   In JavaScript, "null" represents the intentional absence of a value, while "undefined" means a variable has been declared but not assigned a value. Both are used to indicate the absence of meaningful data, but they have different origins and usages.

4. How do you declare a variable in JavaScript?<br>
   You can declare a variable using the "var," "let," or "const" keyword, followed by the variable name. For example: `var x;`, `let y;`, `const z = 10;`.

5. What are the different types of scopes in JavaScript?<br>
   JavaScript has two main scopes: global scope and function scope. In ES6, the "let" and "const" keywords introduced block-scoped variables, adding block scope to the language.

6. What is the purpose of the "use strict" directive in JavaScript?<br>
   The "use strict" directive enables strict mode in JavaScript, which enforces a stricter set of rules for code quality and error handling. It helps avoid common programming pitfalls and makes code easier to optimize.

7. What is the difference between let, const, and var in variable declaration?<br>
   "let" and "const" are block-scoped, allowing the variable to be limited to the block they are declared in, while "var" is function-scoped. Additionally, "const" is used for constants whose value cannot be changed after declaration.

8. Explain the concept of hoisting in JavaScript.
   Hoisting is a JavaScript behavior where variable and function declarations are moved to the top of their containing scope during the compilation phase. However, only the declarations are hoisted, not the assignments.

9. What are closures in JavaScript?<br>
   Closures are functions that have access to variables from their containing (enclosing) scope even after the scope has finished executing. They "remember" the environment in which they were created, allowing data to be preserved and accessed later.

10. How do you handle asynchronous operations in JavaScript?<br>
    JavaScript provides several methods to handle asynchronous operations, such as callbacks, promises, and async/await. Callbacks were traditionally used, but promises and async/await are more modern and provide better readability and error handling.

11. What are callbacks in JavaScript?<br>
    Callbacks are functions passed as arguments to other functions and are executed later asynchronously. They are commonly used to handle asynchronous operations or events, ensuring that certain code executes after specific tasks are completed.

12. How do you create and use a promise in JavaScript?<br>
    Promises are used to handle asynchronous operations more cleanly than callbacks. You can create a promise using the `Promise` constructor, which takes two arguments: `resolve` for successful execution and `reject` for error handling. To use a promise, you can chain `.then()` for resolved cases and `.catch()` for rejected cases.

13. What is the purpose of the fetch API in JavaScript?<br>
    The fetch API is used to make network requests and retrieve resources from the server. It provides a more modern and flexible alternative to the older XMLHttpRequest object, simplifying AJAX requests and handling responses using promises.

14. Explain event bubbling and event capturing in JavaScript.
    Event bubbling and event capturing are two phases of event propagation in the DOM. During event bubbling, an event is first handled on the target element and then propagated upwards through its ancestors. In contrast, during event capturing, the event is captured from the outermost ancestor down to the target element.

15. How do you handle errors in JavaScript?<br>
    Errors in JavaScript can be handled using try-catch blocks. Code that might throw an exception is placed within the try block, and if an exception occurs, it is caught and handled in the catch block. This prevents the program from crashing and allows you to gracefully handle errors.

16. What are higher-order functions in JavaScript?<br>
    Higher-order functions are functions that take one or more functions as arguments or return a function. They enable functional programming paradigms and allow for more concise and reusable code.

17. How do you iterate over an object's properties in JavaScript?<br>
    You can iterate over an object's properties using a for...in loop or the Object.keys(), Object.values(), or Object.entries() methods introduced in ES6.

18. What is the difference between call, apply, and bind methods in JavaScript?<br>
    All three methods are used to set the value of "this" in a function. The `call` method invokes the function with a specified context and additional arguments, while the `apply` method does the same but takes arguments as an array. The `bind` method returns a new function with the specified context, but it doesn't immediately call the function.

19. How do you check if a variable is an array in JavaScript?<br>
    You can use the Array.isArray() method to check if a variable is an array. It returns true if the variable is an array and false otherwise.

20. Explain the concept of prototypal inheritance in JavaScript.<br>
    Prototypal inheritance is a mechanism in JavaScript where objects can inherit properties and methods from another object called the prototype. Each object has an internal prototype link that allows it to access properties and methods defined in its prototype. When a property or method is not found in the object itself, JavaScript looks up the prototype chain until it finds the property or reaches the end of the chain (Object.prototype).

21. What is a closure and how is it used?<br>
    A closure is a function that retains access to its lexical scope even after the outer function has finished executing. It "closes over" the variables in its scope, allowing them to be accessed later. Closures are commonly used for data encapsulation and creating private variables in JavaScript.

22. How do you handle exceptions in JavaScript?<br>
    Exceptions in JavaScript can be handled using try-catch blocks. Code that might throw an exception is placed within the try block, and if an exception occurs, it is caught and handled in the catch block. This prevents the program from crashing and allows you to gracefully handle errors.

23. What are arrow functions in JavaScript?<br>
    Arrow functions are a more concise syntax for writing functions in JavaScript. They have a shorter syntax and automatically inherit the "this" value from the surrounding code. Arrow functions are often used in callback functions and as shortcuts for simple functions.

24. Explain the concept of promises in JavaScript and how they differ from callbacks.
    Promises are objects that represent the eventual completion (or failure) of an asynchronous operation and allow you to handle its result asynchronously using the `.then()` and `.catch()` methods. They provide a more structured way of dealing with asynchronous code compared to traditional callbacks, improving readability and error handling.

25. How do you clone an object in JavaScript?<br>
    You can clone an object in JavaScript using various methods, such as the `Object.assign()` method or the spread operator (`...`). For deep cloning (including nested objects and arrays), you can use libraries like `lodash` or `JSON.parse(JSON.stringify(obj))`, but be aware of potential limitations when cloning complex objects.

26. What are immediately-invoked function expressions (IIFE) in JavaScript?<br>
    An Immediately-Invoked Function Expression (IIFE) is a function that is defined and executed immediately after it is created. It is wrapped in parentheses to ensure that it is treated as an expression rather than a function declaration. IIFEs are commonly used to create a private scope for variables, avoiding variable collisions with other code.

27. How do you add an event listener in JavaScript?<br>
    You can add an event listener to an element using the `.addEventListener()` method. It takes the event type (e.g., "click", "keydown") as the first argument and a function (callback) as the second argument, which will be executed when the event is triggered.

28. What is the purpose of the "this" keyword in JavaScript?<br>
    The "this" keyword in JavaScript refers to the current execution context or the object to which a method belongs. Its value depends on how a function is called or where it is invoked. In a regular function, "this" refers to the global object (window in browsers), but in object methods, "this" points to the object itself.

29. How do you prevent event propagation in JavaScript?<br>
    To prevent event propagation (event bubbling) in JavaScript, you can use the `.stopPropagation()` method on the event object inside the event handler. This will stop the event from propagating to ancestor elements, ensuring it does not trigger other event listeners.

30. Explain the differences between ES5 and ES6 (ECMAScript 2015).<br>
    ES5 (ECMAScript 5) and ES6 (ECMAScript 2015) are different versions of the ECMAScript standard. ES6 introduced several new features and improvements, such as let and const for variable declarations, arrow functions, classes, destructuring assignment, and more. ES6 also improved support for promises and introduced new data structures like Set and Map.

31. What is the purpose of the Map and Set objects in JavaScript<br>
    The Map object in JavaScript is a collection of key-value pairs where keys can be of any data type, and it maintains the order of insertion. The Set object, on the other hand, is a collection of unique values, ensuring that each value appears only once.

32. How do you implement inheritance in JavaScript<br>
    In JavaScript, inheritance can be implemented using prototype-based inheritance. You can create a constructor function and assign its prototype to be an instance of another constructor function. This way, the new object will inherit properties and methods from the parent constructor's prototype.

33. Explain the concept of lexical scoping in JavaScript.
    Lexical scoping, also known as static scoping, is a way to resolve variable names based on where they are defined in the source code. In JavaScript, the scope of a variable is determined by its location in the code, and it is determined at compile time, not runtime.

34. How do you convert a string to a number in JavaScript<br>
    You can convert a string to a number in JavaScript using the `parseInt()` or `parseFloat()` functions for integers and floating-point numbers, respectively. Additionally, using the unary plus operator (`+`) is an alternative way to convert a string to a number.

35. What is a callback hell, and how do you avoid it<br>
    Callback hell refers to the nesting of multiple callbacks inside each other, resulting in deeply indented and hard-to-read code. To avoid callback hell, you can use promises or async/await, which offer a more linear and readable way to handle asynchronous operations.

36. How do you use async/await for asynchronous operations in JavaScript<br>
    Async/await is a syntactical feature introduced in ES2017 that allows you to write asynchronous code in a more synchronous-looking manner. By using the `async` keyword before a function declaration, you can use the `await` keyword within the function to pause execution until an asynchronous operation is resolved.

37. Explain the concept of memoization in JavaScript.
    Memoization is a technique used to optimize functions by caching their results based on their input parameters. When a memoized function is called with the same set of arguments, it returns the cached result instead of recalculating it, reducing unnecessary computations.

38. What is event delegation in JavaScript<br>
    Event delegation is a technique where you add a single event listener to a parent element rather than individual listeners to multiple child elements. It allows you to handle events on dynamically created elements efficiently and improves performance by reducing the number of event listeners.

39. How do you reverse a string in JavaScript<br>
    You can reverse a string in JavaScript using various methods. One common way is to split the string into an array of characters, reverse the array using the `reverse()` method, and then join the characters back into a string using the `join()` method.

40. What is the purpose of the "arguments" object in JavaScript functions<br>
    The "arguments" object is a special array-like object available within JavaScript functions that contains all the arguments passed to the function. It allows you to access arguments dynamically, even if they are not explicitly defined in the function's parameter list.

41. How do you compare two objects in JavaScript<br>
    In JavaScript, objects are compared by reference, not by their content. Two objects with the same key-value pairs are considered equal only if they refer to the same object in memory. To compare the content of two objects, you need to manually compare their properties and values.

42. Explain the differences between "==" and "===" operators in JavaScript.
    The "==" (equality) operator in JavaScript performs type coercion, meaning it tries to convert operands to the same type before comparison. The "===" (strict equality) operator, on the other hand, compares both value and type without any type coercion. It is generally recommended to use "===" to avoid unexpected results.

43. How do you remove duplicate elements from an array in JavaScript<br>
    You can remove duplicate elements from an array in JavaScript using various methods, such as using the `Set` object to store unique values or using the `filter()` method to create a new array with only unique elements.

44. What is the purpose of the "bind" method in JavaScript<br>
    The `bind()` method in JavaScript allows you to create a new function that has a specified "this" value and, optionally, partially pre-defined arguments. It is often used to bind a function to a specific context, ensuring that "this" behaves as expected.

45. How do you convert a string to lowercase or uppercase in JavaScript<br>
    You can convert a string to lowercase in JavaScript using the `toLowerCase()` method or to uppercase using the `toUpperCase()` method. Both methods return a new string with the converted case, leaving the original string unchanged.

46. Explain the concept of currying in JavaScript.
    Currying is a functional programming technique where a function with multiple arguments is transformed into a sequence of functions, each taking a single argument. The result of each function is passed as an argument to the next function until all arguments are processed, and the final result is returned.

47. What are generators in JavaScript<br>
    Generators are functions in JavaScript that allow you to pause and resume their execution using the `yield` keyword. They are denoted by an asterisk (\*) after the function keyword. Generators are useful for handling asynchronous operations, iterating through large datasets, and creating custom iterators.

48. How do you detect the browser's language in JavaScript<br>
    You can detect the browser's language in JavaScript using the `navigator.language` property, which returns the user's preferred language as a string. This information can be used to customize the user experience based on their language preference.

49. What is the purpose of the "instanceof" operator in JavaScript<br>
    The "instanceof" operator in JavaScript is used to check whether an object is an instance of a specific constructor or class. It returns true if the object is an instance of the given constructor or a constructor derived from it; otherwise, it returns false.

50. How do you use the spread operator in JavaScript<br>
    The spread operator (`...`) in JavaScript is used to split an iterable (such as an array or a string) into individual elements. It is commonly used for array manipulation, function arguments, and object copying.

51. Explain the differences between "var", "let", and "const" in terms of scope.<br>
    "var" is function-scoped, which means the variable is only accessible within the function where it is declared. "let" and "const" are block-scoped, which means they are limited to the block (enclosed within curly braces) where they are defined. Additionally, variables declared with "const" cannot be reassigned after declaration, while variables declared with "let" can be reassigned.

52. How do you handle cross-origin requests in JavaScript?<br>
    To handle cross-origin requests (CORS) in JavaScript, you need to set appropriate headers on the server-side to allow requests from different origins. Additionally, on the client-side, you can use the `XMLHttpRequest` or `Fetch API` to make CORS-enabled requests, and the server should respond with the appropriate CORS headers.

53. What is the purpose of the "Symbol" data type in JavaScript?<br>
    The "Symbol" data type in JavaScript was introduced in ES6 and represents a unique and immutable value. It is often used as an identifier for object properties to avoid naming collisions and ensure their uniqueness.

54. How do you sort an array of objects based on a property value in JavaScript?<br>
    You can sort an array of objects based on a property value using the `sort()` method with a custom comparison function. The comparison function takes two objects as arguments and compares the desired property to determine the sorting order.

55. Explain the concept of event loop in JavaScript.
    The event loop is a fundamental part of JavaScript's concurrency model. It continuously checks the call stack and the message queue, executing functions from the queue when the call stack is empty. This mechanism allows JavaScript to handle asynchronous operations efficiently.

56. What is the purpose of the "map" method in JavaScript arrays?<br>
    The "map()" method in JavaScript arrays is used to create a new array by applying a transformation function to each element of the original array. It does not modify the original array but returns a new one with the transformed values.

57. How do you create a private variable in JavaScript?<br>
    JavaScript does not have native support for private variables. However, you can achieve private variables using closures or ES6 modules. By defining a variable inside a closure or module, it becomes inaccessible from the outside, effectively making it private.

58. Explain the differences between "setTimeout" and "setInterval" functions in JavaScript.
    Both "setTimeout" and "setInterval" are used to schedule the execution of a function after a certain delay. The difference lies in how they behave: "setTimeout" executes the function once after the delay, while "setInterval" repeatedly executes the function at fixed intervals until stopped.

59. What is a pure function in JavaScript?<br>
    A pure function is a function that always produces the same output for the same input and has no side effects. It does not modify external state or rely on external state changes, making it more predictable and easier to reason about.

60. How do you implement a debounce function in JavaScript?<br>
    A debounce function is used to delay the execution of a function until after a specific time has passed without any further calls. You can implement a debounce function by using the `setTimeout` function to delay the execution and clearing the timeout if the function is called again within the specified time.

61. What is the purpose of the "reduce" method in JavaScript arrays?<br>
    The "reduce()" method in JavaScript arrays is used to accumulate the elements of an array into a single value. It takes a callback function and an initial value as arguments and applies the callback function to each element, updating the accumulated result at each step.

62. How do you create a custom event in JavaScript?<br>
    To create a custom event in JavaScript, you can use the `CustomEvent` constructor. First, create a new instance of the `CustomEvent`, passing the event name and an optional `detail` object. Then, dispatch the custom event on the target element using the `dispatchEvent` method.

63. Explain the concept of call stack in JavaScript.
    The call stack is a data structure used by the JavaScript runtime to keep track of function calls and their execution context. When a function is called, it is added to the top of the call stack, and when it returns, it is removed from the stack.

64. What is the purpose of the "Object" constructor in JavaScript?<br>
    The "Object" constructor in JavaScript is a built-in constructor function used to create new objects. Objects created with this constructor have the `Object.prototype` as their prototype, and they inherit properties and methods from it.

65. How do you handle CORS (Cross-Origin Resource Sharing) in JavaScript?<br>
    To handle Cross-Origin Resource Sharing (CORS) in JavaScript, you need to set appropriate headers on the server-side to allow requests from different origins. Additionally, on the client-side, you can use the `XMLHttpRequest` or `Fetch API` to make CORS-enabled requests, and the server should respond with the appropriate CORS headers.

66. Explain the differences between synchronous and asynchronous code execution.
    Synchronous code execution means that each statement is executed one after another, and the program waits for each operation to complete before moving to the next one. Asynchronous code execution allows non-blocking operations, and the program can continue executing while waiting for certain tasks to finish.

67. How do you find the maximum and minimum values in an array in JavaScript?<br>
    To find the maximum and minimum values in an array, you can use the `Math.max()` and `Math.min()` functions with the spread operator (`...`) to spread the array elements as arguments.

68. What are the different ways to create objects in JavaScript?<br>
    There are several ways to create objects in JavaScript. You can use object literals `{}`, constructor functions with `new`, the `Object.create()` method, and class syntax introduced in ES6.

69. How do you check if a property exists in an object in JavaScript?<br>
    You can check if a property exists in an object using the `hasOwnProperty()` method or the `in` operator. The `hasOwnProperty()` method returns true if the object has the specified property, while the `in` operator checks if the property exists anywhere in the prototype chain.

70. What is the purpose of the "charAt" method in JavaScript strings?<br>
    The "charAt()" method in JavaScript strings is used to retrieve the character at a specified index in the string. It returns an empty string if the index is out of range.

71. How do you convert an object to an array in JavaScript?<br>
    You can convert an object to an array using the `Object.entries()` method, which returns an array of `[key, value]` pairs for each property in the object.

72. Explain the concept of functional programming in JavaScript.
    Functional programming is a programming paradigm that treats computation as the evaluation of mathematical functions, avoiding mutable data and state changes. In JavaScript, you can use higher-order functions, pure functions, and function composition to apply functional programming principles.

73. What is the purpose of the "isNaN" function in JavaScript?<br>
    The `isNaN()` function is used to check whether a value is NaN (Not-a-Number) in JavaScript. It returns true if the value is NaN and false otherwise.

74. How do you check if a property exists in an object in JavaScript?<br>
    You can check if a property exists in an object using the `hasOwnProperty()` method or the `in` operator. The `hasOwnProperty()` method returns true if the object has the specified property, while the `in` operator checks if the property exists anywhere in the prototype chain.

75. What is the purpose of the "filter" method in JavaScript arrays?<br>
    The `filter()` method in JavaScript arrays is used to create a new array with elements that pass a specific condition. It takes a callback function as an argument, which should return true for elements to be included in the new array.

76. How do you convert a string to a date object in JavaScript?<br>
    You can convert a string to a date object in JavaScript using the `Date` constructor or the `Date.parse()` method. The `Date` constructor can accept various date formats, while `Date.parse()` returns the number of milliseconds since January 1, 1970 (Unix epoch).

77. Explain the differences between "splice" and "slice" methods in JavaScript arrays.
    The `splice()` method is used to add or remove elements from an array by modifying the original array, while the `slice()` method creates a new array by extracting a portion of the original array based on specified start and end indices.

78. What is the purpose of the "JSON.stringify" method in JavaScript?<br>
    The `JSON.stringify()` method is used to convert a JavaScript object or value to a JSON string representation. This is helpful when sending data to a server or storing data in a file, as JSON is a widely used data interchange format.

79. How do you detect the user's browser in JavaScript?<br>
    You can detect the user's browser in JavaScript by accessing the `navigator` object. It provides various properties such as `userAgent`, `appName`, and `appVersion`, which can be used to determine the user's browser.

80. Explain the differences between "Object.create" and "new" keyword for object creation.
    `Object.create()` is used to create a new object with a specified prototype, while the `new` keyword is used with constructor functions to create new instances of objects. The `new` keyword also automatically sets the newly created object as the `this` value inside the constructor function.

81. What is the purpose of the "parseInt" function in JavaScript?<br>
    The `parseInt()` function in JavaScript is used to convert a string to an integer. It parses the string from left to right until it encounters a non-numeric character and returns the integer value of the numeric portion.

82. How do you handle memory leaks in JavaScript?<br>
    Memory leaks in JavaScript can occur when objects are not properly cleaned up, leading to unnecessary memory consumption. To handle memory leaks, make sure to remove event listeners, clear intervals or timeouts, and release references to objects when they are no longer needed.

83. What is the purpose of the "concat" method in JavaScript arrays?<br>
    The `concat()` method in JavaScript arrays is used to combine two or more arrays, creating a new array with the elements of the original arrays.

84. How do you convert an array to a string in JavaScript?<br>
    You can convert an array to a string in JavaScript using the `join()` method, which concatenates the elements of the array into a single string using a specified separator.

85. Explain the concept of functional programming in JavaScript.
    Functional programming is a programming paradigm that treats computation as the evaluation of mathematical functions, avoiding mutable data and state changes. In JavaScript, you can use higher-order functions, pure functions, and function composition to apply functional programming principles.

86. What is the purpose of the "isNaN" function in JavaScript?<br>
    The `isNaN()` function is used to check whether a value is NaN (Not-a-Number) in JavaScript. It returns true if the value is NaN and false otherwise.

87. How do you check if a property exists in an object in JavaScript?<br>
    You can check if a property exists in an object using the `hasOwnProperty()` method or the `in` operator. The `hasOwnProperty()` method returns true if the object has the specified property, while the `in` operator checks if the property exists anywhere in the prototype chain.

88. What is the purpose of the "filter" method in JavaScript arrays?<br>
    The `filter()` method in JavaScript arrays is used to create a new array with elements that pass a specific condition. It takes a callback function as an argument, which should return true for elements to be included in the new array.

89. How do you convert a string to a date object in JavaScript?<br>
    You can convert a string to a date object in JavaScript using the `Date` constructor or the `Date.parse()` method. The `Date` constructor can accept various date formats, while `Date.parse()` returns the number of milliseconds since January 1, 1970 (Unix epoch).

90. Explain the differences between "splice" and "slice" methods in JavaScript arrays.
    The `splice()` method is used to add or remove elements from an array by modifying the original array, while the `slice()` method creates a new array by extracting a portion of the original array based on specified start and end indices.

91. What is the purpose of the "JSON.stringify" method in JavaScript?<br>
    The `JSON.stringify()` method in JavaScript is used to convert a JavaScript object or value to a JSON string representation. This is helpful when sending data to a server or storing data in a file, as JSON is a widely used data interchange format.

92. How do you detect the user's browser in JavaScript?<br>
    You can detect the user's browser in JavaScript by accessing the `navigator` object. It provides various properties such as `userAgent`, `appName`, and `appVersion`, which can be used to determine the user's browser.

93. Explain the differences between "Object.create" and "new" keyword for object creation.
    `Object.create()` is used to create a new object with a specified prototype, while the `new` keyword is used with constructor functions to create new instances of objects. The `new` keyword also automatically sets the newly created object as the `this` value inside the constructor function.

94. What is the purpose of the "parseInt" function in JavaScript?<br>
    The `parseInt()` function in JavaScript is used to convert a string to an integer. It parses the string from left to right until it encounters a non-numeric character and returns the integer value of the numeric portion.

95. How do you handle memory leaks in JavaScript?<br>
    Memory leaks in JavaScript can occur when objects are not properly cleaned up, leading to unnecessary memory consumption. To handle memory leaks, make sure to remove event listeners, clear intervals or timeouts, and release references to objects when they are no longer needed.

96. What is the purpose of the "concat" method in JavaScript arrays?<br>
    The `concat()` method in JavaScript arrays is used to combine two or more arrays, creating a new array with the elements of the original arrays.

97. How do you convert an array to a string in JavaScript?<br>
    You can convert an array to a string in JavaScript using the `join()` method, which concatenates the elements of the array into a single string using a specified separator.

98. Explain the concept of functional programming in JavaScript.
    Functional programming is a programming paradigm that treats computation as the evaluation of mathematical functions, avoiding mutable data and state changes. In JavaScript, you can use higher-order functions, pure functions, and function composition to apply functional programming principles.

99. What is the purpose of the "isNaN" function in JavaScript?<br>
    The `isNaN()` function is used to check whether a value is NaN (Not-a-Number) in JavaScript. It returns true if the value is NaN and false otherwise.

100. How do you check if a property exists in an object in JavaScript?<br>
     You can check if a property exists in an object using the `hasOwnProperty()` method or the `in` operator. The `hasOwnProperty()` method returns true if the object has the specified property, while the `in` operator checks if the property exists anywhere in the prototype chain.
