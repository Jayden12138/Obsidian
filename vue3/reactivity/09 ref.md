# happy path
## UT
```javascript
it('happy path', () => {
	const a = ref(1)
	expect(a.value).toBe(1)
})
```
## 思路

因为 ref 也需要收集依赖 触发依赖
但是相比 reactive 而言，reactive 是接收一个对象，通过 创建 proxy 代理，通过 get 和 set 方法来收集依赖以及触发依赖
但是 ref 接受的是基础类型，例如 1 、 '123'
这些并没有 get set 来进行控制
所以通过 class 来创建

```javascript
class RefImpl{
	get value(){}
	set value(newValue){}
}
```

在实现 effect 时，通过创建targetMap{target -> depsMap{key -> dep}} 进行收集依赖
但当前 ref 并没有对应的 target key
所以在 class 中定义一个 deps 来保存依赖

这里需要把 effect 中原来的 track 和 trigger 进行重构，将 dep 相关的抽离出来，（相同逻辑进行复用）=> trackEffect / triggerEffect

# should be reactive
## UT
```javascript
it('should be reactive', () => {
      const a = ref(1);
      let dummy;
      let calls = 0;
      effect(() => {
        calls++;
        dummy = a.value;
      });
      expect(calls).toBe(1);
      expect(dummy).toBe(1);
      a.value = 2;
      expect(calls).toBe(2);
      expect(dummy).toBe(2);
      // same value should not trigger
      a.value = 2;
      expect(calls).toBe(2);
      expect(dummy).toBe(2);
});
```

1. 触发 set 时，需要执行 trigger
2. 当 value 没有发生改变的话，不执行 trigger（复用isTracking）


# nested object

## UT
```javascript
it('should make nested properties reactive', () => {
      const a = ref({
        count: 1,
      });
      let dummy;
      effect(() => {
        dummy = a.value.count;
      });
      expect(dummy).toBe(1);
      a.value.count = 2;
      expect(dummy).toBe(2);
});
```

reactive()

value isObject -> reactive(value) : value

==> hasChanged 需要修改
存储_rawValue（没有进行处理过的）





## question


```javascript



第一次盲写没有考虑到的（缺少对应的单测）
1. 在set value中没有考虑到非基础类型的change判断(Object.is)
	1. 判断时需要引入一个rawValue，用来对比是否发生了改变(因为这里可以传对象给ref，ref内处理对象时，需要在访问对象属性时，收集依赖，所以这里ref内进行了判断，如果是对象则通过reactive直接包裹一层，这样即可进行依赖的收集)











```
