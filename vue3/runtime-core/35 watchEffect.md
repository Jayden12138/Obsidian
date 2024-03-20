
> [watchEffect - Vue.js](https://cn.vuejs.org/api/reactivity-core.html#watcheffect)


## case

```javascript


describe('api: watch', () => {
  it('effect', async () => {
    const state = reactive({ count: 0 });
    let dummy;
    watchEffect(() => {
      dummy = state.count;
    });
    expect(dummy).toBe(0);

    state.count++;
    await nextTick();
    expect(dummy).toBe(1);
  });

  it('stopping the watcher (effect)', async () => {
    const state = reactive({ count: 0 });
    let dummy;
    const stop: any = watchEffect(() => {
      dummy = state.count;
    });
    expect(dummy).toBe(0);

    stop();
    state.count++;
    await nextTick();
    // should not update
    expect(dummy).toBe(0);
  });

  it('cleanup registration (effect)', async () => {
    const state = reactive({ count: 0 });
    const cleanup = vi.fn();
    let dummy;
    const stop: any = watchEffect((onCleanup) => {
      onCleanup(cleanup);
      dummy = state.count;
    });
    expect(dummy).toBe(0);

    state.count++;
    await nextTick();
    expect(cleanup).toHaveBeenCalledTimes(1);
    expect(dummy).toBe(1);

    stop();
    expect(cleanup).toHaveBeenCalledTimes(2);
  });
});




```

## thinking

```
区别：

reactivity/effect
watchEffect
watch


reactivity和Vue运行时没有依赖关系
watchEffect和Vue运行时强相关，默认是在组件渲染之前执行
渲染时机可以通过设置flush来控制


```