

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

## code

 >[commit: effect & reactive -- Jayden Github](https://github.com/Jayden12138/tiny-vue/commit/0f1d85545d0f85ab05b65fdece3eeae869bca4f7)

```javascript









```



## question





## others

> 1. [[Proxy]]
> 2. [[Reflect]]