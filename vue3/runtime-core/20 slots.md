

## 实现cases


### 1.  基础展示

```javascript

// 1. 基础展示

// App.js
import { h } from '../../lib/guide-mini-vue.esm.js'
import { Foo } from './Foo.js'
export const App = {
	name: 'App',
	render() {
		const app = h('div', {}, 'App')

		// 单个p标签 slots传入
		const foo = h(Foo, {}, h('p', {}, '123'))

		return h('div', {}, [app, foo])
	},
	setup() {},
}

// Foo.js
import { h } from '../../lib/guide-mini-vue.esm.js'
export const Foo = {
	name: 'Foo',
	setup() {
		return {}
	},
	render() {
		const foo = h('p', {}, 'foo')
		
		return h('div', {}, [foo, this.$slots])
	},
}








```


### 2. 处理数组slots

```javascript

// 2. 处理数组slots 添加renderSlots

// App.js
import { h } from '../../lib/guide-mini-vue.esm.js'
import { Foo } from './Foo.js'
export const App = {
	name: 'App',
	render() {
		const app = h('div', {}, 'App')

		// 数组 slots传入
		const foo = h(Foo, {}, [h('p', {}, '123'), h('p', {}, '456')])

		return h('div', {}, [app, foo])
	},
	setup() {},
}

// Foo.js
import { h, renderSlots } from '../../lib/guide-mini-vue.esm.js'
export const Foo = {
	name: 'Foo',
	setup() {
		return {}
	},
	render() {
		const foo = h('p', {}, 'foo')
		
		return h('div', {}, [
			foo,
			renderSlots(this.$slots)
		])
	},
}






```



### 3. 具名slots

```javascript

// 3. 具名slots array -> object


// App.js
import { h } from '../../lib/guide-mini-vue.esm.js'
import { Foo } from './Foo.js'
export const App = {
	name: 'App',
	render() {
		const app = h('div', {}, 'App')

		// object
		const foo = h(
			Foo,
			{},
			{
				header: h('p', {}, 'header'),
				footer: h('p', {}, 'footer'),
			}
		)

		return h('div', {}, [app, foo])
	},
	setup() {},
}

// Foo.js
import { h, renderSlots } from '../../lib/guide-mini-vue.esm.js'
export const Foo = {
	name: 'Foo',
	setup() {
		return {}
	},
	render() {
		const foo = h('p', {}, 'foo')

		return h('div', {}, [
			renderSlots(this.$slots, 'header'),
			foo,
			renderSlots(this.$slots, 'footer'),
		])
	},
}




```


### 4. 作用域slots

```javascript

// 4. 作用域slots


// App.js
import { h } from '../../lib/guide-mini-vue.esm.js'
import { Foo } from './Foo.js'
export const App = {
	name: 'App',
	render() {
		const app = h('div', {}, 'App')

		// object
		const foo = h(
			Foo,
			{},
			{
				header: ({ age }) => h('p', {}, 'header - ' + age),
				footer: () => h('p', {}, 'footer'),
			}
		)

		return h('div', {}, [app, foo])
	},
	setup() {},
}

// Foo.js
import { h, renderSlots } from '../../lib/guide-mini-vue.esm.js'
export const Foo = {
	name: 'Foo',
	setup() {
		return {}
	},
	render() {
		const foo = h('p', {}, 'foo')
		const age = 1
		return h('div', {}, [
			renderSlots(this.$slots, 'header', { age }),
			foo,
			renderSlots(this.$slots, 'footer'),
		])
	},
}


```


### 5. 添加SLOTS_CHILDREN

