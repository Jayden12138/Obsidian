## UT
```javascript
 it('proxyRefs', () => {
    const user = {
      age: ref(10),
      name: 'xiaohong'
    }

    const proxyUser = proxyRefs(user)
    expect(user.age.value).toBe(10)
    expect(proxyUser.age).toBe(10)
    expect(proxyUser.name).toBe('xiaohong')

    proxyUser.age = 20
    expect(proxyUser.age).toBe(20)
    expect(user.age.value).toBe(20)

    proxyUser.age = ref(10)
    expect(proxyUser.age).toBe(10)
    expect(user.age.value).toBe(10)
  })
```


## thinking

```

get

ref 返回 ref.value
非ref 直接返回

（注意：这里逻辑和unRef一致，直接复用unRef即可）


set

// oldVal ref / newVal 非ref 给ref.value赋值

// 直接赋值
// oldVal ref / newVal ref
// oldVal 非ref / newVal ref
// oldVal 非ref / newVal 非ref





```


## code


```
















```