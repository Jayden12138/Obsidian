## UT
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

##
```
reactive(raw) -> return Proxy {get set}

get track 收集依赖
set trigger 触发依赖

effect(fn) -> class ReactiveEffect{} run() 

targetMap -> depsMap -> dep
targetMap{target -> depsMap{key -> dep}}

Reflect.get()
Reflect.set()

```


