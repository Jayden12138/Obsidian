## 流程搭建

> case: 当count更新时，视图需要更新

```javascript

import { h, ref } from '../../lib/guide-mini-vue.esm.js'

export const App = {
	name: 'App',
	setup() {
		const count = ref(0)

		const onClick = () => {
			count.value++
			console.log(count.value)
		}

		return {
			count,
			onClick,
		}
	},
	render() {
		console.log(this.count)
		return h(
			'div',
			{
				id: 'root',
			},
			[
				h('div', {}, 'count: ' + this.count),
				h('button', { onClick: this.onClick }, 'update'),
			]
		)
	},
}


```

> 1. ref 正确导出
> 2. 上述App.js中的this.count可以正确渲染 - [[11 proxyRefs | proxyRefs]]
> 3. patchElement

2

```javascript

// 一开始渲染时，访问this.count并没有直接返回count.value，而是返回了ref
// 在render函数中，对ref的访问是不需要通过.value的，所以这里需要借助proxyRefs来将setup的返回值进行包裹
// proxyRefs的作用 - 如果访问value是ref，则返回ref.value，否则直接返回value


// runtime-core/component.ts
function handleSetupResult(instance, setupResult){
	if(typeof setupResult === 'function'){
		instance.setupResult = ProxyRefs(setupResult)
	}
	...
}


```


3

3-1 effect

需要在响应式对象发生改变的时候，重新执行render函数，以获取到新的subTree（虚拟节点）

因为响应式对象会在get时收集依赖(track)，在更新时触发所收集到的所有依赖(trigger)，这里可以通过effect将render进行包裹，
当第一次执行render时，访问响应式对象的数据，触发get 进行收集依赖，将当前的render函数进行收集，
之后响应式对象发生了改变，触发set 触发依赖，将收集到的依赖遍历执行（这里会将render重新执行，这样就可以获取到新的subTree）

```javascript

// runtime-core/renderer.ts
function setupRenderEffect(instance, initialVNode, container) {
	effect(() => {
		const { proxy } = instance
		
		// 执行 render
		const subTree = instance.render.call(proxy)

		patch(subTree, container, instance)

		instance.isMounted = true
		// 处理完
		initialVNode.el = subTree.el
	})
}


```



3-2 需要区分mount | update

> 在实例上新建一个flag isMounted = false

```javascript


// runtime-core/renderer.ts
function setupRenderEffect(instance, initialVNode, container) {
	effect(() => {
		const { proxy } = instance
		if(!instance.isMounted){
			// 执行 render
			const subTree = instance.render.call(proxy)
	
			patch(subTree, container, instance)
	
			instance.isMounted = true
			// 处理完
			initialVNode.el = subTree.el
		}else{
			console.log('update')
		}
	})
}


```

3-3 新旧 vnode

在触发update后，需要进行新旧vnode对比，所以在首次mount时需要保存下subTree

在update中可以通过instance.subTree获取到上一次的subTree，并通过再次执行render得到新的subTree

这里需要更新下patch相关函数，之前只是处理了mount，现在需要传入两个vnode，一新一旧，用于对比后更新


```javascript

// runtime-core/renderer.ts
function setupRenderEffect(instance, initialVNode, container) {
	effect(() => {
		const { proxy } = instance
		if(!instance.isMounted){
			// 执行 render
			const subTree = (instance.subTree = instance.render.call(proxy))
	
			patch(null, subTree, container, instance)
	
			instance.isMounted = true
			// 处理完
			initialVNode.el = subTree.el
		}else{
			console.log('update')

			const prevSubTree = instance.subTree
			const subTree = instance.render.call(proxy)

			patch(prevSubTree, subTree, container, instance)

			// 更新
			instance.subTree = subTree

		}
	})
}


```


## 更新props

### cases

foo: foo -> foo: new-foo -> 修改
foo: foo -> foo: null | undefined -> 删除
 foo: foo, bar: bar -> foo: foo -> 删除bar

