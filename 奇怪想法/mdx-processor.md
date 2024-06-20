

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






codehike 23年开始有v1.0的打算

后续自建时需要遵循tdd





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

---

# @mdx-js/mdx


## runSync


```ts

import { runSync } from "@mdx-js/mdx"; 

const content = ` # Hello, MDX! This is a **markdown** document with JSX embedded. `; 

const result = runSync(content); 
console.log(result);


```





# codehike

>项目跑起来了，demo也能看了，因为单测的用例太少了，所以只能根据demo去写一些test来调试去了解每个过程
>cd packages/mdx
>yarn install
>yarn dev
>yarn test

remove shiki
必须搞懂底层转换的各个节点，以及方式，进而找到适合自己预期的转换方式 -- 构建自己的转换器（最好不依赖，shiki在0.14 -> 1.6.4改变很大，在后续维护也很不方便，可以在前期进行快速迭代开发，但还是不依赖的好）

- prism-react-renderer
- clsx
- @mdx-js/mdx

prism-react-renderer
需要了解下prism和他的关系，目前使用的是vue3+ts所以这个用不到，但有必要了解下基础的用法


clsx 对classname进行处理的，后续在转换器中可能需要

@mdx-js/mdx  很关键，用来处理mdx文件



## utils

### focus


#### parseExtremes
// Transforms something like
// - "1:3" to {start:1, end: 3}
// - "4" to {start:4, end:4}

#### mapFocusToLineNumbers

```

1:2,3[1:5,7]

=> {
	1: true,
	2: true,
	3: [
		{start: 1, end: 5},
		{start: 7, end: 7}
	]
}


```

#### relativeToAbsolute

![[Pasted image 20240619154715.png]]

前两个都好理解，第三个不太懂，为什么需要转换成这样



### color

好像是自定义那块有用到，直接定义颜色，这里作为一个工具使用吧，暂时没看到哪里调用

#### hexToObject

```ts

"#ff88ff"

=> {
	"a": 1,
	"b": 255,
	"g": 136,
	"r": 255,
}


```

#### objectToHex

```ts

{ r: 255, g: 136, b: 255, a: 0.5 }


=> "#ff88ff80"


```


### theme


#### transparent

```ts

"#ffffff", 0.5

=> "#ffffff80"


```


## smooth-code

### splitter

#### splitTokens

![[Pasted image 20240619161020.png]]









---

需要了解下
- @docusaurus/core




可以选择了解
- shiki

