## UT
```javascript
it('isRef', () => {
      const a = ref(1);
      const user = reactive({
        age: 1
      })
      expect(isRef(a)).toBe(true);
      expect(isRef(1)).toBe(false);
      expect(isRef(user)).toBe(false);
});


it('unRef', () => {
    const a = ref(1)
    expect(unRef(a)).toBe(1)
    expect(unRef(1)).toBe(1)
})
```

## 思路
isRef
在 class 中定义一个标识即可，只有通过 ref 创建的，才可以访问到
```javascript
class RefImpl{
	public __v_isRef = true
}

function isRef(value){
	return !!value['__v_isRef']
}
```

unref
通过 isRef 来判断
	如果是 ref，则返回 ref.value 
	否则直接返回 当前 ref