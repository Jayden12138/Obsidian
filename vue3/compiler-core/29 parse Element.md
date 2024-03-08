
```javascript


 test('simple element', ()=>{
	const ast = baseParse("<div></div>");

	expect(ast.children[0]).toStrictEqual({
		type: NodeTypes.ELEMENT,
		tag: 'div'
	})
})

```