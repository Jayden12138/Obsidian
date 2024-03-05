>1. 需要给setup传入一个对象，对象中emit
>2. 调用emit时，可以传入一个事件名
>	1. on + Event => 当前组件props中是否有对应的函数，找到后直接调用







## others

### 1. [[Function.prototype.bind]]

>这里使用到了[[Function.prototype.bind#Partial functions - 部分函数 | function bind - Partial function]]
>通过Partial function，使emit实际使用时，不需要传入instance，直接传入event以及args即可
>ps：emit需要instance是需要通过访问instance.props来查找event对应的事件，父组件在使用子组件时，通过props传入的方法

```javascript

// 路径

const foo = h(Foo, {
	onClick(){}
})

function h(type, props?, children) => createVnode(...)

function createVNode(type, props, children) => vnode

vnode = {
	type,
	props,
	children,
	...
}

function emit(instance, event, ...args){}
// emit('add', 1, 2)
// 'add' -> 'onAdd' -> instance.props['onAdd'](...args)
// 'add-foo' -> 'addFoo' -> 'onAddFoo'


// render

mount
createVNode(rootComponent)
render(vnode, rootContainer)
patch(vnode, container)
<component | element> // component
processComponent(vnode, container)
<mount | update> // mount
mountComponent(vnode, container) 
// 1. createComponentInstance 2. setupComponent 3. setupRenderEffect
instance = createComponentInstance(initialVNode) // 创建组件实例

instance = {
	vnode,
	type: vnode.type,
	props: {},
	slots: {},
	emit: () => {}
}
instance.emit = emit.bind(null, instance)  (***)

setupComponent(instance)
// 1. initProps 2. initSlots 3. setupStatefulComponent
initProps() // instance.props = instance.vnode.props
initSlots() // instance.slots = instance.vnode.children
setupStatefulComponent()




```

```javascript
// function createComponentInstance

component.emit = emit.bind(null, component)

// 使用emit时，无需传入第一个参数instance


// componentEmit.ts

function emit(instance, event, ...args){}

// 这里emit方法中，需要通过instance.props来查找emit传入的方法

```
