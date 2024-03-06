 添加这两种类型
`patch`时进行判断，当前vnode需要如何process

```javascript

const Fragment = Symbol('Fragment')
const Text = Symbol('Text')

switch(type){
	case Fragment:
		processFragment();
		break;
	case Text:
		processText();
		break;
	default:
		// element | component
		processElement() | processComponent();
		break;
}


```


在processFragment，因为没有外层包裹，只需要处理`children`即可，所以这里直接调用`mountChildren()`进行处理

在`processText`中，需要通过`document.createTextNode(text)`，来创建text节点
在`render`函数中，`textNode`需要通过特定的函数进行生成`TextVNode`
因为`render`函数中，需要虚拟节点，但h需要接受一个`type`，而这里`textNode`并没有`type`
调用`createTextVNode(xxx)`=>`createVNode(Text, {}, xxx)`，会生成一个vnode，type为text，在patch时可以捕获并进行后续的处理
```javascript


// renderer.ts
function processText(vnode, container){
	const { children } = vnode;
	const textNode = document.createTextNode(children)
	container.append(textNode)
}


// App.js / Foo.js
render(){
	const App = h('div', {}, 'App')
	const text = createTextVNode('text')

	return h('div', {}, [App, text])
}


// vnode.ts
function createTextVNode(text){
	return createVNode(Text, {}, text)
}


```