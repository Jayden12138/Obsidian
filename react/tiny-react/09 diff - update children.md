

```js

// type 不一致
// 删除旧的 创建新的



```



![[Pasted image 20240328102456.png]]
当处理完bar后，new处理完毕，但此时old节点中还存在foo/bar 的sibling未处理

```js

let deleteArr = []

children.forEach(child=>{
	...
})

while(oldFiber){
	// 这里oldFiber就是没有处理的兄弟节点
	deleteArr.push(oldFiber)

	oldFiber = oldFiber.sibling
}


```




```js

- edge case


let showBar = false
function Counter() {
	const bar = <div>bar</div>
	function toggleShow() {
		showBar = !showBar
		CReact.update()
	}
	return (
		<div>
			Counter
			<button onClick={toggleShow}>toggleShow</button>
			{showBar && bar}
		</div>
	)
}


节点false时，不需要处理


- edge case2

注意sibling指向



```

![[Pasted image 20240328113841.png]]



优化更新逻辑

>减少不必要的更新

```js


需要找到 开始 - 结束
开始：当前更新的组件
结束：当前更新组件的兄弟节点



更新 countFoo，在调用update时可以获取到 “开始”

结尾：
	当前“根组件”为“开始”
	判断当前“根组件”的兄弟节点 等于 下一个任务的type，则为结尾
如果是，则不执行下一个任务，直接跳出即可


function workLoop(deadline) {
	let shouldYield = false

	while (!shouldYield && nextWork) {
		nextWork = performWorkOfUnit(nextWork)

		if (root?.sibling?.type === nextWork?.type) {
			// 如果是则跳出循环
			nextWork = undefined
		}

		shouldYield = deadline.timeRemaining() < 1
	}

	if (!nextWork && root) {
		commitRoot()
	}

	requestIdleCallback(workLoop)
}


ps：react中优化 尽可能拆分 function component ?




```

