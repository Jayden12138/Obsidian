
```javascript

describe('should parse text', ()=>{
	test('simple text', ()=>{
		const ast = baseParse("some text");
		expect(ast.children[0]).toStrictEqual({
			type: NodeTypes.TEXT,
			content: 'some text'
		})
	})
})

```