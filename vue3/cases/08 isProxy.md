## UT
```javascript
it('isProxy happy path reactive', () => {

	const original = { foo: 1, bar: { baz: 2 } }
	
	const observed = readonly(original)
	
	expect(isProxy(observed)).toBe(true)

})

it('isProxy happy path readonly', () => {

	const original = { foo: 1, bar: { baz: 2 } }
	
	const observed = reactive(original)
	
	expect(isProxy(observed)).toBe(true)

})
```

非常的简单

https://cn.vuejs.org/api/reactivity-utilities.html#isproxy
官方解释，该 api 用于检查一个对象是否是通过 reactive readonly shallowReadonly shallowReactive创建的代理

前面已经实现了isReactive 以及 isReadonly

这里只需要调用即可

```javascript
export const isProxy(value){
	return isReactive(value) || isReadonly(value)
}
```