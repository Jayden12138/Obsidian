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




```javascript


















```













































## 问题

runtime-dom/index.ts function patchProp prevVal并没有用到？

runtime-core/renderer.ts function patchProps 但这里不去判断EMPTY_OBJ感觉也可以，for in访问一个空的对象，也不会执行内部逻辑























