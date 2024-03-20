## UT
```javascript
 it('proxyRefs', () => {
    const user = {
      age: ref(10),
      name: 'xiaohong'
    }

    const proxyUser = proxyRefs(user)
    expect(user.age.value).toBe(10)
    expect(proxyUser.age).toBe(10)
    expect(proxyUser.name).toBe('xiaohong')

    proxyUser.age = 20
    expect(proxyUser.age).toBe(20)
    expect(user.age.value).toBe(20)

    proxyUser.age = ref(10)
    expect(proxyUser.age).toBe(10)
    expect(user.age.value).toBe(10)
  })
```


## thinking

```

get

ref 返回 ref.value
非ref 直接返回

（注意：这里逻辑和unRef一致，直接复用unRef即可）


set

// oldVal ref / newVal 非ref 给ref.value赋值

// 直接赋值
// oldVal ref / newVal ref
// oldVal 非ref / newVal ref
// oldVal 非ref / newVal 非ref





```


## code


```js

export function proxyRefs(ref) {
	return new Proxy(ref, {
		get(target, key) {
			return unRef(Reflect.get(target, key))
		},
		set(target, key, value) {
			const oldVal = Reflect.get(target, key)
			if (isRef(oldVal) && !isRef(value)) {
				return (target[key].value = value)
			} else {
				return Reflect.set(target, key, value)
			}
		},
	})
}


```




## source code

```js


/**
 * Returns a reactive proxy for the given object.
 *
 * If the object already is reactive, it's returned as-is. If not, a new
 * reactive proxy is created. Direct child properties that are refs are properly
 * handled, as well.
 *
 * @param objectWithRefs - Either an already-reactive object or a simple object
 * that contains refs.
 */
export function proxyRefs<T extends object>(
	objectWithRefs: T
): ShallowUnwrapRef<T> {
	return isReactive(objectWithRefs)
		? objectWithRefs
		: new Proxy(objectWithRefs, shallowUnwrapHandlers)
}

const shallowUnwrapHandlers: ProxyHandler<any> = {
	get: (target, key, receiver) => unref(Reflect.get(target, key, receiver)),
	set: (target, key, value, receiver) => {
		const oldValue = target[key]
		if (isRef(oldValue) && !isRef(value)) {
			oldValue.value = value
			return true
		} else {
			return Reflect.set(target, key, value, receiver)
		}
	},
}



```


```js

const obj = {}

original = reactive(obj)

proxyRefs(original)

/**

// reactive
new Proxy(obj, {

	get(){
		key === '__v_isReactive' return xx
	}

})

class x

// proxyRefs

isReactive(raw)
raw['__v_isReactive'] -> proxy get





*/










```