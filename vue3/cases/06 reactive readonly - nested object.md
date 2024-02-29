## UT
```javascript
test('nested reactives', () => {
	  const original = {
		nested: {
		  foo: 1,
		},
		array: [{ bar: 2 }],
	  };
	  const observed = reactive(original);
	  expect(isReactive(observed.nested)).toBe(true);
	  expect(isReactive(observed.array)).toBe(true);
	  expect(isReactive(observed.array[0])).toBe(true);
});
```

