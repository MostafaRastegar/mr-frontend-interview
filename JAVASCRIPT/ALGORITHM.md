# JavaScript algorithm interview questions along with their answers

1. Reverse a String
   Question: Write a function to reverse a given string.
   Answer:

```javascript
function reverseString(str) {
  return str.split("").reverse().join("");
}

// Example usage:
console.log(reverseString("hello")); // Output: "olleh"
```

<br/>

2. Find the Largest Number in an Array
   Question: Write a function to find the largest number in an array.
   Answer:

```javascript
function findLargestNumber(arr) {
  return Math.max(...arr);
}

// Example usage:
console.log(findLargestNumber([3, 7, 1, 9, 4])); // Output: 9
```

<br/>

3. Check for Palindrome
   Question: Write a function to check if a string is a palindrome (reads the same backward as forward).
   Answer:

```javascript
function isPalindrome(str) {
  const reversedStr = str.split("").reverse().join("");
  return str === reversedStr;
}

// Example usage:
console.log(isPalindrome("racecar")); // Output: true
```

<br/>

4. Factorial of a Number
   Question: Write a function to calculate the factorial of a given number.
   Answer:

```javascript
function factorial(num) {
  if (num === 0 || num === 1) {
    return 1;
  } else {
    return num * factorial(num - 1);
  }
}

// Example usage:
console.log(factorial(5)); // Output: 120
```

<br/>

5. Find the Longest Word in a String
   Question: Write a function to find the longest word in a given string.
   Answer:

```javascript
function findLongestWord(str) {
  const words = str.split(" ");
  return words.reduce(
    (longest, current) => (current.length > longest.length ? current : longest),
    ""
  );
}

// Example usage:
console.log(findLongestWord("The quick brown fox jumps over the lazy dog")); // Output: "jumps"
```

<br/>

6. Check for Prime Number
   Question: Write a function to check if a given number is a prime number.
   Answer:

```javascript
function isPrime(num) {
  if (num <= 1) return false;
  for (let i = 2; i <= Math.sqrt(num); i++) {
    if (num % i === 0) return false;
  }
  return true;
}

// Example usage:
console.log(isPrime(7)); // Output: true
```

<br/>

7. Title Case a Sentence
   Question: Write a function to convert the first letter of each word in a sentence to uppercase.
   Answer:

```javascript
function titleCase(str) {
  return str.toLowerCase().replace(/(^|\s)\w/g, (match) => match.toUpperCase());
}

// Example usage:
console.log(titleCase("hello world")); // Output: "Hello World"
```

<br/>

8. Return Largest Numbers in Arrays
   Question: Write a function to return an array containing the largest number from each sub-array in a given array of arrays.
   Answer:

```javascript
function largestNumbersInArrays(arr) {
  return arr.map((subArr) => Math.max(...subArr));
}

// Example usage:
console.log(
  largestNumbersInArrays([
    [1, 5, 8],
    [3, 9, 2],
    [4, 6, 7],
  ])
); // Output: [8, 9, 7]
```

<br/>

9. Confirm the Ending
   Question: Write a function to check if a given string ends with a target substring.
   Answer:

```javascript
function confirmEnding(str, target) {
  return str.slice(-target.length) === target;
}

// Example usage:
console.log(confirmEnding("Hello world", "world")); // Output: true
```

<br/>

10. Repeat a String
    Question: Write a function to repeat a given string a specified number of times.
    Answer:

```javascript
function repeatString(str, num) {
  if (num <= 0) return "";
  return str.repeat(num);
}

// Example usage:
console.log(repeatString("hello", 3)); // Output: "hellohellohello"
```

<br/>

11. Truncate a String
    Question: Write a function to truncate a string if it is longer than a specified maximum length and add "..." to the end.
    Answer:

```javascript
function truncateString(str, maxLength) {
  if (str.length > maxLength) {
    return str.slice(0, maxLength) + "...";
  } else {
    return str;
  }
}

// Example usage:
console.log(truncateString("Lorem ipsum dolor sit amet", 10)); // Output: "Lorem ipsu..."
```

<br/>

12. Find the Smallest Common Multiple
    Question: Write a function to find the smallest common multiple of the numbers in a given range.
    Answer:

```javascript
function smallestCommonMultiple(arr) {
  arr.sort((a, b) => a - b);
  let scm = arr[1];
  for (let i = arr[1]; i <= arr[0]; i++) {
    if (scm % i !== 0) {
      scm += arr[1];
      i = arr[1];
    }
  }
  return scm;
}

// Example usage:
console.log(smallestCommonMultiple([1, 5])); // Output: 60
```