```javascript

import { h, ref } from '../../lib/guide-mini-vue.esm.js'

/**
 * 1. foo: foo -> foo: new-foo -> 修改
 * 2. foo: foo -> foo: null | undefined -> 删除
 * 3. foo: foo, bar: bar -> foo: foo -> 删除bar
 */

export const App = {
	name: 'App',
	setup() {
		const props = ref({ foo: 'foo', bar: 'bar' })

		const onChangePropsDemo1 = () => {
			props.value.foo = 'new-foo'
		}

		const onChangePropsDemo2 = () => {
			props.value.foo = undefined
		}

		const onChangePropsDemo3 = () => {
			props.value = {
				foo: 'foo',
			}
		}

		return {
			props,
			onChangePropsDemo1,
			onChangePropsDemo2,
			onChangePropsDemo3,
		}
	},
	render() {
		return h(
			'div',
			{
				id: 'root',
				...this.props,
			},
			[
				h(
					'button',
					{ onClick: this.onChangePropsDemo1 },
					'onChangePropsDemo1'
				),
				h(
					'button',
					{ onClick: this.onChangePropsDemo2 },
					'onChangePropsDemo2'
				),
				h(
					'button',
					{ onClick: this.onChangePropsDemo3 },
					'onChangePropsDemo3'
				),
			]
		)
	},
}


```

### resolve

#### 修改

foo: foo -> foo: new-foo -> 修改

```javascript

// runtime-core/renderer.ts
// n1 旧节点
// n2 新节点
function patchElement(n1, n2, container){

	const prevProps = n1.props || {}
	const nextProps = n2.props || {}

	// el
	// 这里需要更新当前新subTree中的el，通过访问老节点中的el即可
	const el = n2.el = n1.el
	patchProps(el, prevProps, nextProps)

}


function patchProps(el, prevProps, nextProps){

	// 遍历 nextProps，判断prop是否和之前一致，如果不一致执行hostPatchProp
	// hostPatchProp 需要el，通过el来修改prop
	for(let key in nextProps){
		const prevProp = prevProps[key]
		const nextProp = nextProps[key]
		if(prevProp !== nextProp){
			hostPatchProp(el, key, prevProp, nextProp)
		}
	}

}


// runtime-dom/index.ts
// 需要传入新旧prop // TODO: 这里旧节点没有用到
function patchProp(el, key, prevVal, nextVal){
	if(isOn(key)){
		// on + Event 
		// eg: onClick => el.addEventListener()
		const event = key.slice(2).toLowerCase()
		el.addEventListener(event, nextVal)
	}else{
		el.setAttribute(key, nextVal)
	}
}


```

#### 删除（null | undefined）

foo: foo -> foo: null | undefined -> 删除

```javascript


// runtime-dom/index.ts
function patchProp(el, key, prevVal, nextVal){
	if(isOn(key)){
		const event = key.slice(2).toLowerCase()
		el.addEventListener(event, nextVal)
	}else{
		// 这里对新值进行判断，如果是null | undefined，则删除该prop
		if(nextVal == null){
			el.removeAttribute(key)
		}else{
			el.setAttribute(key, nextVal)
		}
	}
}

```

#### 删除

foo: foo, bar: bar -> foo: foo -> 删除bar

```javascript

function patchProps(el, prevProps, nextProps){
	for(let key in nextProps){
		const prevProp = prevProps[key]
		const nextProp = nextProps[key]
		if(prevProp !== nextProp){
			hostPatchProp(el, key, prevProp, nextProp)
		}
	}

	// 这里需要对旧节点props进行遍历，判断key是否在新值中存在，如果不存在需要删除
	for(let key in prevProps){
		if(!(key in nextProps)){
			hostPatchProp(el, key, prevProp, null)
		}
	}
}

```

### refactor

小优化点
1. 在patchProps中，只有当新旧节点props不一致时才去执行对比更新
2. 在遍历旧节点props前，判断如果之前props为空，则跳过 // 但这里不跳过感觉也可以，for in访问一个空的对象，也不会执行内部逻辑


