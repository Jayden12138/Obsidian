
## case

```javascript

describe('reactive', ()=>{
	it('isReactive happy path', () => {
		const original = { foo: 1 }
		const observed = reactive(original)
		expect(observed).not.toBe(original)
		expect(observed.foo).toBe(1)
		expect(isReactive(observed)).toBe(true)
		expect(isReactive(original)).toBe(false)
	})
	it('isReadonly happy path', () => {
		const original = { foo: 1 }
		const observed = readonly(original)
		expect(observed).not.toBe(original)
		expect(observed.foo).toBe(1)
		expect(isReadonly(observed)).toBe(true)
		expect(isReadonly(original)).toBe(false)
	})
})

```

## thinking

1. isReactive接收一个对象，判断他是否是reactive
2. 普通对象和响应式对象的区别在于，响应式对象通过reactive包裹了一层，这里可以在reactive方法中做些手脚
	1. 如果触发了用于拦截的get方法，则说明这个对象经过reactive包裹过，是响应式对象（obj\['__v_isReactive']）
	2. 在get中，匹配key值，如果当前访问的是__v_isReactive，则返回!isReadonly
	3. (这里的isReadonly 是在实现readonly时，将reactive、readonly的实现封装后，通过这个flag来区分时readonly或者reactive)
3. isReadonly 和isReactive差不多，直接返回isReadonly即可

```javascript



```

小优化点：
1. 使用字符串__v_isReactive不好，使用enum改写

```javascript

export enum ReactiveFlags{
	IS_REACTIVE = '__v_isReactive'
}

if(ReactiveFlags.IS_REACTIVE){
	...
}


```

## code

[isReactive&isReadonly - Jayden Github](https://github.com/Jayden12138/tiny-vue/pull/26)

```javascript


```


## source code


```javascript




```