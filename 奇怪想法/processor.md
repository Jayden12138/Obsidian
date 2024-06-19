

>  定个flag
>  0618-0718 v0.1（shiki mdx 基础mdx转换code）
>  0801 v0.2（支持特定格式 focus）
>  0901 v0.3（加入scroll动画）
>  ...
>  


 - [ ] shiki-processor
	 - [ ] 阅读
	 - [ ] 总结
- [ ] vue-shiki-input
	- [ ] 阅读
	- [ ] 总结
- [ ] shiki
	- [ ] 使用
	- [ ] 


# shiki

> 在shiki-processor中，底层使用的是shiki

- code（原文本）
- tokens（标记数组）
- hast(hypertext abstract syntax tree 超文本抽象语法树)



# shiki-processor

> 原项目引用的[shiki](https://github.com/shikijs/shiki)，版本0.14.0，后续并没有更新迭代，截止23.06.18，最新版本为shiki@1.6.0

![[Pasted image 20240618141048.png]]

### diff

```

function() {
	console.log('owo') // [!code --]
	console.log('uwu') // [!code ++]
}


主题: nord
行为: has-diff
codeLine类型: diff remove / add

<pre class="shiki nord has-diff" style="">
	<code>
		<span class="line diff remove">
			<span></span><span></span>...
		</span>
		<span class="line diff add"></span>
	</code>
</pre>



```


![[Pasted image 20240618143745.png]]


```ts diff.test.ts


it('generates diffed lines on the same line as their tag', async () => {
	const snippet = `
	function() {
		console.log('owo') // [!code --]
		console.log('uwu') // [!code ++]
	}
	`
	await testDiffProcessor(snippet)
})





```





```ts

目的：配合《重构》，将代码渐进式的展示，用于学习

展示形式可以多样

初步定为code hike中scroll形式

后期可考虑实现step，diff，diff+git等方式

究极后期（趋近完美）
编辑器：方便用户编写自己的渐进式代码

.mdx文件
 支持特定的备注
 例如： focus:[1:2, 3]



将mdx中的文本信息转换为对应的code进行展示





```

- [ ] 


# .mdx



https://mdxjs.com/docs/

```

.mdx

\```js app.js [code]

[code...]

\```




```






---

---

# 整理

## v0.1

[v0.1 - codehike playground](https://play.codehike.org/#N4IgtgJgHiBcIB0B2ADNAHAngFwBYHslkAbAU2wAIBjfCUigXgoHIAVXASwGcLuKBhWvQASHANalmydACcOSbAAoadAJTIaSLvjIA6YvgDmyoetRpkyNCix5CFXKWIHdWEuWpDGLdnz6C6ClEJKSRZeSUVUjNNbT0DYyiza0skAB5+YV0A0gA+KwwcAiQHJxc3JDJKKO82Th5-IQBGUPCFEzUNQjjSfSMO6IKUIdti0ud8JtdMd2qvJjq-HhyAJla5dqSurR1ehIHki3SAekzsoXyiK9Sb67vbh-unx+QQABoQKi4uOBBXGVIADcOKQAO4AWk02AAhvJSDIKMAKMgKBQwNCZIZ5LAKAAOdBQADcyKQAF93p9CAAzDiGOCgYhwgByAFcwAAjeE-WDYGQs0gfLgEUGCLAAIRZ2GwhDgvP5H2hkvwABl5GJZXyBSA8KQwKRfujsPCONDiOCIBiJDIQKTSUA)
  
 通过mdx定义的code片段，转换为对应html代码进行展示

需要注意的：
- line number
- copy button
- theme
- 带文件名的
- 多文件


![[Pasted image 20240619135755.png]]

## v0.2

[v0.2 - codehike playground](https://play.codehike.org/#N4IgtgJgHiBcIB0B2BiFACAZgewMYFcBnZZHAwgXgEZlCBTAGztwBdD0WALOrASwCdCLdA15I6JJGSIUAzLACstRszYdu6fgEMkAcx7ZMIsXXayO2dEqSk8MqgG0A7LACcAXWVNW7Lj1zYDPhgSOxOFuiu6IbqPJgCQsbiktKUVAA08grpAGwOVLAALOkuHsgBYABGJr4aWpXYAG48dFBaYAAOTMQ2qGjore1dpil2acgABlMdAJ5c2Ejo3AwM2AB0s1hj1Mhae-sH+7uHJ1rHpwfnF3tX17cXk1Oj5HKKjxOz84vLqxszWy8svdTsCTqDDuDLkhrlCYUckFMJs97M43J4EdM5pwFktGL9NqlqKiytC4WdSXDIfCyeSaVSbhikb1CRksrl8kUSmj3p9sd88esCdtWYp2QViqV0XSKTD6bSyXLFYzJCreqr1WrNZIQOkQLhCIQ4CANvw6I1eHQAO4AWgCSBYWhM-HQwHQyHQ6DAWn4ujEsHQAA4OlAANxupAAXx1eoW8V0cFAonEADlgpU6II4JgtAx6LrCNjLQBhbCzABC+BYLAWWZzeZAWkr2AAMmIANa13N0XV+MB0I1elgZ3g560Qb1tjMgCMRoA)

focus 的基础实现

```ts

focus=1 selects the first line

focus=3:5 selects the range of lines 3 to 5

focus=1[7:9] selects the columns 7 to 9 of the first line

focus=1,3:5,6[1:4,7:9] combines the above examples

```

![[Pasted image 20240619141449.png]]


## v0.3

