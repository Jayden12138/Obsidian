
>模拟结果来自：[ Vue-template-explorer](https://template-explorer.vuejs.org/)

# string

```javascript

// codegen.spec.ts happy path

it('string', () => {
	const ast = baseParse('hi')

	const { code } = generate(ast)

	expect(code).toMatchSnapshot()
})

  
// hi
return function render(_ctx, _cache, $props, $setup, $data, $options) {
	return "hi"
}


// codegen.spec.ts.snap
exports[`codegen string 1`] = `"return function render(_ctx, _cache) {return 'hi'}"`;


```
# interpolation

```javascript


it('interpolation', () => {
	const ast = baseParse('{{message}}')

	transform(ast)
	const { code } = generate(ast)

	expect(code).toMatchSnapshot()
})


// {{message}}
const { toDisplayString: _toDisplayString } = Vue

return function render(_ctx, _cache, $props, $setup, $data, $options) {
	return _toDisplayString(_ctx.message)
}


// codegen.spec.ts.snap
const { toDisplayString: _toDisplayString } = Vue
return function render(_ctx, _cache) {return _toDisplayString(_ctx.message)}



```


```javascript

interpolation -> toDisplayString


```



# 三种联合

## simple element


```javascript

// codegen.spec.ts

it('element', () => {
	const ast = baseParse('<div></div>')

	transform(ast, {
		nodeTransforms: [transformElement],
	})
	const { code } = generate(ast)

	expect(code).toMatchSnapshot()
})



// <div></div>
const { openBlock: _openBlock, createElementBlock: _createElementBlock } = Vue

return function render(_ctx, _cache, $props, $setup, $data, $options) {
	return (_openBlock(), _createElementBlock("div"))
}

/**
	这里openBlock不进行实现，openBlock作用为优化
*/


// codegen.spec.ts.snap
const { createElementVNode: _createElementVNode } = Vue
return function render(_ctx, _cache) {return _createElementVNode("div")}"



```

## union

```javascript
















```