<br/>

13. Seek and Destroy
    Question: Write a function to remove all elements from an array that are of the same value as the given arguments.
    Answer:

```javascript
function destroyer(arr, ...args) {
  return arr.filter((item) => !args.includes(item));
}

// Example usage:
console.log(destroyer([1, 2, 3, 4, 5], 2, 4)); // Output: [1, 3, 5]
```

<br/>

14. Wherefore art thou
    Question: Write a function that takes an array of objects and a source object. It should return an array of objects from the first array that have matching properties and values with the source object.
    Answer:

```javascript
function whatIsInAName(collection, source) {
  const sourceKeys = Object.keys(source);
  return collection.filter((item) => {
    return sourceKeys.every(
      (key) => item.hasOwnProperty(key) && item[key] === source[key]
    );
  });
}

// Example usage:
console.log(
  whatIsInAName(
    [
      { name: "apple", color: "red" },
      { name: "banana", color: "yellow" },
    ],
    { color: "red" }
  )
); // Output: [{ name: 'apple', color: 'red' }]
```

<br/>

15. FizzBuzz
    Question: Write a function that prints numbers from 1 to a given number. For multiples of 3, print "Fizz" instead of the number. For multiples of 5, print "Buzz." For numbers that are multiples of both 3 and 5, print "FizzBuzz."
    Answer:

```javascript
function fizzBuzz(num) {
  for (let i = 1; i <= num; i++) {
    if (i % 3 === 0 && i % 5 === 0) {
      console.log("FizzBuzz");
    } else if (i % 3 === 0) {
      console.log("Fizz");
    } else if (i % 5 === 0) {
      console.log("Buzz");
    } else {
      console.log(i);
    }
  }
}

// Example usage:
fizzBuzz(15); // Output: 1, 2, Fizz, 4, Buzz, Fizz, 7, 8, Fizz, Buzz, 11, Fizz, 13, 14, FizzBuzz
```

<br/>

16. Seek and Replace
    Question: Write a function that takes a sentence and replaces a given word with another word while preserving the word's case.
    Answer:

```javascript
function myReplace(sentence, wordToReplace, newWord) {
  const isUpperCase = wordToReplace[0].toUpperCase() === wordToReplace[0];
  const newWordWithCase = isUpperCase
    ? newWord[0].toUpperCase() + newWord.slice(1)
    : newWord;
  return sentence.replace(new RegExp(wordToReplace, "gi"), newWordWithCase);
}

// Example usage:
console.log(myReplace("He is a good person", "good", "kind")); // Output: "He is a kind person"
```

<br/>

17. DNA Pairing
    Question: Write a function that takes a DNA strand as a string and returns an array of its base pairs. In DNA, A pairs with T, and C pairs with G.
    Answer:

```javascript
function pairDNA(dnaStrand) {
  const dnaPairs = {
    A: "T",
    T: "A",
    C: "G",
    G: "C",
  };
  return dnaStrand.split("").map((base) => [base, dnaPairs[base]]);
}

// Example usage:
console.log(pairDNA("ATCG")); // Output: [["A","T"],["T","A"],["C","G"],["G","C"]]
```

<br/>

18. Missing Letters
    Question: Write a function that takes a string of consecutive letters (lowercase) and returns the missing letter in the sequence. If all letters are present, return undefined.
    Answer:

```javascript
function missingLetter(letters) {
  for (let i = 0; i < letters.length - 1; i++) {
    if (letters.charCodeAt(i + 1) - letters.charCodeAt(i) !== 1) {
      return String.fromCharCode(letters.charCodeAt(i) + 1);
    }
  }
}

// Example usage:
console.log(missingLetter("abcdefghjklmno")); // Output: "i"
```

<br/>

19. Sorted Union
    Question: Write a function that takes two or more arrays and returns a new array of unique values in the order of the original provided arrays.
    Answer:

```javascript
function uniteUnique(...arrays) {
  const uniqueValues = [].concat(...arrays);
  return [...new Set(uniqueValues)];
}

// Example usage:
console.log(uniteUnique([1, 2, 3], [3, 4, 5], [5, 6, 7])); // Output: [1, 2, 3, 4, 5, 6, 7]
```

<br/>

20. Convert HTML Entities
    Question: Write a function that converts the characters &, <, >, " (double quote), and ' (apostrophe) in a string to their corresponding HTML entities.
    Answer:

```javascript
function convertHTML(str) {
  const htmlEntities = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&apos;'
  };
  return str.replace(/[&<>"']/g, match => htmlEntities[match]);
}

// Example usage:
console.log(convertHTML('This is "quoted" & 'apo'strophe'.')); // Output: "This is &quot;quoted&quot; &amp; 'apo&apos;strophe'."
```