> const EMPTY_OBJ = {}
> 这里全局定义了一个EMPTY_OBJ，初始获取props时为其赋值空，后续判断可以直接通过EMPTY_OBJ来判断，如果初始使用{}来赋值，则后续判断无法通过`!==`来判断是否为空



```javascript

// runtime-core/renderer.ts
const EMPTY_OBJ = {}

function patchElement(n1, n2, container){
	const prevProps = n1.props || EMPTY_OBJ
	const nextProps = n2.props || EMPTY_OBJ
	
	const el = n2.el = n1.el
	patchProps(el, prevProps, nextProps)

}

function patchProps(el, prevProps, nextProps) {
	if (prevProps !== nextProps) {
		for (let key in nextProps) {
			const nextProp = nextProps[key]
			const prevProp = prevProps[key]

			if (nextProp !== prevProp) {
				hostPatchProp(el, key, prevProp, nextProp)
			}
		}

		if (prevProps !== EMPTY_OBJ) {
			for (let key in prevProps) {
				if (!(key in nextProps)) {
					hostPatchProp(el, key, prevProps, null)
				}
			}
		}
	}
}

```



### 扩展（TODO）
#### 删除（使用delete操作符）

上述cases中的删除，是通过赋予新对象实现的，触发的是set，但如果使用delete来删除一个对象中的值，并不会触发set，而是会触发proxy中的deleteProperty，这个并没有在reactivity/reactive中实现，所以使用delete没有用

> [Proxy handler.deleteProperty - MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/deleteProperty)



## update Children

这里的children有text 或者 children类型，所以在update children中，按类型可以分为四种

text -> array
array -> text
text -> text
array -> array

### cases
#### array -> text

```javascript

// ArrayToText.js
import { h, ref } from '../../lib/guide-mini-vue.esm.js'

const nextChildren = 'newChildren'
const prevChildren = [h('div', {}, 'A'), h('div', {}, 'B')]

export default {
	name: 'ArrayToText',
	setup() {
		const isChange = ref(false)
		window.isChange = isChange

		return {
			isChange,
		}
	},
	render() {
		const self = this
		return self.isChange === true
			? h('div', {}, nextChildren)
			: h('div', {}, prevChildren)
	},
}


```



#### text -> text

```javascript

// TextToText.js

import { h, ref } from '../../lib/guide-mini-vue.esm.js'

const prevChildren = 'oldChildren'
const nextChildren = 'newChildren'

export default {
	name: 'TextToText',
	setup() {
		const isChange = ref(false)
		window.isChange = isChange

		return {
			isChange,
		}
	},
	render() {
		const self = this
		return self.isChange === true
			? h('div', {}, nextChildren)
			: h('div', {}, prevChildren)
	},
}


```




#### text -> array

```javascript

// TextToArray.js

import { h, ref } from '../../lib/guide-mini-vue.esm.js'

const prevChildren = 'newChildren'
const nextChildren = [h('div', {}, 'A'), h('div', {}, 'B')]

export default {
	name: 'ArrayToText',
	setup() {
		const isChange = ref(false)
		window.isChange = isChange

		return {
			isChange,
		}
	},
	render() {
		const self = this
		return self.isChange === true
			? h('div', {}, nextChildren)
			: h('div', {}, prevChildren)
	},
}




```



#### array -> array


##### 左侧对比


```javascript

// 1. 左侧的对比
// (a b) c
// (a b) d e
const prevChildren = [
	h('p', { key: 'A' }, 'A'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'C' }, 'C'),
]
const nextChildren = [
	h('p', { key: 'A' }, 'A'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'D' }, 'D'),
	h('p', { key: 'E' }, 'E'),
]

// init: i = 0, e1 = 2, e2 = 3
// 左端对比: i = 2, e1 = 2, e2 = 3

```


##### 右端对比

```javascript


// 2. 右侧的对比
// a (b c)
// d e (b c)
const prevChildren = [
	h('p', { key: 'A' }, 'A'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'C' }, 'C'),
]
const nextChildren = [
	h('p', { key: 'D' }, 'D'),
	h('p', { key: 'E' }, 'E'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'C' }, 'C'),
]

// init: i = 0, e1 = 2, e2 = 3
// 左端对比: i = 0, e1 = 2, e2 = 3
// 右端对比: i = 0, e1 = 0, e2 = 1

```

