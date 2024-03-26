[深入了解requestIdleCallback - juejin](https://juejin.cn/post/7033959714794766372)
[requestIdleCallback - w3c](https://w3c.github.io/requestidlecallback/)


[Scheduler - react](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/scheduler/src/Scheduler.js)


```js

- 问题
// 渲染复杂DOM，现在的render会卡顿

- 思路

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


// 当前的render处理children时，是通过递归实现，并且在最后
// container.append(dom)
// 将每一个创建的dom每次遍历都渲染到了当前的视图上,多次重绘


// 这里引入一个API requestIdleCallback


/**
requestIdleCallback

这个函数将在浏览器空闲时期被调用

IdleDeadline.timeRemaining() 可以获取到剩余空闲时间（毫秒）
*/



function callback(IdleDeadline){
	let shouldYield = false

	while(!shouldYield){
		console.log(IdleDeadline.timeRemaining())

		shouldYield = IdleDeadline.timeRemaining() < 1
	}

	requestIdleCallback(callback)
}

requestIdleCallback(callback)


- 通过上述实现的callback，对render进行重构

1. 先对当前的render进行 初步的重构（通过注释，将功能进行封装）


2. 树 转 链表

每次只执行一个任务，再看剩余时间是否足够运行下一个任务
依次进行的任务 -> 链表

child
sibiling
	需要在处理children时，给child上绑定个sibling的属性
uncle(.parent.sibling)
	需要通过 .parent.sibling 进行访问
	parent


// 执行
function preformWorkOfUnit(work){
	// 处理dom
	// 处理props

	// 处理children （生成链表）
	let prevChild = null
	children.forEach((child, index)=>{
	    const newWork = {
		    type: child.type,
			props: child.props,
			dom: null,
			parent: work,
			child: null,
			sibling: null,
	    }
	    
	    if(index === 0){
			work.child = newWork
	    } else {
	        prevChild.sibling = newWork
	    }
	    prevChild = newWork
	})
	
	
	// 返回下一个需要执行的task
	if(work.children){return work.children}
	
	if(work.sibling){return work.sibling}
	
	return work.parent.sibling


}




```



```js


- 问题
当出现当前节点没有兄弟节点，其父节点也没有兄弟节点，但在其祖先节点上是存在兄弟节点的
按之前的代码，并不会渲染这个

- test case
<div id="app">
	<div>
		<div>1-1</div>
	</div>
	<div>11</div>
</div>




// 执行
function preformWorkOfUnit(work){
	...
	
	// 返回下一个需要执行的task
	if(work.children){return work.children}
	
	// if(work.sibling){return work.sibling}
	
	// return work.parent.sibling

	let nextWork = work
	while(nextWork){
		if(nextWork.sibling){
			return nextWork.sibling
		}else{
			nextWork = nextWork.parent
		}
	}
}







```





