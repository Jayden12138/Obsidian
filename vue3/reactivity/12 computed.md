
> computed 在使用上类似 ref，都需要通过.value来访问值，但 computed 有缓存
## case

```javascript

describe('computed', () => {
	it('happy path', () => {
		const user = reactive({ age: 1 })
		const age = computed(() => user.age)
		expect(age.value).toBe(1)
	})

	it('should compute lazily', () => {
		const value = reactive({ foo: 1 })
		const getter = jest.fn(() => value.foo)
		const cValue = computed(getter)

		// lazy
		expect(getter).not.toHaveBeenCalled()

		expect(cValue.value).toBe(1)
		expect(getter).toHaveBeenCalledTimes(1)

		// should not compute again
		cValue.value
		expect(getter).toHaveBeenCalledTimes(1)

		// should not compute until needed
		value.foo = 2 // trigger
		expect(getter).toHaveBeenCalledTimes(1)

		// now it should compute
		expect(cValue.value).toBe(2)
		expect(getter).toHaveBeenCalledTimes(2)

		// should not compute again
		cValue.value
		expect(getter).toHaveBeenCalledTimes(2)
	})
})



```

## thinking

```javascript

1. happy path
2. lazy
3. shoud not compute again
4. shoud not compute until needed
5. now it should compute
6. shoud not compute again



1. happy path

	it('happy path', () => {
		const user = reactive({ age: 1 })
		const age = computed(() => user.age)
		expect(age.value).toBe(1)
	})
	
	a. computed接收一个参数getter
	b. computed的返回值类似ref的用法，可以通过.value来访问到getter的返回值

	tip: class

2. lazy

	it('should compute lazily', () => {
		const value = reactive({ foo: 1 })
		const getter = jest.fn(() => value.foo)
		const cValue = computed(getter)

		// lazy
		expect(getter).not.toHaveBeenCalled()

		expect(cValue.value).toBe(1)
		expect(getter).toHaveBeenCalledTimes(1)
	})


	a. 在并没有触发get前，getter不会调用执行
	b. 当触发了get，getter才会执行

	tip: get value(){ ... }

3. shoud not compute again


	it('should compute lazily', () => {
		const value = reactive({ foo: 1 })
		const getter = jest.fn(() => value.foo)
		const cValue = computed(getter)

		// lazy
		expect(getter).not.toHaveBeenCalled()

		expect(cValue.value).toBe(1)
		expect(getter).toHaveBeenCalledTimes(1)

		// should not compute again
		cValue.value
		expect(getter).toHaveBeenCalledTimes(1)
	})

	a. 当执行过一次getter后，再次触发get，并不会执行getter

	tip: cache


4. shoud not compute until needed

	it('should compute lazily', () => {
		const value = reactive({ foo: 1 })
		const getter = jest.fn(() => value.foo)
		const cValue = computed(getter)

		// lazy
		expect(getter).not.toHaveBeenCalled()

		expect(cValue.value).toBe(1)
		expect(getter).toHaveBeenCalledTimes(1)

		// should not compute again
		cValue.value
		expect(getter).toHaveBeenCalledTimes(1)

		// should not compute until needed
		value.foo = 2 // trigger
		expect(getter).toHaveBeenCalledTimes(1)
	})

	a. 触发set，并不会执行getter
	b. 但是set会触发trigger
	
	trigger -> dep -> effect.run -> getter x scheduler ✔

	tip: effect scheduler


5. now it should compute

	it('should compute lazily', () => {
		const value = reactive({ foo: 1 })
		const getter = jest.fn(() => value.foo)
		const cValue = computed(getter)

		// lazy
		expect(getter).not.toHaveBeenCalled()

		expect(cValue.value).toBe(1)
		expect(getter).toHaveBeenCalledTimes(1)

		// should not compute again
		cValue.value
		expect(getter).toHaveBeenCalledTimes(1)

		// should not compute until needed
		value.foo = 2 // trigger
		expect(getter).toHaveBeenCalledTimes(1)

		// now it should compute
		expect(cValue.value).toBe(2)
		expect(getter).toHaveBeenCalledTimes(2)
	})

	a. 当执行过set后，再次触发get会执行getter，以获取到新值
	
	tip: effect scheduler


6. shoud not compute again


	it('should compute lazily', () => {
		const value = reactive({ foo: 1 })
		const getter = jest.fn(() => value.foo)
		const cValue = computed(getter)

		// lazy
		expect(getter).not.toHaveBeenCalled()

		expect(cValue.value).toBe(1)
		expect(getter).toHaveBeenCalledTimes(1)

		// should not compute again
		cValue.value
		expect(getter).toHaveBeenCalledTimes(1)

		// should not compute until needed
		value.foo = 2 // trigger
		expect(getter).toHaveBeenCalledTimes(1)

		// now it should compute
		expect(cValue.value).toBe(2)
		expect(getter).toHaveBeenCalledTimes(2)

		// should not compute again
		cValue.value
		expect(getter).toHaveBeenCalledTimes(2)
	})

	a. 这之前触发过get，所以这里再次触发get并不会执行getter



```



