
```js

- 问题

在 04 中使用 requestIdleCallback 将渲染dom的操作放在了浏览器的空闲时间中进行，虽然不会有明显的卡顿，但还是存在一些问题

在渲染复杂DOM过程中，存在一段时间内，浏览器并没有空闲时间，那么可预见的会发生：渲染了部分后，卡顿了一小会后，又开始渲染

- 解决

统一commit

function performWorkOfUnit(work){
	if(!work.dom){
		const dom = (work.dom = createDom(work.type))
		work.parent.dom.append(dom)
	}
}

当前代码是依次执行渲染，完成一个则进行渲染
work.parent.dom.append(dom)

这里可以等待所有的vdom都渲染完成，最后进行提交一次

- code















```