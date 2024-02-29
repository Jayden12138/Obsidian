# happy path
## UT

```javascript
it('happy path', () => {
	const user = reactive({ age: 1 })
	const age = computed(() => user.age)
	expect(age.value).toBe(1)
})

```


computed 在使用上类似 ref，都需要通过.value来访问值，但 computed 有缓存

.value => value
通过创建一个类，来实现
```javascript
class ComputedRefImpl{
	constructor(){}
	get value(){}
}

```


# lazily
## UT
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

## 分步
#### 1 should not compute again
```javascript
// should not compute again
cValue.value
expect(getter).toHaveBeenCalledTimes(1)
```

第一次执行 getter 获取到 value 后，需要一个“塞子”，在之后触发 get 时，直接返回第一次执行时的 value

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
#### 2 should not compute until needed
```javascript
// should not compute until needed
value.foo = 2  // trigger
expect(getter).toHaveBeenCalledTimes(1)
```

1. set -> trigger targetMap
依赖的响应式对象中的值发生了改变，触发 trigger，需要去执行 dep 中收集到的依赖，但是现在 targetMap 并没有初始化，没有 effect

class ComputedRefImpl 添加一个 effect，这里 ActiveEffect 在 createGetter 时，会执行 track，对target，key =》targetMap 进行初始化，并收集依赖

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


2. 触发 set 后，并没有执行 getter

scheduler

triggerEffect 中，会获取到 dep，遍历 dep 执行 effect.run()，如果 effect.scheduler不为空，则会执行 scheduler

```javascript

class ComputedRefImpl{
	constructor(getter){
		this._effect = new ActiveEffect(getter, ()=>{
			if(!flag)flag = true
		})
	}
}

```

修改响应式对象的值
触发 set -> trigger -> triggerEffect 执行 scheduler -> init flag
当下一次触发 get value
flag == true => 重新执行 effect.run()去获取到响应式对象的 newValue

#### 3 now it should compute
```javascript
// now it should compute
expect(cValue.value).toBe(2)
expect(getter).toHaveBeenCalledTimes(2)
```

2 中最后说的，下一次触发 get，这里重新调用了 getter 1 => 2

