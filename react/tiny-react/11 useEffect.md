
```js

// 1


function Foo() {
	console.log('re foo')
	const [count, setCount] = CReact.useState(10)
	function handleClick() {
		setCount(c => c + 1)
	}

	CReact.useEffect(() => {
		console.log('init')
	}, [])

	return (
		<div>
			foo
			<p>count: {count}</p>
			<button onClick={handleClick}>click</button>
		</div>
	)
}


function commitRoot() {
	deleteArr.forEach(deleteOldNode)
	commitWork(root.child)
	commitEffectHook()
	currentRoot = root
	root = null
	deleteArr = []
}

function commitEffectHook(){
	function run(fiber){
		if(!fiber){
			return
		}
		fiber.effectHook?.callback()
		run(fiber.child)
		run(fiber.sibling)
	}

	run(wipFiber)
}

function useEffect(callback, deps){
	const effectHook = {
		deps,
		callback
	}

	wipFiber.effectHook = effectHook
}








// 2

	CReact.useEffect(() => {
		console.log('init')
	}, [count])

function commitEffectHook(){
	function run(fiber){
		if(!fiber){
			return
		}
		if(!fiber.alternate){
			// init
			fiber.effectHook?.callback()
		}else{
			// update
			// deps 是否改变
			const oldEffectHook = fiber.alternate?.effectHook

			// some
			const needUpdate = oldEffectHook?.deps.some((oldDep, index)=>{
				return oldDep !== fiber.effectHook.deps[index]
			})

			needUpdate && fiber.effectHook?.callback()

		}
		run(fiber.child)
		run(fiber.sibling)
	}

	run(wipFiber)
}



// 3


	CReact.useEffect(() => {
		console.log('init')
	}, [])

	CReact.useEffect(() => {
		console.log('update count: ', count)
	}, [count])


function commitEffectHooks(){
	function run(fiber){
		if(!fiber){
			return
		}
		if(!fiber.alternate){
			// init
			fiber.effectHooks?.forEach(hook=>{
				hook.callback()
			})
		}else{
			// update
			// deps 是否改变
			fiber.effectHooks?.forEach((newHook, index)=>{
				const oldEffectHook = fiber.alternate?.effectHooks[index]

				// some
				const needUpdate = oldEffectHook?.deps.some((oldDep, i)=>{
					return oldDep !== newHook.deps[i]
				})
	
				needUpdate && newHook?.callback()
			})

		}
		run(fiber.child)
		run(fiber.sibling)
	}

	run(wipFiber)
}


let effectHooks
function useEffect(callback, deps){
	const effectHook = {
		deps,
		callback
	}

	effectHooks.push(effectHook)

	wipFiber.effectHooks = effectHooks
}

















```



```js

// cleanup
// 1. 清除副作用
// 2. 如果deps.length为0，则不会执行cleanup







```