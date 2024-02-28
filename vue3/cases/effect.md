## runner
### UT
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
###
1. effect() return runner
2. const r = runner() == effect.run() 
3. effect.run return this._fn()

## scheduler
### UT
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
###
1. 通过 effect 的第二个参数 传入scheduler
2. effect 第一次正常执行 fn
3. 当响应式数据发生 set 时，执行 scheduler
## stop
### UT
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
### 
1. stop runner
2. effect -> deps 清空