##### 新的比老的长 创建新的

```javascript

// 3. 新的比老的长 创建新的
// 左侧
// (a b)
// (a b) c
const prevChildren = [h('p', { key: 'A' }, 'A'), h('p', { key: 'B' }, 'B')]
const nextChildren = [
	h('p', { key: 'A' }, 'A'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'C' }, 'C'),
]

// init: i = 0, e1 = 1, e2 = 2
// 左端对比: i = 2, e1 = 1, e2 = 2
// 右端对比: i = 2, e1 = 1, e2 = 2
// i <= e2  [i, e2] -> patch 


```



##### 中间对比

###### 删除

1. 删除老的(老的里面存在，新的里面不存在)

```javascript

// a b (c d) f g
// a b (e c) f g
// D 节点在新的里面没有需要删除
// C 节点 props 也发生了变化
const prevChildren = [
	h('p', { key: 'A' }, 'A'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'C', id: 'c-prev' }, 'C'),
	h('p', { key: 'D' }, 'D'),
	h('p', { key: 'F' }, 'F'),
	h('p', { key: 'G' }, 'G'),
]
const nextChildren = [
	h('p', { key: 'A' }, 'A'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'E' }, 'E'),
	h('p', { key: 'C', id: 'c-next' }, 'C'),
	h('p', { key: 'F' }, 'F'),
	h('p', { key: 'G' }, 'G'),
]

```


2. 删除 优化


```javascript


// a b (c e d) f g
// a b (e c) f g
// 当所有的新的节点都对比完了，老节点还存在元素，这些元素都可以被删除
const prevChildren = [
	h('p', { key: 'A' }, 'A'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'C', id: 'c-prev' }, 'C'),
	h('p', { key: 'E' }, 'E'),
	h('p', { key: 'D' }, 'D'),
	h('p', { key: 'Z' }, 'Z'),
	h('p', { key: 'X' }, 'X'),
	h('p', { key: 'F' }, 'F'),
	h('p', { key: 'G' }, 'G'),
]
const nextChildren = [
	h('p', { key: 'A' }, 'A'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'E' }, 'E'),
	h('p', { key: 'C', id: 'c-next' }, 'C'),
	h('p', { key: 'F' }, 'F'),
	h('p', { key: 'G' }, 'G'),
]


```


###### 移动

```javascript

// a b (c d e) f g
// a b (e c d) f g
// 只需要移动e
const prevChildren = [
	h('p', { key: 'A' }, 'A'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'C' }, 'C'),
	h('p', { key: 'D' }, 'D'),
	h('p', { key: 'E' }, 'E'),
	h('p', { key: 'F' }, 'F'),
	h('p', { key: 'G' }, 'G'),
]
const nextChildren = [
	h('p', { key: 'A' }, 'A'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'E' }, 'E'),
	h('p', { key: 'C' }, 'C'),
	h('p', { key: 'D' }, 'D'),
	h('p', { key: 'F' }, 'F'),
	h('p', { key: 'G' }, 'G'),
]


```


### resolve

#### array -> text

1. 删除旧节点中的children
2. 渲染text  setElementText

patch -> processElement -> patchElement

```javascript

// runtime-core/renderer.ts

function patchELement(n1, n2, container){
	...

	const el = (n2.el = n1.el)

	patchChildren(n1, n2, el)
	
	patchProps(...)
}


function patchChildren(n1, n2, container){
	const { children: c1, shapeFlag: prevShapeFlag } = n1;
	const { children: c2, shapeFlag: nextShapeFlag } = n2;

	if(prevShapeFlag & ShapeFlags.ARRAY_CHILDREN){
		if(nextShapeFlag & ShapeFlags.TEXT_CHILDREN){
			// array -> text
			// 1. 删除旧节点中的children
			unmountChildren(c1)

			// 2. 渲染text
			hostSetElementText(container, c2)
		}
	}


}


function unmountChildren(children){
	children.forEach({el}=>{
		hostRemove(el)
	})
}



// runtime-dom/index.ts


function remove(el){
	const parent = el.parentNode
	if(parent){
		parent.removeChild(el)
	}
}


function setElementText(container, text){
	container.textContent = text;
}

```



