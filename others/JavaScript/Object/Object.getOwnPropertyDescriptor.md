
![[Pasted image 20240522114709.png]]

在vue2源码中有使用到这个API
他会返回一个对象，是该对象属性的描述，修改这个对象不会对原始对象有任何修改

![[Pasted image 20240522115252.png]]

```ts

const obj = { foo: 1 }

const objFooDes = Object.getOwnPropertyDescriptor(obj, 'foo')

/**
	// objFooDes
	{
		configurable: true,
		enumerable: true,
		value: 1,
		writable: true
	}
*/


```

configurable：是否可以配置
enumerable：是否可迭代
value：值
writable: 是否可写

