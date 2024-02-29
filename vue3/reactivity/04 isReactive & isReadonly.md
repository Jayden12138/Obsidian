> isReactive & isReadonly
> 思路是一样的，在前一集实现 readonly 时，对 reactive 的重构，将 get 和 set 都统一处理，通过传入的参数 isReadonly 来判断当前需要创建的 get 函数是 reactive 的还是 readonly 的
> 当访问数据中的字段时会触发 get 操作，即可以在 get 中获取到这个 isReadonly，通过 isReadonly 来判断当前的 target 数据isReactive 或 isReadonly
# isReactive
## UT
```javascript

it('isReactive happy path', () => {
	const original = { foo: 1 }
	const observed = reactive(original)
	expect(observed).not.toBe(original)
	expect(observed.foo).toBe(1)
	expect(isReactive(observed)).toBe(true)
	expect(isReactive(original)).toBe(false)
})

```

# isReadonly
## UT
```javascript

it('isReadonly happy path', () => {
	const original = { foo: 1 }
	const observed = readonly(original)
	expect(observed).not.toBe(original)
	expect(observed.foo).toBe(1)
	expect(isReadonly(observed)).toBe(true)
	expect(isReadonly(original)).toBe(false)
})

```