#### text -> text

直接调用`hostSetElementText`替换即可

```javascript

// text -> text

hostSetElementText(container, c2)

```


#### text -> array
1. 清空原来的内容
2. mount Array
	1. 数组中是vnode，这里需要mount


这里需要改两点
1. mountChildren原来接收第一个参数为vnode，需要改为children
2. mountChildren需要接收参数parentComponent，在patchChildren这里需要添加

```javascript


if(prevShapeFlag & ShapeFlags.ARRAY_CHILDREN){
	if(nextShapeFlag & ShapeFlags.ARRAY_CHILDREN){
		// text -> array
		// 1. 清空旧节点的内容
		hostSetElementText(container, '')

		// 2. mountChildren
		mountChildren(c2, container, parentComponent)
	}
}


```




#### array -> array

> 双端对比（diff）
> -> 找出乱序部分


##### 左侧对比


```javascript

// 1. 左侧的对比
// (a b) c
// (a b) d e
const prevChildren = [
	h('p', { key: 'A' }, 'A'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'C' }, 'C'),
]
const nextChildren = [
	h('p', { key: 'A' }, 'A'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'D' }, 'D'),
	h('p', { key: 'E' }, 'E'),
]

// init: i = 0, e1 = 2, e2 = 3
// 左端对比: i = 2, e1 = 2, e2 = 3

```


##### 右端对比

```

```




##### 新的比老的长 创建新的

```javascript

// 新的比老的长
// i > e1 && i <= e2



```

这里insert使用的是append，只能在后边添加，如果在左侧有新元素则会添加失败，他会添加到末尾，这里需要对insert方法进行重新修改，使用insertBefore


```javascript

// runtime-dom/index.ts

function insert(el, container, anchor){
	container.insertBefore(el, anchor = null)
}



// runtime-core/renderer.ts
// 实现和不一样，但感觉没啥影响

for (let j = e2; j >= i; j--) {
	const anchor = c2[e2 + 1].el || null
	patch(null, c2[j], container, parentComponent, anchor)
	e2--
}

// 原
const nextPos = e2 + 1;
const anchor = nextPos < l2 ? c2[nextPos].el : null;
while (i <= e2) {
  patch(null, c2[i], container, parentComponent, anchor);
  i++;
}


```


##### 老的比新的长 删除
```
// 老的比新的长
// i > e2 && i <= e1












```



##### 中间对比

`i <= e1 && i <= e2`

###### 删除

1. 删除老的(老的里面存在，新的里面不存在)

> 1. 遍历新节点创建`newIndexMap`
> 2. 遍历老节点 通过 `key` + `newIndexMap` 查询在新节点中的`newIndex`
> 	1. key 如果不存在，则需要遍历新节点(时间复杂度 O(n))
> 	2. key 存在 则直接去newIndexMap中获取newIndex(时间复杂度 O(1))
> 3. 获取到newIndex后进行判断
> 	1. newIndex 为 undefined 则说明当前旧节点在新的节点中不存在，执行remove
> 	2. newIndex 有值，则执行patch（这里有两种情况，一种需要移动，一种不需要可以直接patch）



```javascript

// 删除老的(老的里面存在，新的里面不存在)
// a b (c d) f g
// a b (e c) f g
// D 节点在新的里面没有需要删除
// C 节点 props 也发生了变化
const prevChildren = [
	h('p', { key: 'A' }, 'A'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'C', id: 'c-prev' }, 'C'),
	h('p', { key: 'D' }, 'D'),
	h('p', { key: 'F' }, 'F'),
	h('p', { key: 'G' }, 'G'),
]
const nextChildren = [
	h('p', { key: 'A' }, 'A'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'E' }, 'E'),
	h('p', { key: 'C', id: 'c-next' }, 'C'),
	h('p', { key: 'F' }, 'F'),
	h('p', { key: 'G' }, 'G'),
]



```


