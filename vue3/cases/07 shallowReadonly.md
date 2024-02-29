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

shallowReadonly 
处理嵌套对象时，只有第一层时 readonly，嵌套的对象不予处理
和 readonly 一样，也不允许 set


在 添加 readonly 时，对 reactive 和 readonly 的 get set 进行了重构，通过 createGetter 进行创建 get 方法

createGetter(isReadonly, shallow)

添加一个入参 shallow，来判断是否是 shallowReadonly，如果是 shallowReadonly，则不进行处理嵌套对象，也不进行 track 依赖的收集（因为不能 set 也就不需要在 get 的时候进行依赖收集）

这里因为 shallowReadonly 和 readonly 的 set 一样，所以直接使用了 extend (== Object.assign)，复用了 readonly 上的 set 方法，重写了 get 方法
```javascript
extend({}, readonlyHandlers, {
	get: ShallowReadonlyHandlers
})
```