<br/>

21. Spinal Tap Case
    Question: Write a function that converts a string to spinal tap case. In spinal tap case, all words are lowercase and separated by hyphens.
    Answer:

```javascript
function spinalCase(str) {
  return str
    .replace(/([a-z])([A-Z])/g, "$1-$2")
    .toLowerCase()
    .replace(/\s|_/g, "-");
}

// Example usage:
console.log(spinalCase("ThisIsSpinalTap")); // Output: "this-is-spinal-tap"
```

<br/>

22. Sum All Primes
    Question: Write a function that sums all prime numbers up to and including a given number.
    Answer:

```javascript
function sumAllPrimes(num) {
  function isPrime(number) {
    if (number <= 1) return false;
    for (let i = 2; i <= Math.sqrt(number); i++) {
      if (number % i === 0) return false;
    }
    return true;
  }

  let sum = 0;
  for (let i = 2; i <= num; i++) {
    if (isPrime(i)) {
      sum += i;
    }
  }
  return sum;
}

// Example usage:
console.log(sumAllPrimes(10)); // Output: 17 (2 + 3 + 5 + 7)
```

<br/>

23. Smallest Common Multiple
    Question: Write a function that finds the smallest common multiple of the provided range of numbers.
    Answer:

```javascript
function smallestCommonMultiple(arr) {
  arr.sort((a, b) => a - b);
  let scm = arr[1];
  for (let i = arr[1]; i <= arr[0]; i++) {
    if (scm % i !== 0) {
      scm += arr[1];
      i = arr[1];
    }
  }
  return scm;
}

// Example usage:
console.log(smallestCommonMultiple([1, 5])); // Output: 60
```

<br/>

24. Drop It
    Question: Write a function that removes elements from an array until the passed function returns true. The function should return the remaining elements of the array.
    Answer:

```javascript
function dropElements(arr, func) {
  while (arr.length > 0 && !func(arr[0])) {
    arr.shift();
  }
  return arr;
}

// Example usage:
console.log(dropElements([1, 2, 3, 4], (n) => n >= 3)); // Output: [3, 4]
```

<br/>

25. Flatten a Nested Array
    Question: Write a function that takes a nested array and returns a flattened version (single-level array) of the original array.
    Answer:

```javascript
function flattenArray(arr) {
  return arr.reduce(
    (result, item) =>
      Array.isArray(item)
        ? result.concat(flattenArray(item))
        : result.concat(item),
    []
  );
}

// Example usage:
console.log(flattenArray([1, [2, [3, 4], 5], 6])); // Output: [1, 2, 3, 4, 5, 6]
```

<br/>

26. Steamroller
    Question: Write a function that flattens a nested array, removing any unnecessary levels of nesting.
    Answer:

```javascript
function steamrollArray(arr) {
  const result = [];
  flatten(arr);
  function flatten(arr) {
    arr.forEach((item) => {
      if (Array.isArray(item)) {
        flatten(item);
      } else {
        result.push(item);
      }
    });
  }
  return result;
}

// Example usage:
console.log(steamrollArray([1, [2], [3, [[4]]]])); // Output: [1, 2, 3, 4]
```

<br/>

27. Binary Agents
    Question: Write a function that converts a binary string into an English sentence. The binary string is space-separated.
    Answer:

```javascript
function binaryAgent(str) {
  return str
    .split(" ")
    .map((binary) => String.fromCharCode(parseInt(binary, 2)))
    .join("");
}

// Example usage:
console.log(
  binaryAgent(
    "01001001 00100000 01101100 01101111 01110110 01100101 00100000 01001000 01001111 01001101 01000101"
  )
); // Output: "I love HOME"
```

<br/>

28. Everything Be True
    Question: Write a function that checks if the second argument is truthy on all elements in a collection (first argument) with the specified property.
    Answer:

```javascript
function truthCheck(collection, pre) {
  return collection.every(
    (obj) => obj.hasOwnProperty(pre) && Boolean(obj[pre])
  );
}

// Example usage:
console.log(
  truthCheck(
    [
      { name: "Alice", age: 30 },
      { name: "Bob", age: NaN },
    ],
    "age"
  )
); // Output: false
```

<br/>

29. Arguments Optional
    Question: Write a function that adds two arguments together. If only one argument is provided, the function should return a new function that expects a second argument of type number. The returned function should then add the first argument and the second argument together.
    Answer:

