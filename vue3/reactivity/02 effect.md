## 1. runner
### case

```javascript
 it('should return runner when call effect', () => {
        let foo = 10
        const runner = effect(() => {
            foo++
            return 'foo'
        })

        expect(foo).toBe(11)
        const r = runner()
        expect(foo).toBe(12)
        expect(r).toBe('foo')
    })
```
### thinking

需要实现三点
1. effect函数会返回一个runner函数
2. 当调用runner函数，会执行effect传入的方法
3. runner的返回值为effect传入方法的返回值

> this 指向问题: [[Function.prototype.bind]]



## 2. scheduler
### case

```javascript
it('scheduler', () => {
        let dummy;
        let run: any;
        const scheduler = vi.fn(() => {
          run = runner;
        });
        const obj = reactive({ foo: 1 });
        const runner = effect(
          () => {
            dummy = obj.foo;
          },
          { scheduler }
        );
        expect(scheduler).not.toHaveBeenCalled();
        expect(dummy).toBe(1);
        // should be called on first trigger
        obj.foo++;
        expect(scheduler).toHaveBeenCalledTimes(1);
        // should not run yet
        expect(dummy).toBe(1);
        // manually run
        run();
        // should have run
        expect(dummy).toBe(2);
});
```
### thinking

需要实现的点：
1. 通过 effect 的第二个参数 传入scheduler
2. effect 第一次正常执行 fn
3. 当响应式数据发生 set 时（trigger），执行 scheduler

## 3. stop
### case

```javascript
it('stop', () => {
    let dummy;
    const obj = reactive({ prop: 1 });
    const runner = effect(() => {
      dummy = obj.prop;
    });
    obj.prop = 2;
    expect(dummy).toBe(2);
    stop(runner);
    obj.prop = 3;
    expect(dummy).toBe(2);

    // stopped effect should still be manually callable
    runner();
    expect(dummy).toBe(3);
  });
```
### thinking

1. stop接收runner函数
2. 在stop中，需要获取到该runner相关的effect收集到的所有依赖，将这些依赖删除即可
	1. 后续在执行set -> trigger => 当遍历deps时，此时所有依赖已被清空，则不会更新

```javascript

/**

	1. runner <-> effect
	2. effect.stop <-> deps




	1. stop => runner._effect.stop()
	2. 
		1. track -> activeEffect.deps.push(deps)
		2. effect.stop -> this.deps

*/




class ReactiveEffect{
	deps: any = []
	...

	stop(){
		
	}
}

function track(target, key){
	let depsMap = targetMap.get(target)
	...
	
	let dep = depsMap.get(key)
	...

	dep.add(activeEffect)

	// 反向收集dep
	activeEffect.deps.push(dep)
}

function effect(fn){
	const _effect = new ReactiveEffect(fn)

	const runner = _effect.run.bind(_effect)
	runner._effect = _effect

	return runner
}


function stop(runner){
	runner._effect.stop()
}





```



小优化点：
1. 封装cleanupEffect
2. 引入active：避免多次stop重复执行cleanupEffect
3. cleanupEffect执行后，length置0（deps中的依赖dep清空后，deps长度并没有归0）


## onStop
### case
```javascript
it('events: onStop', () => {
    const onStop = jest.fn();
    const runner = effect(() => {}, {
      onStop,
    });

    stop(runner);
    expect(onStop).toHaveBeenCalled();
  });

```

### thinking

1. effect 第二个参数中 传入一个 onStop
2. 绑定在 effect 对象上，在 stop 调用时判断是否存在 onStop，存在即调用

```javascript

// shared/index.ts

export const extend = Object.assign


```
