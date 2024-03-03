
> 标记类型
> 类型判断


这里选择使用了位运算的方式进行类型判断

```javascript
const enum ShapeFlags{
	element: 1, // 0001
	statefulComponent: 1 << 1, // 0010 == 2
	text_children: 1 << 2, // 0100 == 4
	array_children: 1 << 3 // 1000 == 5
}
```

但需要校验type 是否是 element 时，只需要进行&运算

```javascript

// currentType: 0000
// 校验是否时 element （0010）


	0000
&.  0010
	0000
	false
	当前 type 不是 element

// currentType: 0010
	
	0010
&.  0010
	0010
	true
	当前 type 是 element

	
```


如果需要，为某个 type 添加一个类型，则执行|运算
```javascript

// 当前 vnode type 是 App （根组件），vnode.shapeFlag 是 statefulComponent
// 在 createVNode 中，需要去判断当前 vnode.children是什么类型
// 假设根组件下只有一个 string 作为 children，则此时 vnode.type需要在原来 statefulComponent 的基础上，添加上text_children类型

vnode = { shapeFlag: 0010 }
vnode.children = '123'

if(typeof children == 'string'){
	vnode.shapeFlag |= 0100
}


// 最后 vnode.shapeFlag 为 0110
// 也就是当前 vnode.shapeFlag记录了当前自身的 type 以及 children 的type
// 方便后续 在 patch 时 进行判断，需要使用什么方法来进行 处理


```