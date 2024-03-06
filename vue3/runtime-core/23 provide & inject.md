
## 实现case


### 单层 存取

```javascript

import { h, provide, inject } from '../../lib/mini-vue.esm.js'

const Provider = {
	name: 'Provider',
	setup() {
		provide('foo', 'fooVal')
		provide('bar', 'barVal')
	},
	render() {
		return h('div', {}, [h('p', {}, 'Provider'), h(Consumer)])
	},
}

const Consumer = {
	name: 'Consumer',
	setup() {
		const foo = inject('foo')
		const bar = inject('bar')
		return {
			foo,
			bar,
		}
	},
	render() {
		return h('div', {}, [
			h('p', {}, `foo: ${this.foo}`),
			h('p', {}, `bar: ${this.bar}`),
		])
	},
}

export const App = {
	name: 'App',
	setup() {},
	render() {
		return h('div', {}, [h('p', {}, 'app'), h(Provider)])
	},
}


```

实际上是一个存取的过程（provide存，inject取）
`getCurrentInstance`

> provide在父组件中调用，传入key value，这里可以存入instance.provides={}，加上之前实现`getCurrentInstance`，在子级组件中调用getCurrentInstance并通过parent属性向上访问到父级的instance，进而访问到provides，通过key来获取到对应的值

```javascript


// component.ts

function createComponentInstance(instance, ..., parent){
	const instance = {
		type,
		...,
		provides: {},
		parent: parent
	}
}


// apiInject.ts
function provide(key, value){
	const instance = getCurrentInstance()

	if(instance){
		const { provides } = instance

		provides[key] = value
	}
}


function inject(key){
	const instance = getCurrentInstance()

	if(instance){
		const { provides } = instance.parent

		return provides[key]
	}
}


```


### 中间加一层

```javascript

// App.js
const Provider = {
	name: 'Provider',
	setup() {
		provide('foo', 'fooVal')
		provide('bar', 'barVal')
	},
	render() {
		return h('div', {}, [h('p', {}, 'Provider'), h(ProviderTwo)])
	},
}

const ProviderTwo = {
	name: 'ProviderTwo',
	setup() {},
	render() {
		return h('div', {}, [h('p', {}, 'ProviderTwo'), h(Consumer)])
	},
}


```

在子组件初始化调用`createComponentInstance`时，将parent.provides传到子组件的provides上即可

如果在子组件中也调用了`provide()`，则会对当前instance上 的provides进行修改

```javascript

function createComponentInstance(..., parent){
	const instance = {
		provides: parent ? parent.provides : {}
		parent: parent
	}
}

```



```javascript

// A > B > C
// A 

// A
instance.provides: {}

// provide('foo', 'fooVal')
// provide('bar', 'barVal')

instance.provides: {
	'foo': 'fooVal',
	'bar': 'barVal'
}


// B
// instance.provides = A.instance.provides
instance.provides: {
	'foo': 'fooVal',
	'bar': 'barVal'
}

// provide('foo', 'fooTwo')

instance.provides: {
	'foo': 'fooTwo',
	'bar': 'barVal'
}

// C
// instance.provides = B.instance.provides
instance.provides: {
	'foo': 'fooTwo',
	'bar': 'barVal'
}

// inject('foo') => instance.parent.provides.foo => 'fooTwo'


```

### Object.create


```javascript

const ProviderTwo = {
	name: 'ProviderTwo',
	setup() {
		provide('foo', 'fooTwo')
		const foo = inject('foo')
		return {
			foo,
		}
	},
	render() {
		return h('div', {}, [
			h('p', {}, 'ProviderTwo - ' + this.foo),
			h(Consumer),
		])
	},
}

```

在B组件中，通过`inject('foo')`，预期获取到A上`provide('foo', 'fooVal')`的值，但是实际上显示获取到的是`'fooTwo'`

```javascript

// B
// instance.provides = A.instance.provides
instance.provides: {
	'foo': 'fooVal',
	'bar': 'barVal'
}

// provide('foo', 'fooTwo')

instance.provides: {
	'foo': 'fooTwo',
	'bar': 'barVal'
}

// inject('foo') => B.instance.parent.provides => 'fooTwo'

```

这里通过`Object.create`将父级的provides作为prototype绑定在子组件的provides上

```javascript

// apiInject.ts function provide()
function provide(key, value){
	const instance =  getCurrentInstance()

	if(instance){
		let { provides } = instance
		const parentProvides = instance.parent?.provides

		if(provides === parentProvides){
			// init
			provides = instance.provides = Object.create(parentProvides)
		}

		provides[key] = value
	}
}

```


```javascript


// A > B > C
// A 

// A
instance.provides: {} // createComponentInstance()

// provide('foo', 'fooVal')
// provide('bar', 'barVal')

instance.provides: {
	'foo': 'fooVal',
	'bar': 'barVal'
}


// B
// instance.provides = A.instance.provides // createComponentInstance()
instance.provides: {
	'foo': 'fooVal',
	'bar': 'barVal'
}

// provide('foo', 'fooTwo') 
// B.instance.provides === parentProvides ?
// B.instance.provides = Object.create(parentProvides)
instance.provides: {
	prototype: A.instance.provides
}

intsance.provides: {
	'foo': 'fooTwo',
	prototype: A.instance.provides
}

// inject('foo') => B.instance.parent.provides => 'fooVal'

// C
// instance.provides = B.instance.provides
instance.provides: {
	'foo': 'fooTwo',
	prototype: A.instance.provides
}

// inject('foo') => C.instance.parent.provides.foo => 'fooTwo'


// inject('bar') 
// => C.instance.parent.provides.bar 
// => B.instance.provides.bar 
// => B.instance.provides.prototype.bar => 'barVal'


```


### defaultValue
#### base type

```javascript

const Consumer = {
	name: 'Consumer',
	setup() {
		const foo = inject('foo')
		const bar = inject('bar')
		const baz = inject('baz', 'defaultBaz')
		return {
			foo,
			bar,
			baz,
		}
	},
	render() {
		return h('div', {}, [
			h('p', {}, `foo: ${this.foo}`),
			h('p', {}, `bar: ${this.bar}`),
			h('p', {}, `baz: ${this.baz}`),
		])
	},
}

```

1. inject需要接收第二个参数defaultValue
2. 当key不在provides上且不在原型链上则需要返回defaultValue

```javascript

function inject(key, defaultValue){

	const instance = getCurrentInstance()

	if(instance){
		const { provides } = instance

		if(key in provides){
			return provides[key]
		}else{
			return defaultValue
		}
	}

}

```

这里使用到了in操作符，具体跳转[[in|in 操作符]]，in可以判断当前key是否在一个对象上或在原型链上


#### function

```javascript

const Consumer = {
	name: 'Consumer',
	setup() {
		const baz = inject('baz', 'defaultBaz')
		const bazFunc = inject('bazFunc', () => 'defaultBazFunc')
		return {
			baz,
			bazFunc,
		}
	},
	render() {
		return h('div', {}, [
			h('p', {}, `baz: ${this.baz}`),
			h('p', {}, `bazFunc: ${this.bazFunc}`),
		])
	},
}

```


这块直接在inject中进行判断，如果是function，则调用把返回值return回去，如果不是则直接返回

```javascript

// apiInject.ts inject
function inject(key, defaultValue){

	const instance = getCurrentInstance()

	if(instance){
		const { provides } = instance.parent

		if(key in provides){
			return provides[key]
		}else{
			if(typeof defaultValue === 'function'){
				return defaultValue()
			}else{
				return defaultValue		
			}
		}
	}

}

```

