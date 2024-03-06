当前渲染函数如下

```javascript
function mountElement(vnode, container, parentComponent) {
	const el = (vnode.el = document.createElement(vnode.type))

	// string | array
	const { children, shapeFlag } = vnode
	if (shapeFlag & ShapeFlags.TEXT_CHILDREN) {
		el.textContent = children
	} else if (shapeFlag & ShapeFlags.ARRAY_CHILDREN) {
		mountChildren(vnode, el, parentComponent)
	}

	// props
	const { props } = vnode
	for (const key in props) {
		const val = props[key]
		if (isOn(key)) {
			const event = key.slice(2).toLowerCase()
			el.addEventListener(event, val)
		} else {
			el.setAttribute(key, val)
		}
	}

	container.append(el)
}

```

提出关键操作

```javascript
// DOM 平台
// -- DOM API

// 创建元素 createElement
const el = (vnode.el = document.createElement(vnode.type))

// 设置属性 patchProp
el.setAttribute(key, val)

// 添加到容器上 insert
container.append(el)


```


```javascript

// runtime-core/renderer.ts

// before (DOM)
mountElement
	const el = (vnode.el = document.createElement(vnode.type))
	el.setAttribute(key, val)
	container.append(el)


// createRenderer(options)
// options: { createElement, patchProp, insert }

// renderer.ts

export function createRenderer(options){
	const { createElement, patchProp, insert } = options

	function render(){}
	
	function patch(){}

	function processElement(){}

	...

	function mountElement(){
		// 这里所有DOM API的操作需要改为options中传入的方法
		
		// createElement - 创建元素
		
		// patchProp - 处理props
		
		// insert - 插入
	}

	return {
		createApp: createAppAPI(render)
	}

}




// App.js
import { createApp } from '../../lib/guide-mini-vue.esm.js'
import { App } from './App.js'

const rootContainer = document.querySelector('#app')
createApp(App).mount(rootContainer)


// runtime-core/createApp.ts

// before
// 一开始，main.js中所调用的createApp，是runtime-core/createApp.ts导出的
// 这里的render也是强依赖于DOM API
export function createApp(rootComponent){
	return {
		mount(rootContainer){
			const vnode = createVNode(rootComponent)

			render(vnode, rootContainer)
		}
	}
}
// 因为render方法是`customRenderer`中内部的方法，这里通过包装一层，将render作为参数进行传入

// after
export function createAppAPI(render){

	return function createApp(rootComponent){
		return {
			mount(rootContainer){
				const vnode = createVNode(rootComponent)

				render(vnode, rootContainer)
			}
		}
	}

}


// runtime-dom/index.ts

function createElement(){}

function patchProp(){}

function insert(){}

const render = createRenderer({
	createElement,
	patchProp,
	insert
}) // return { createApp: createAppAPI(render) }

export function createApp(...args){
	return render.createApp(...args)
}
// 这里重新定义一个createApp, 进行了一次转发，这样在main.js中就可以直接使用createApp()，而不是render.createApp()







```


```javascript

// vue => runtime-dom => runtime-core => reactivity



```