```javascript
function addTogether(a, b) {
  if (typeof a !== "number") return undefined;
  if (b !== undefined) {
    if (typeof b === "number") return a + b;
    return undefined;
  } else {
    return function (b) {
      return typeof b === "number" ? a + b : undefined;
    };
  }
}

// Example usage:
console.log(addTogether(2, 3)); // Output: 5
console.log(addTogether(5)(7)); // Output: 12
```

<br/>

30. Make a Person
    Question: Write a function that takes a first and last name and returns an object with "firstName" and "lastName" properties. The returned object should also have methods for getting and setting these properties.
    Answer:

```javascript
function Person(firstName, lastName) {
  this.firstName = firstName;
  this.lastName = lastName;
  this.getFullName = function () {
    return this.firstName + " " + this.lastName;
  };
  this.setFullName = function (fullName) {
    const names = fullName.split(" ");
    this.firstName = names[0];
    this.lastName = names[1];
  };
}

// Example usage:
const person = new Person("John", "Doe");
console.log(person.getFullName()); // Output: "John Doe"
person.setFullName("Jane Smith");
console.log(person.getFullName()); // Output: "Jane Smith"
```

<br/>

31. Map the Debris
    Question: Write a function that takes an array of objects representing the orbital period for each satellite. Return a new array that contains the corresponding average altitude (orbital radius) rounded to the nearest whole number.
    Answer:

```javascript
function orbitalPeriod(arr) {
  const GM = 398600.4418;
  const earthRadius = 6367.4447;
  return arr.map(({ name, avgAlt }) => ({
    name,
    orbitalPeriod: Math.round(
      2 * Math.PI * Math.sqrt(Math.pow(earthRadius + avgAlt, 3) / GM)
    ),
  }));
}

// Example usage:
console.log(
  orbitalPeriod([
    { name: "iss", avgAlt: 413.6 },
    { name: "hubble", avgAlt: 556.7 },
  ])
); // Output: [{ name: 'iss', orbitalPeriod: 5557 }, { name: 'hubble', orbitalPeriod: 5734 }]
```

<br/>

32. Pairwise
    Question: Write a function that takes an array and a target value. It should find unique pairs of indices whose values add up to the target and return the sum of their indices.
    Answer:

```javascript
function pairwise(arr, target) {
  let sum = 0;
  const usedIndices = [];
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[i] + arr[j] === target && !usedIndices.includes(i) && !usedIndices.includes(j)) {
        sum += i + j;
        usedIndices.push(i, j);
      }
    }
  }
  return sum;
}

// Example usage:
console.log(pairwise([1, 4, 2, 3, 0, 5], 7)); // Output: 11 (Indices

 1 and 2 sum to 7, indices 3 and 4 sum to 7, and their sum is 11)
```

<br/>

33. Find the Missing Letter
    Question: Write a function that takes an array of consecutive (lowercase) letters and returns the missing letter in the sequence. If all letters are present, return undefined.
    Answer:

```javascript
function findMissingLetter(letters) {
  for (let i = 0; i < letters.length - 1; i++) {
    if (letters.charCodeAt(i + 1) - letters.charCodeAt(i) !== 1) {
      return String.fromCharCode(letters.charCodeAt(i) + 1);
    }
  }
}

// Example usage:
console.log(findMissingLetter("abcdefghjklmno")); // Output: "i"
```

<br/>

34. Pig Latin
    Question: Write a function that translates a given string to Pig Latin. In Pig Latin, words that begin with a consonant have their first letter moved to the end of the word, followed by "ay." Words that begin with a vowel stay unchanged and end with "way."
    Answer:

```javascript
function translatePigLatin(str) {
  const vowels = ["a", "e", "i", "o", "u"];
  if (vowels.includes(str[0])) {
    return str + "way";
  } else {
    for (let i = 1; i < str.length; i++) {
      if (vowels.includes(str[i])) {
        return str.slice(i) + str.slice(0, i) + "ay";
      }
    }
  }
}

// Example usage:
console.log(translatePigLatin("banana")); // Output: "ananabay"
```

<br/>

35. Search and Replace
    Question: Write a function that performs a search-and-replace on a sentence using the provided word as the search term. The replacement should preserve the case of the original word.
    Answer:

```javascript
function myReplace(sentence, wordToReplace, newWord) {
  const isUpperCase = wordToReplace[0].toUpperCase() === wordToReplace[0];
  const newWordWithCase = isUpperCase
    ? newWord[0].toUpperCase() + newWord.slice(1)
    : newWord;
  return sentence.replace(new RegExp(wordToReplace, "gi"), newWordWithCase);
}

// Example usage:
console.log(myReplace("He is a good person", "good", "kind")); // Output: "He is a kind person"
```

<br/>
