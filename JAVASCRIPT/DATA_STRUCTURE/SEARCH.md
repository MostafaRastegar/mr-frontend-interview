# Implementations of linear search, binary search (in a sorted array), and searching in JavaScript

1. Linear Search in an Array:

```javascript
function linearSearch(arr, target) {
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === target) {
      return i; // Return the index if the element is found
    }
  }
  return -1; // Return -1 if the element is not found
}

// Example usage:
const myArray = [10, 5, 7, 2, 15];
console.log(linearSearch(myArray, 7)); // Output: 2 (index of 7 in the array)
console.log(linearSearch(myArray, 20)); // Output: -1 (element not found)
```

2. Binary Search in a Sorted Array:

```javascript
function binarySearch(arr, target) {
  let left = 0;
  let right = arr.length - 1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);

    if (arr[mid] === target) {
      return mid; // Return the index if the element is found
    } else if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  return -1; // Return -1 if the element is not found
}

// Example usage:
const sortedArray = [2, 5, 7, 10, 15];
console.log(binarySearch(sortedArray, 7)); // Output: 2 (index of 7 in the sorted array)
console.log(binarySearch(sortedArray, 20)); // Output: -1 (element not found)
```

3. Searching in a JavaScript Object:

```javascript
function searchInObject(obj, key, target) {
  if (obj.hasOwnProperty(key) && obj[key] === target) {
    return key; // Return the key if the element is found
  }
  return null; // Return null if the key or target is not found
}

// Example usage:
const myObject = { name: "John", age: 30, city: "New York" };
console.log(searchInObject(myObject, "age", 30)); // Output: "age" (key found)
console.log(searchInObject(myObject, "city", "Los Angeles")); // Output: null (key or target not found)
```

Remember that the implementations provided above are basic and don't handle all possible edge cases. Depending on the use case, you might need to add additional checks and optimizations to the search algorithms. Also, note that binary search requires the array to be sorted beforehand.
