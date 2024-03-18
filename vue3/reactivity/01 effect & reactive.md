

## cases

```javascript
describe('reactive', () => {
    it('happy path', () => {
        const original = { foo: 1 }
        const observed = reactive(original)
        expect(observed).not.toBe(original)
        expect(observed.foo).toBe(1)
    })

    it('should observe basic properties', () => {
        let dummy
        const obj = reactive({ prop: 1 })
        effect(() => { dummy = obj.prop })
        expect(dummy).toBe(1)
        obj.prop = 2
        expect(dummy).toBe(2)
    });
})
```

## thinking

1. reactive需要接收一个变量
2. 当变量中的值发生get、set等操作需要被拦截（Proxy）
3. effect接收一个函数fn，首次会执行fn
4. 当fn中的响应式对象中的值发生改变，也会执行fn

```

get track 收集依赖
set trigger 触发依赖

targetMap -> depsMap -> dep
targetMap{target -> depsMap{key -> dep}}

```

## code

 >[commit: effect & reactive -- Jayden Github](https://github.com/Jayden12138/tiny-vue/commit/0f1d85545d0f85ab05b65fdece3eeae869bca4f7)

```javascript

// reactive.ts
function reactive(raw){
	return new Proxy(raw, {
		get(target, key){
			const res = Reflect.get(target, key)
			
			track(target, key)
			
			return res
		},
		set(target, key, value){
			const res = Reflect.set(target, key, value)

			trigger(target, key)
		
			return res
		}
	})
}

// effect.ts
const targetMap = new Map()
let activeEffect
class ReactiveEffect{
	private _fn
	constructor(fn){
		this._fn = fn
	}

	run(){
		activeEffect = this
		this._fn()
	}
}

function effect(fn){
	let _effect = new ReactiveEffect(fn)

	_effect.run()
}

function track(target, key){
	const depsMap = targetMap.get(target)

	if(!depsMap){
		depsMap = new Map()
		targetMap.set(target, depsMap)
	}
	
	const dep = depsMap.get(key)

	if(!dep){
		dep = new Set()
		depsMap.set(key, dep)
	}

	dep.add(activeEffect)
}

function trigger(target, key){
	const depsMap = targetMap.get(target)
	const dep = depsMap.get(key)

	for(const effect of dep){
		effect.run()
	}
}



```



## source code

[ reactive - core](https://github1s.com/vuejs/core/blob/main/packages/reactivity/src/reactive.ts#L77)

```javascript

/**
 * Returns a reactive proxy of the object.
 *
 * The reactive conversion is "deep": it affects all nested properties. A
 * reactive object also deeply unwraps any properties that are refs while
 * maintaining reactivity.
 *
 * @example
 * ```js
 * const obj = reactive({ count: 0 })
 * ```
 *
 * @param target - The source object.
 * @see {@link https://vuejs.org/api/reactivity-core.html#reactive}
 */
export function reactive<T extends object>(target: T): UnwrapNestedRefs<T>
export function reactive(target: object) {
  // if trying to observe a readonly proxy, return the readonly version.
  if (isReadonly(target)) {
    return target
  }
  return createReactiveObject(
    target,
    false,
    mutableHandlers,
    mutableCollectionHandlers,
    reactiveMap,
  )
}



```

## question





## others

> 1. [[Proxy]]
> 2. [[Reflect]]