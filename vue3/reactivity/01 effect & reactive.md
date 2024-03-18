

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



## question





## others

> 1. [[Proxy]]
> 2. [[Reflect]]