2. 删除 优化（patched, toBePatched）

> 这里是一种特殊情况，如果当前新旧节点都对比完了(例如下面的CE节点)
> 此时老节点中还存在一些节点（DZX）
> 这些节点并不需要去获取newIndex，可以直接进行remove

```javascript


// a b (c e d) f g
// a b (e c) f g
// 当所有的新的节点都对比完了，老节点还存在元素，这些元素都可以被删除
const prevChildren = [
	h('p', { key: 'A' }, 'A'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'C', id: 'c-prev' }, 'C'),
	h('p', { key: 'E' }, 'E'),
	h('p', { key: 'D' }, 'D'),
	h('p', { key: 'Z' }, 'Z'),
	h('p', { key: 'X' }, 'X'),
	h('p', { key: 'F' }, 'F'),
	h('p', { key: 'G' }, 'G'),
]
const nextChildren = [
	h('p', { key: 'A' }, 'A'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'E' }, 'E'),
	h('p', { key: 'C', id: 'c-next' }, 'C'),
	h('p', { key: 'F' }, 'F'),
	h('p', { key: 'G' }, 'G'),
]


```


###### 移动

> 寻找最长递增子序列 这些是不需要改变的，除了这个子序列之外的是不稳定的元素，需要进行移动 
> 算法： [[最长递增子序列]]


```javascript

// a b (c d e) f g
// a b (e c d) f g
// 只需要移动e
const prevChildren = [
	h('p', { key: 'A' }, 'A'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'C' }, 'C'),
	h('p', { key: 'D' }, 'D'),
	h('p', { key: 'E' }, 'E'),
	h('p', { key: 'F' }, 'F'),
	h('p', { key: 'G' }, 'G'),
]
const nextChildren = [
	h('p', { key: 'A' }, 'A'),
	h('p', { key: 'B' }, 'B'),
	h('p', { key: 'E' }, 'E'),
	h('p', { key: 'C' }, 'C'),
	h('p', { key: 'D' }, 'D'),
	h('p', { key: 'F' }, 'F'),
	h('p', { key: 'G' }, 'G'),
]



// c1: a b (c d e) f g
// c2: a b (e c d) f g

// init i = 0, e1 = 6, e2 = 6 // toBePatched = 3, patched = 0
// 左端对比 i = 2, e1 = 6, e2 = 6
// 右端对比 i = 2, e1 = 4, e2 = 4

// for c2 -> newIndexMap: { E: 2, C: 3, D: 4 }

// toBePatched => newIndexToOldIndexMap: [0, 0, 0]

/** 
for c1 -> 
	s1: 2 -> C 
		-> newIndexMap[C] => 3 
		-> newIndexToOldIndexMap[newIndex - s2] = s1 + 1
		=> newIndexToOldIndexMap[1] = 3
	s1: 3 -> D
		-> newIndexMap[D] => 4
		=> newIndexToOldIndexMap[2] = 4
	s1: 4 -> E 
		-> newIndexMap[E] => 2
		=> newIndexToOldIndexMap[0] = 5

*/

// newIndexToOldIndexMap: [5, 3, 4]

// sequence = getSequence(newIndexToOldIndexMap) => [1, 2] (最长递增子序列)

// noNeedToMoveIdx = sequence.length - 1 => 1

// for c2 -> 
// newIndexToOldIndexMap中值为0的话，表示该节点需要新增，在老节点中并没有这个的坐标

/** 
c1: a b (c d e) f g
c2: a b (e c d) f g
		[5 3 4]

sequence => [1,2]

for c2 倒着遍历 -> 
	s2: 4 -> D
		-> index = e2 - s2 = 2
		-> sequence[noNeedToMoveIdx] => 2 === index
		-> 不需要处理
		-> noNeedToMoveIdx-- => noNeedToMoveIdx = 0
	s2: 3 -> C
		-> index = e2 - s2 = 1
		-> sequence[noNeedToMoveIdx] => 1 === index
		-> 不需要处理
		-> noNeedToMoveIdx-- => noNeedToMoveIdx = -1
	s2: 2 -> E
		-> noNeedToMoveIdx < 0
		-> patch

*/


```