## code

#### happy path

```javascript

it('happy path', () => {
	const user = reactive({ age: 1 })
	const age = computed(() => user.age)
	expect(age.value).toBe(1)
})

```


```javascript

class ComputedRefImpl{
	constructor(){}
	get value(){}
}

```


####  lazily

```javascript

it('should compute lazily', () => {
	const value = reactive({ foo: 1 })
	const getter = jest.fn(() => value.foo)
	const cValue = computed(getter)

	// lazy
	expect(getter).not.toHaveBeenCalled()

	expect(cValue.value).toBe(1)
	expect(getter).toHaveBeenCalledTimes(1)

	// should not compute again
	cValue.value
	expect(getter).toHaveBeenCalledTimes(1)

	// should not compute until needed
	value.foo = 2  // trigger
	expect(getter).toHaveBeenCalledTimes(1)

	// now it should compute
	expect(cValue.value).toBe(2)
	expect(getter).toHaveBeenCalledTimes(2)

	// should not compute again
	cValue.value
	expect(getter).toHaveBeenCalledTimes(2)
})
```

##### 1. should not compute again

```javascript

// should not compute again
cValue.value
expect(getter).toHaveBeenCalledTimes(1)

```


```javascript
// init
flag = true
value

// get
get value(){
	if(flag){
		flag = false;
		value = this.getter()
	}
	return value;
}

```
##### 2. should not compute until needed

```javascript

// should not compute until needed
value.foo = 2  // trigger
expect(getter).toHaveBeenCalledTimes(1)

```

```


1. set -> trigger targetMap
依赖的响应式对象中的值发生了改变，触发 trigger，需要去执行 dep 中收集到的依赖，但是现在 targetMap 并没有初始化，没有 effect

class ComputedRefImpl 添加一个 effect，这里 ActiveEffect 在 createGetter 时，会执行 track，对target key => targetMap 进行初始化，并收集依赖



```

```javascript

class ComputedRefImpl{
	constructor(getter){
		this._effect = new ActiveEffect(getter)
	}

	get value(){
		this._value = this._effect.run()
	}
}

```

```


2. 触发 set 后，并没有执行 getter

scheduler

triggerEffect 中，会获取到 dep，遍历 dep 执行 effect.run()，如果 effect.scheduler不为空，则会执行 scheduler



```

```javascript

class ComputedRefImpl{
	constructor(getter){
		this._effect = new ActiveEffect(getter, ()=>{
			if(!flag)flag = true
		})
	}
}

```

```


修改响应式对象的值
触发 set -> trigger -> triggerEffect 执行 scheduler -> init flag

当下一次触发 get value
flag == true => 重新执行 effect.run()去获取到响应式对象的 newValue



```

##### 3. now it should compute

```javascript

// now it should compute
expect(cValue.value).toBe(2)
expect(getter).toHaveBeenCalledTimes(2)

```

2 中最后说的，下一次触发 get，这里重新调用了 getter 1 => 2

