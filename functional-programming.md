### Pure functions

Pure Functions are functions that do not produce any side effects. They always have an input, always return an output, and do not produce side effects:

```js
const purity = (input) => {
  var result = [];

  for (var i = 0; i < input.length; i++) {
    result.push(input[i] * 2)
  }

	return result;
};

const input = [1, 2, 4];
const output = purity(input);
// logs 2, 4, 8
console.log(output);

// logs false
console.log(input === output);
```

Impure functions produce side effects like modifying the input, or modifying global state:

```js
const impurity = (input) => {
  for (var i = 0; i < input.length; i++) {
    input[i] = input[i] * 2;
  }
};

const globalInput = [1, 2, 4];
const output = impurity(globalInput);
// logs undefined
console.log(output);

// logs false, input array was replaced in the function
console.log(globalInput[0] === 1);
```

### Immutable State

Immutable state avoids bugs introduced by different functions modifying the same object. Consider the previous example, where the `globalInput` array is being replaced by the first item. If another function was iterating over the `globalInput` array and it didn't check `typeof globalInput` then it would throw an error, potentially breaking javascript from executing on a page:

```js
const globalImpurity = () => {
  // replacing the globalInput array with an integer;
  globalInput = globalInput[0] * 2;
}

const pageBreaker = () => {
  globalInput.map(function(i) {
    i = i * 2;
  });
}

var globalInput = [2];
// logs 'undefined, 4'
console.log(globalImpurity(), globalInput);

// Uncaught TypeError: globalInput.forEach is not a function
console.log(pageBreaker());
```

If we refrain from mutating the `globalInput` array, then we would be able to call multiple functions on the same input in a safer manner. We can call them in any order and they will always produce the same output:

```js
const multiplyAllByTwo = (numbersArray) => {
  const result = numbersArray.map((item) => {
    return item * 2;
  });

  return result;
}

var numbers = [2, 4, 6 , 8];

// logs '[ 4, 8, 12, 16 ]'
console.log(multiplyAllByTwo(numbers));

// logs '[ 4, 8, 12, 16 ]'
console.log(multiplyAllByTwo(numbers));

// logs '[ 4, 8, 12, 16 ]'
console.log(multiplyAllByTwo(numbers));
```

### Declarative Vs Imperative code

In my opinion, good code is code that puts the machine to work instead of manually telling it what to do, and good code is also easy to read. Modifying existing code that you have never seen before is an everyday reality in Software Engineering and being able to pick it up fast is important from professional development and business standpoints. Declarative code abstracts away the thoughts of 'how is this working' to let engineers focus on 'what is this doing'. The following function takes an input array, multiplies every number in it, and stores the result in a separate array which is then returned.

```js
const modifyNumbers = (numbers) => {
  const newArray = [];
  for (var i = 0; i < numbers.length; i++) {
    newArray[i] = numbers[i] * 2;
  }

  return newArray;
}

const numbersArray = [1,2,3,5,8,13,21];
const modifiedNumbers = modifyNumbers(numbersArray)
// logs [ 2, 4, 6, 10, 16, 26, 42 ]
console.log(modifiedNumbers);
```

The downside is we have to read every line of the function to understand that this is a mapping function, we're taking an input array, modifying it, and returning a new array. The function could be named better to be more clear, or we could leverage declarative code:

```js
const numbersArray = [1,2,3,5,8,13,21];
const modifiedNumbers = numbersArray.map((item) => {
  return item * 2;
});

// logs [ 2, 4, 6, 10, 16, 26, 42 ]
console.log(modifiedNumbers);
```

### Function composition

Function Composition is a way to keep functions pure. Think of it as using small functions as building blocks of large functions. This way, we can keep functions pure, because they have one simple task, like the map function above. We know that it will always produce an array that is a modified version of a preexisting array. The following code filters out all of the even numbers of an array, multiplies the odd numbers by 2, and adds all of the numbers together.

```js
const numbersArray = [1,2,3,5,8,13,21];
const modifiedNumbers = numbersArray.filter((item) => {
  return item % 2 !== 0;
	// returns [ 1, 3, 5, 13, 21 ]
}).map((item) => {
  return item * 2;
	// returns [ 2, 6, 10, 26, 42 ]
}).reduce((accumulator, item) => {
  return accumulator + item;
	// this is the same as (2 + 6 + 10 + 26 + 42)
	// returns 86
});

// logs 86
console.log(modifiedNumbers);
```

Imagine the same with a `for loop`:

```js
const modifiedNumbers = (numbers) => {
  let result = 0;
  for (var i = 0; i < numbers.length; i++) {
    var item = numbers[i];
    if (item % 2 !== 0) {
      result = result + item * 2;
    }
  }

  return result;
}

const numbersArray = [1,2,3,5,8,13,21];

// logs 86
console.log(modifiedNumbers(numbersArray));
```

The code does the same thing but if you didn't write it, or you wrote it 6 months ago, it takes longer to understand. This also reinforces the idea of declarative code vs imperative.

### Higher-Order Functions & Callbacks (Two Common Terms)
Higher-Order Functions either accept a function as an argument, or return a function. Functions passed in as an argument to a Higher-Order Function are also known as callbacks. The example above contains callbacks and could be written differently to understand them better. The higher order functions in this case are `.filter`, `.map`, and `.reduce`, the callbacks are the anonymous functions being passed in:

```js
const numbersArray = [1,2,3,5,8,13,21];
const filterCallback = (item) => {
  return item % 2 !== 0;
	// returns [ 1, 3, 5, 13, 21 ]
};
const mapCallback = (item) => {
  return item * 2;
	// returns [ 2, 6, 10, 26, 42 ]
};
const reduceCallback = (accumulator, item) => {
  return accumulator + item;
	// this is the same as (2 + 6 + 10 + 26 + 42)
	// returns 86
};
const modifiedNumbers = numbersArray.filter(
	filterCallback
).map(
  mapCallback
).reduce(
  reduceCallback
);

// logs 86
console.log(modifiedNumbers);
```

#### Closures

Closures are when a function returns another function, if we had to call an API for the proper reduce algorithm, then we would have to wait until the ajax call is complete before we could pass in the mapped Array:

```js
const numbersArray = [1,2,3,5,8,13,21];
const modifiedNumbers = numbersArray.filter((item) => {
  return item % 2 !== 0;
	// returns [ 1, 3, 5, 13, 21 ]
}).map((item) => {
  return item * 2;
	// returns [ 2, 6, 10, 26, 42 ]
});

const fakeAjax = {};
fakeAjax.complete = () => {
	const reduceClosure = (accumulator, item) => {
		return accumulator + item;
		// returns 86
	}

	return reduceClosure;
}

// ajax call executed, and is complete
const reduceAlgorithm = fakeAjax.complete();

// logs 86
console.log(modifiedNumbers.reduce(reduceAlgorithm));
```
