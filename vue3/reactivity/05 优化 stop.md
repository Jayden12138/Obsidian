>优化点：
>1. 边缘 case：处理 obj.prop++ 
>2. dep.add前，对 activeEffect 进行判断是否 has，如果有了就不进行重复的依赖收集
>3. cleanupEffect 中，delete 后 数组清空（length = 0）
## UT 
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


## 1. 边缘 case：处理 obj.prop++ 
```javascript
obj.prop++
// 等同于 obj.prop = obj.prop + 1
```
会先触发 get，get -> track -> 依赖收集
后触发 set，从 targetMap 中取到 dep，遍历执行 effect.run()

```javascript
stop(runner)

/**
stop函数接受一个参数 runner
在 runner.effect中存了当前的 activeEffect，进而通过 runner.effect.stop()调用 ActiveEffect 中定义的 stop 方法
在 stop 方法中需要获取到相关的 dep，遍历 dep 删除所有收集到的依赖，并设置 active 标识当前已执行过 stop，之后在 run 方法中，如果执行过 stop 则不进行依赖收集，dep.forEach(i=>o.delete(this))
这里为了能获取到 dep
在get依赖收集时，对 dep 反向收集，dep.add(activeEffect)
activeEffect.deps.push(dep)
这样之后可以直接通过this.deps获取到dep
*/
```
