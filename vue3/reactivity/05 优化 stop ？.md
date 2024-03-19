
## case

```javascript
it('stop obj.prop++', () => {
	let dummy
	const obj = reactive({ prop: 1 })
	const runner = effect(() => {
		dummy = obj.prop
	})
	obj.prop = 2
	expect(dummy).toBe(2)
	stop(runner)
	// obj.prop = 3
	obj.prop++ // obj.prop = obj.prop + 1
	expect(dummy).toBe(2)
	
	// stopped effect should still be manually callable
	runner()
	expect(dummy).toBe(3)

})
```


## thinking


### 1. 处理 obj.prop++ 

```javascript

obj.prop++ // => obj.prop = obj.prop + 1

```




```javascript

let shouldTrack


function track(target, key){
	if(!isTracking()) return
	
	...
}


function isTracking(){
	return shouldTrack && activeEffect !== undefined
}


// shouldTrack 在哪里定义


/**

	obj.prop++ // obj.prop = obj.prop + 1
	
	obj.prop
		=> getter
		=> track
	obj.prop = x + 1
		=> setter
		=> trigger
			find dep by target and key
			then "for dep run effect.run"
				effect.run() ==> fn()

	effect.run 时会收集依赖

*/


class ReactiveEffect{
	_fn
		
	...

	run(){
		if(!this.active){
			// 当执行stop后，就不进行依赖收集
			return this._fn()
		}

		// 需要进行依赖收集，前期准备
		shouldTrack = true
		activeEffect = this

		const res = this.fn()

		// reset
		shouldTrack = false

		return res
	}
}



```

### 优化点

1. dep.add前，对 activeEffect 进行判断是否 has，如果有了就不进行重复的依赖收集


## question

1. shouldTrack 定义的位置/run重构