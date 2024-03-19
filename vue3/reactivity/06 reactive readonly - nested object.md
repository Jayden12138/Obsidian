## case

```javascript
it('nested object reactive', () => {
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

 it('nested object readonly', () => {
	const original = { foo: 1, bar: { baz: 2 } }
	const observed = readonly(original)
	expect(observed).not.toBe(original)
	expect(observed.foo).toBe(1)
	expect(isReadonly(observed)).toBe(true)
	expect(isReadonly(original)).toBe(false);
	expect(isReadonly(observed.bar)).toBe(true);
	expect(isReadonly(original.bar)).toBe(false);
})
```


## thinking

```

proxy getter

```


## code

```javascript


// 不动原来的对象
// 在getter中对数据进行包裹

// proxy getter中判断当前访问的数据是否为对象，
// 如果是对象则通过reactive/readonly进行包裹返回
// 如果不是对象则走之前的逻辑（返回通过Reflect获取到的原始值）


function createGetter(isReadonly){
	return (target, key)=>{
		
		...
		
		const res = Reflect.get(target, key)

		if(isObject(res)){
			return isReadonly ? readonly(res) : reactive(res)
		}

		...

		return res
	}
}




```