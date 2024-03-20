

>proxyRefs edge case
>
>1. 当传入是reactive直接返回
>2. 当传入的值是array且访问时使用的是int类型的key，则需要通过.value来访问（其余状态可以不用.value来获取）
>
>实际看代码修改的点主要在reactive中，需要在reactive get中，在track前进行判断，如果访问的key不是可以track的就直接返回res，不然会在执行track时报错

在收集的时候发现会有很多其实不需要track的key，需要过滤

![[Pasted image 20240320145335.png]]

```js




it('should return as-is when original is reactive', () => {
	const user = {
		age: ref(10),
		name: 'Jayden'
	}
	const original = reactive(user)
	const proxyUser = proxyRefs(original)

	expect(proxyUser).toBe(original)
	expect(proxyUser.age).toBe(10)
})

it('should use .value get value when it is ref in Array', () => {
	const arrWithRefs = [ref(1), 2, 3]
	const original = reactive(arrWithRefs)
	const proxyUser: any = proxyRefs(original)

	expect(proxyUser).toBe(original)
	expect(proxyUser[0].value).toBe(1)
})


```