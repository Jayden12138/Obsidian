## case

```javascript

describe('readonly', () => {
	it('happy path', () => {
		const original = { foo: 1 }
		const observed = readonly(original)
		expect(observed).not.toBe(original)
		expect(observed.foo).toBe(1)
	})

	it('should not allow reassignment', () => {
		console.warn = jest.fn()
		const original = { foo: 1 }
		const observed = readonly(original)
		observed.foo = 2
		expect(console.warn).toHaveBeenCalled()
	})
})

```


## thinking

1. 和reactive差不多，都是通过代理对象来包裹一层，区别是readonly不需要进行依赖收集，因为不允许进行set操作
2. 这里对之前实现的reactive需要进行重构，将相同的逻辑以及较为底层的实现抽离出来


```javascript

// reactive.ts 旧
function reactive(raw){
	return new Proxy(raw, {
		get(target, key){
			const res = Reflect.get(target, key)

			track(target, key)

			return res
		},
		set(target, key, value){
			const res = Reflect.set(target, key, value)

			trigger(target, key, value)

			return res
		}
	})
}

// reactive.ts 新 添加了readonly

/**

	这里readonly 相较于reactive的实现，大体一致，只有部分有调整
	并且在reactive和readonly中，都是使用了Proxy并直接返回（调用Proxy算偏底层的实现，这里需要将其抽离出来）

*/
function readonly(raw){
	return new Proxy(raw, {
		get(target, key){
			const res = Reflect.get(target, key)

			// track(target, key)

			return res
		},
		set(target, key, value){
			// const res = Reflect.set(target, key, value)

			// trigger(target, key, value)

			return true
		}
	})
}

// baseHandlers.ts
const get = createGetter()
const set = createSetter()
const readonlyGet = createGetter(true)

function createGetter(isReadonly = false){
	return (target, key) => {
		const res = Reflect.get(target, key)

		!isReadonly && track(target, key)

		return res
	}
}

function createSetter(){
	return (target, key, value) => {
		const res = Reflect.set(target, key, value)

		trigger(target, key, value)

		return res
	}
}

const mutableHandlers = {
	get,
	set
}

const readonlyHandlers = {
	get: readonlyGet,
	set: (target, key, value)=>{
		console.warn('wrong')
		return true
	}
}

// reactive.ts 
function reactive(raw){
	return createActiveObject(raw, mutableHandlers)
}

function readonly(raw){
	return createActiveObject(raw, readonlyHandlers)
}

function createActiveObject(raw, baseHandlers){
	return new Proxy(raw, baseHandlers)
}


```



## code

[readonly - Jayden Github](https://github.com/Jayden12138/tiny-vue/pull/24)

```javascript


// baseHandlers.ts
const get = createGetter()
const set = createSetter()
const readonlyGet = createGetter(true)

function createGetter(isReadonly = false){
	return (target, key) => {
		const res = Reflect.get(target, key)

		!isReadonly && track(target, key)

		return res
	}
}

function createSetter(){
	return (target, key, value) => {
		const res = Reflect.set(target, key, value)

		trigger(target, key, value)

		return res
	}
}

const mutableHandlers = {
	get,
	set
}

const readonlyHandlers = {
	get: readonlyGet,
	set: (target, key, value)=>{
		console.warn('wrong')
		return true
	}
}

// reactive.ts 
function reactive(raw){
	return createActiveObject(raw, mutableHandlers)
}

function readonly(raw){
	return createActiveObject(raw, readonlyHandlers)
}

function createActiveObject(raw, baseHandlers){
	return new Proxy(raw, baseHandlers)
}


```