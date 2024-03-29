

```js


// App.jsx
function Foo() {
	const [count, setCount] = CReact.useState(10)
	function handleClick() {
		setCount((c) => c + 1)
	}
	return (
		<div>
			foo
			{count}
			<button onClick={handleClick}>click</button>
		</div>
	)
}



```



```js




function useState(initial) {
	console.log('useState: ', initial)
	let currentFiber = wipFiber
	let stateHook = {
		state: currentFiber?.stateHook?.state || initial,
	}
	function setState(setter) {
		stateHook.state = setter(stateHook.state)

		currentFiber.stateHook = stateHook
		nextWork = {
			...currentFiber,
			alternate: currentFiber,
		}
		root = nextWork
	}

	return [stateHook.state, setState]
}




```

```js


多个

function Foo() {
	const [count, setCount] = CReact.useState(10)
	const [bar, setBar] = CReact.useState('bar')
	function handleClick() {
		setCount(c => c + 1)
		setBar(b => b + ' bar')
	}
	return (
		<div>
			foo
			<p>count: {count}</p>
			<p>bar: {bar}</p>
			<button onClick={handleClick}>click</button>
		</div>
	)
}

数组

let stateHooks = []
let stateHookIndex = 0
function useState(initial) {
	let currentFiber = wipFiber
	let oldHook = currentFiber.alternate?.stateHooks[stateHookIndex]
	let stateHook = {
		state: oldHook ? oldHook.state : initial,
	}

	stateHookIndex++
	stateHooks.push(stateHook)

	currentFiber.stateHooks = stateHooks

	function setState(setter) {
		stateHook.state = setter(stateHook.state)

		nextWork = {
			...currentFiber,
			alternate: currentFiber,
		}
		root = nextWork
	}

	return [stateHook.state, setState]
}



//reset

function updateFunctionComponent(work) {
	stateHookIndex = 0
	stateHooks = []
	wipFiber = work
	const children = [work.type(work.props)]

	initChildren(work, children)
}





```



>hook不能在if语句中去写，要去function最外层调用
>需要 顺序





优化1: 批次处理
优化2: 提前检测，减少不必要的更新

```js


批量处理


let stateHooks = []
let stateHookIndex = 0
function useState(initial) {
	let currentFiber = wipFiber
	let oldHook = currentFiber.alternate?.stateHooks[stateHookIndex]
	let stateHook = {
		state: oldHook ? oldHook.state : initial,
		queue: oldHook ? oldHook.queue : [],
	}

	stateHook.queue.forEach(action => {
		stateHook.state = action(stateHook.state)
	})

	stateHook.queue = []

	stateHookIndex++
	stateHooks.push(stateHook)

	currentFiber.stateHooks = stateHooks

	function setState(setter) {
		const isFunction = typeof setter === 'function'

		const eagerState = isFunction ? setter(stateHook.state) : setter

		if (eagerState === stateHook.state) return

		stateHook.queue.push(isFunction ? setter : () => setter)

		nextWork = {
			...currentFiber,
			alternate: currentFiber,
		}
		root = nextWork
	}

	return [stateHook.state, setState]
}






```




```

// https://react.dev/reference/react/useState

 
示例1
// https://react.dev/reference/react/useState#text-field-string

使用onChange有个问题，只有失去焦点时才会触发handleChange事件
































```


