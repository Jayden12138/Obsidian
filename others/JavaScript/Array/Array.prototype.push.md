[Array.prototype.push - ecma262](https://tc39.es/ecma262/multipage/indexed-collections.html#sec-array.prototype.push)


```js

function pushPolyfill(...items) {
	// Step 1
	let o = Object(this)
	if (o === null || o === undefined) {
		throw new TypeError('Cannot convert undefined or null to object')
	}

	// Step 2
	let len = o.length >>> 0

	// Step 4
	if (len + items.length > 2 ** 53 - 1) {
		throw new TypeError('The array is too large')
	}

	// Step 5
	for (let i = 0; i < items.length; i++) {
		const E = items[i]
		// Step 5a
		o[len] = E
		// Step 5b
		len++
	}

	// Step 6
	o.length = len

	// Step 7
	return len
}

// Test the polyfill
const arr = [1, 2, 3]
const result = pushPolyfill.call(arr, 4, 5, 6)
console.log(arr) // Output: [1, 2, 3, 4, 5, 6]
console.log(result) // Output: 6





```