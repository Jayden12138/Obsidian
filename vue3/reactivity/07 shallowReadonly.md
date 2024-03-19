## UT
```javascript
it('happy path', () => {
	const original = { foo: 1, bar: { baz: 2 } }
	const observed = shallowReadonly(original)
	expect(observed).not.toBe(original)
	expect(observed.foo).toBe(1)
	expect(isReadonly(observed)).toBe(true)
	expect(isReadonly(observed.bar)).toBe(false)
})

it('should not allow reassignment', () => {
	console.warn = jest.fn()
	const original = { foo: 1 }
	const observed = shallowReadonly(original)
	observed.foo = 2
	expect(console.warn).toHaveBeenCalled()
})
```


## thinking


```javascript















// 秒！
// 使用 extend 复用了与readonly相同的部分
export const shallowReadonlyHandlers = extend({}, readonlyHandlers, {
	get: shallowReadonlyGet,
})



// 因为shallowxx不需要进行nested处理，也不需要进行依赖的收集，
// 所以这里可以单独去判断是否shallow
if(shallow){
	return res
}


```



## code