```



/**
c1: a b (c d e) f g
c2: a b (e c d) f g

i: 2
s1: 2
s2: 2
e1: 4
e2: 4

newIndexToOldIndexMap: [0, 0, 0]

需要确定在旧节点中哪几个不需要进行移动

这里不需要移动的是cd，相对位置是没有改变的 => 找出最长递增子序列

for c1 -> 
C -> newIndexMap[C] => 3
D -> newIndexMap[D] => 4
D -> newIndexMap[E] => 2
这里是说明C在新节点中的坐标为3，D的newIndex为4
s2: 2

s1: 2
newIndexToOldIndexMap[C newIndex - s2] = s1 + 1
== newIndexToOldIndexMap[1] = 3

newIndexToOldIndexMap: [0, 3, 0]

s1: 3
newIndexToOldIndexMap[D newIndex - s2] = s1 + 1
== newIndexToOldIndexMap[2] = 4

newIndexToOldIndexMap: [0, 3, 4]

s1: 4
newIndexToOldIndexMap[E newIndex - s2] = s1 + 1
== newIndexToOldIndexMap[0] = 5

newIndexToOldIndexMap: [5, 3, 4]
// 如果看成 [4, 2, 3] => 
// 这里的相对位置就是新节点的位置

c1: a b (c d e) f g
c2: a b (e c d) f g
		[4 2 3]
		对应旧节点的位置（但0是初始，所以这里都+1）


sequence = getSequence(newIndexToOldIndexMap) => [1, 2]


for c2 [s2 -> e2] 
s2: 2 -> 
s2: 3 -> 
s2: 4 -> 


*/



```


### 源码实现



1. sync from start
2. sync from end
	1. 确定中间乱序部分  --- i, e1, e2
3. common sequence + mount
	1. 新的比老的长 新增 (a b) -> c (a b) / (a b) c --- anchor
4. common sequence + unmount
	1. 老的比新的长 删除 c (a b) / (a b) c -> (a b)
5. unknown sequence
	4. build key:index map for newChildren --- keyToNewIndexMap
	5. loop through old children left to be patched and try to patch
		1. newIndexToOldIndexMap --- used for determining longest stable subsequence
		2. patched, toBePatched --- optimize: all new children have been patched, the rest can be deleted
		3. moved, maxNewIndexSoFar --- used to track whether any node has moved
	6. move and mount --- increasingNewIndexSequence




