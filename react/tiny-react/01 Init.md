
看得见的思考

```js

// 1. hello world
const app = document.querySelector('#root')
const dom = document.createElement('div')
dom.id = 'app'
app.append(dom)

const textEl = document.createTextNode('')
textEl.nodeValue = 'hello world'
dom.append(textEl)



```





```js


// 2. react -> vdom -> js object
const TextEl = {
	type: 'TEXT_ELEMENT',
	props: {
		nodeValue: 'hello world',
		children: [],
	},
}

const el = {
	type: 'div',
	props: {
		id: 'app',
		children: [textEl],
	},
}

const app = document.querySelector('#root')

// const dom = document.createElement('div')
const dom = document.createElement(el.type)
// dom.id = 'app'
dom.id = el.props.id
app.append(dom)

const textEl = document.createTextNode('')
// textEl.nodeValue = 'hello world'
textEl.nodeValue = TextEl.props.nodeValue
dom.append(textEl)


```



```js


/**
 * 3.
 *
 * 上面两个逻辑类似
 * 封装函数进行重构（createElement createTextNode）
 *
 * render 递归处理vdom
 *
 * type props children
 *
 * */

const TextEl = {
	type: 'TEXT_ELEMENT',
	props: {
		nodeValue: 'hello world',
		children: [],
	},
}

const el = {
	type: 'div',
	props: {
		id: 'app',
		children: [TextEl],
	},
}

function createTextNode(text) {
	return {
		type: 'TEXT_ELEMENT',
		props: {
			nodeValue: text,
			children: [],
		},
	}
}

function createElement(type, props, ...children) {
	return {
		type,
		props: {
			...props,
			children: children.map(child =>
				typeof child === 'string' ? createTextNode(child) : child
			),
		},
	}
}

function render(el, container) {
	// dom
	const dom =
		el.type === 'TEXT_ELEMENT'
			? document.createTextNode('')
			: document.createElement(el.type)

	// props
	Object.keys(el.props).forEach(key => {
		if (key !== 'children') {
			dom[key] = el.props[key]
		}
	})

	// children
	const children = el.props.children
	if (children) {
		children.forEach(child => render(child, dom))
	}

	// append
	container.append(dom)
}

const app = document.querySelector('#root')
render(el, app)


```




```js





/**
 *
 *
 * ReactDOM.createRoot(document.getElementById('root')).render(
	<React.StrictMode>
		<App />
	</React.StrictMode>
)
    参考react调用形式，对现有代码进行封装
 *
 *
 *
 */

function createTextNode(text) {
	return {
		type: 'TEXT_ELEMENT',
		props: {
			nodeValue: text,
			children: [],
		},
	}
}

function createElement(type, props, ...children) {
	return {
		type,
		props: {
			...props,
			children: children.map(child =>
				typeof child === 'string' ? createTextNode(child) : child
			),
		},
	}
}

function render(el, container) {
	// dom
	const dom =
		el.type === 'TEXT_ELEMENT'
			? document.createTextNode('')
			: document.createElement(el.type)

	// props
	Object.keys(el.props).forEach(key => {
		if (key !== 'children') {
			dom[key] = el.props[key]
		}
	})

	// children
	const children = el.props.children
	if (children) {
		children.forEach(child => render(child, dom))
	}

	// append
	container.append(dom)
}

const ReactDOM = {
	createRoot(container) {
		return {
			render(el) {
				render(el, container)
			},
		}
	},
}

// const app = document.querySelector('#root')
// render(el, app)
const el = createElement('div', { id: 'app' }, 'hello world')
ReactDOM.createRoot(document.getElementById('root')).render(el)





```





```js


/**
 * 
 * 
 * ReactDOM.createRoot(document.getElementById('root')).render(
	<React.StrictMode>
		<App />
	</React.StrictMode>
)
    参考react调用形式，对现有代码进行封装
 * 
 * 
 * 
 */

// main.js
import ReactDOM from './core/ReactDOM.js'
import App from './App.js'

ReactDOM.createRoot(document.getElementById('root')).render(App)


// App.js
import React from './core/React.js'

const App = React.createElement('div', { id: 'app' }, 'hello world')

export default App


// core/React.js
function createTextNode(text) {
	return {
		type: 'TEXT_ELEMENT',
		props: {
			nodeValue: text,
			children: [],
		},
	}
}

function createElement(type, props, ...children) {
	return {
		type,
		props: {
			...props,
			children: children.map(child =>
				typeof child === 'string' ? createTextNode(child) : child
			),
		},
	}
}

function render(el, container) {
	// dom
	const dom =
		el.type === 'TEXT_ELEMENT'
			? document.createTextNode('')
			: document.createElement(el.type)

	// props
	Object.keys(el.props).forEach(key => {
		if (key !== 'children') {
			dom[key] = el.props[key]
		}
	})

	// children
	const children = el.props.children
	if (children) {
		children.forEach(child => render(child, dom))
	}

	// append
	container.append(dom)
}

export default {
	createElement,
	render,
}



// core/ReactDOM.js
import React from './React.js'
const ReactDOM = {
	createRoot(container) {
		return {
			render(el) {
				React.render(el, container)
			},
		}
	},
}

export default ReactDOM





```