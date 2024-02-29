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
```