```javascript


// 1. sync from start
// (a b) c
// (a b) d e
while (i <= e1 && i <= e2) {
	const n1 = c1[i]
	const n2 = c2[i]
	if (isSameVNodeType(n1, n2)) {
		patch(n1, n2, ...)
	} else {
		break
	}
	i++
}



// 2. sync from end
// a (b c)
// d e (b c)
while (i <= e1 && i <= e2) {
	const n1 = c1[e1]
	const n2 = c2[e2]
	if (isSameVNodeType(n1, n2)) {
		patch(n1, n2, ...)
	} else {
		break
	}
	e1--
	e2--
}


// 3. common sequence + mount
// (a b)
// (a b) c
// i = 2, e1 = 1, e2 = 2
// (a b)
// c (a b)
// i = 0, e1 = -1, e2 = 0
if (i > e1) {
	if (i <= e2) {
		const nextPos = e2 + 1
		const anchor = nextPos < l2 ? (c2[nextPos] as VNode).el : parentAnchor
		while (i <= e2) {
			patch(null, c2[i], container, anchor, ...)
			i++
		}
	}
}



// 4. common sequence + unmount
// (a b) c
// (a b)
// i = 2, e1 = 2, e2 = 1
// a (b c)
// (b c)
// i = 0, e1 = 0, e2 = -1
else if (i > e2) {
	while (i <= e1) {
		unmount(c1[i], parentComponent, parentSuspense, true)
		i++
	}
}



// 5. unknown sequence
// [i ... e1 + 1]: a b [c d e] f g
// [i ... e2 + 1]: a b [e d c h] f g
// i = 2, e1 = 4, e2 = 5
else {
    const s1 = i // prev starting index
    const s2 = i // next starting index


	// 5.1 build key:index map for newChildren
	const keyToNewIndexMap: Map<string | number | symbol, number> = new Map()
	for (i = s2; i <= e2; i++) {
		const nextChild = c2[i]
		if (nextChild.key != null) {
			if (keyToNewIndexMap.has(nextChild.key)) {
				warn(
					`Duplicate keys found during update:`,
					JSON.stringify(nextChild.key),
					`Make sure keys are unique.`
				)
			}
			keyToNewIndexMap.set(nextChild.key, i)
		}
	}


	// 5.2 loop through old children left to be patched and try to patch
	// matching nodes & remove nodes that are no longer present
	let j
	let patched = 0
	const toBePatched = e2 - s2 + 1
	let moved = false
	// used to track whether any node has moved
	let maxNewIndexSoFar = 0
	// works as Map<newIndex, oldIndex>
	// Note that oldIndex is offset by +1
	// and oldIndex = 0 is a special value indicating the new node has
	// no corresponding old node.
	// used for determining longest stable subsequence
	const newIndexToOldIndexMap = new Array(toBePatched)
	for (i = 0; i < toBePatched; i++)
		newIndexToOldIndexMap[i] = 0
	  
	for (i = s1; i <= e1; i++) {
		const prevChild = c1[i]
		if (patched >= toBePatched) {
			// all new children have been patched so this can only be a removal
			unmount(prevChild, parentComponent, parentSuspense, true)
			continue
		}
		let newIndex
		if (prevChild.key != null) {
			newIndex = keyToNewIndexMap.get(prevChild.key)
		} else {
			// key-less node, try to locate a key-less node of the same type
			for (j = s2; j <= e2; j++) {
				if (
					newIndexToOldIndexMap[j - s2] === 0 &&
					isSameVNodeType(prevChild, c2[j] as VNode)
				) {
					newIndex = j
					break
				}
			}
		}
		if (newIndex === undefined) {
			unmount(prevChild, parentComponent, parentSuspense, true)
		} else {
			newIndexToOldIndexMap[newIndex - s2] = i + 1
			if (newIndex >= maxNewIndexSoFar) {
				maxNewIndexSoFar = newIndex
			} else {
				moved = true
			}
			patch(prevChild, c2[newIndex] as VNode, container, null, ...)
			patched++
		}
	}


	// 5.3 move and mount
	// generate longest stable subsequence only when nodes have moved
	const increasingNewIndexSequence = moved
		? getSequence(newIndexToOldIndexMap)
		: EMPTY_ARR
	j = increasingNewIndexSequence.length - 1
	// looping backwards so that we can use last patched node as anchor
	for (i = toBePatched - 1; i >= 0; i--) {
		const nextIndex = s2 + i
		const nextChild = c2[nextIndex] as VNode
		const anchor =
			nextIndex + 1 < l2 ? (c2[nextIndex + 1] as VNode).el : parentAnchor
		if (newIndexToOldIndexMap[i] === 0) {
			// mount new
			patch(null, nextChild, container, anchor, ...)
		} else if (moved) {
			// move if:
			// There is no stable subsequence (e.g. a reverse)
			// OR current node is not among the stable sequence
			if (j < 0 || i !== increasingNewIndexSequence[j]) {
				move(nextChild, container, anchor, MoveType.REORDER)
			} else {
				j--
			}
		}
	}

}



```



























## 问题

runtime-dom/index.ts function patchProp prevVal并没有用到？

runtime-core/renderer.ts function patchProps 但这里不去判断EMPTY_OBJ感觉也可以，for in访问一个空的对象，也不会执行内部逻辑























