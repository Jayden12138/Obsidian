

```js

- 问题

1. 如何得到新的dom树
2. 如何找到老节点
3. 如果 diff props


- 思路

1. 如何得到新的dom树

// 重新执行 render 就可以得到新的dom 树

// 这里需要将初始的根节点保存下来，作为对比需要
// 无法直接使用root，因为在 统一提交 时，将root置为null了

let currentRoot = null

function commitRoot(){
	commitWork(root.child)
	currentRoot = root
	root = null
}


function update(){
	root = {
		dom: currentRoot.dom,
		props: currentRoot.props
	}
}

2. 如何找到老节点

function update(){
	root = {
		dom: currentRoot.dom,
		props: currentRoot.props,
		alternate: currentRoot
	}
}

在 update 中将老节点通过 alternate 进行保存


在initChildren中

let oldChild = work.alternate?.child

const isSameType = oldChild?.type === child.type

let newWork = null
if (isSameType) {
	newWork = {
		type: child.type,
		props: child.props,
		dom: oldChild.dom, // 初始创建过一次，这里不需要重复创建
		parent: work,
		child: null,
		sibling: null,
		tag: 'update',
		alternate: oldChild,
	}
} else {
	...
}

这里也是一样 通过 alternate 来记录老节点
tag 用于区分 当前执行的类型


// React.ts initChildren
if (oldChild) {
	oldChild = oldChild.sibling
}

在 children.forEach 遍历第一次执行（index == 0）
为 可能存在的兄弟节点 将 oldChild 指针指向他
oldChild = oldChild.sibling

4. 如何 diff props

重构 updateProps 

/**

1. new 存在 old 不存在 删除
2. new 存在 old 存在 更新
3. new 不存在 old 存在 新增

*/

function updateProps(dom, nextProps, prevProps){
	// 1. new 存在 old 不存在 删除
	
	Object.keys(prevProps).forEach(key => {
		if (!(key in nextProps)) {
			dom.removeAttribute(key)
		}
	})
	
	// 2. new 存在 old 存在 更新
	// 3. new 不存在 old 存在 新增
	
	Object.keys(nextProps).forEach(key => {
		if (key !== 'children') {
			if (key.startsWith('on')) {
				// onClick => click
				const eventName = key.slice(2).toLocaleLowerCase()
				dom.removeEventListener(eventName, prevProps[key]) // 重复绑定事件，需要移除， 这里是 prevProps[key]
				dom.addEventListener(eventName, nextProps[key])
			} else {
				dom[key] = nextProps[key]
			}
		}
	})
}

4. 最后需要提交

function commitWork(work){
	...

	// 这里只处理了创建的逻辑，并没有update
	if (work.dom) {
		workParent.dom.append(work.dom)
	}
	
	...
}

function commitWork(work){
	...

	// 根据 tag 区分 当前操作的类型
	// update 只需要 updateProps
	if (work.tag === 'placement') {
		if (work.dom) {
			workParent.dom.append(work.dom)
		}
	} else if (work.tag === 'update') {
		updateProps(work.dom, work.props, work.alternate.props)
	}

}
















```
