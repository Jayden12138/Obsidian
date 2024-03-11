>[AST Explorer](https://astexplorer.net/)
>[Vue 3 Template Explorer](https://template-explorer.vuejs.org/)



```javascript

 test('simple interpolation', ()=>{
	const ast = baseParse("{{message}}");

	// root
	expect(ast.children[0]).toStrictEqual({
		type: 'interpolation', // 'interpolation'
		content: {
			type: 'simple_expression', // 'simple_expression'
			content: 'message'
		}
	})
})



```