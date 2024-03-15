# init 

```shell
yarn init -y

yarn add typescript --dev

npx tsc --init // generate tsconfig.json

yarn add jest @types/jest --dev

```

```json
// tsconfig.json

types: ["jest"],

noImplicitAny: false, // 项目中 隐式 any 不要报错

```


```json
// package.json

"script": {
	"test": "jest"
}

```

> jest 默认运行环境 nodejs -> commonjs 规范
> 需要使用 babel 进行转换（https://jestjs.io/docs/getting-started#using-babel）
```shell
yarn add --dev babel-jest @babel/core @babel/preset-env

```

```javascript
// babel.config.js

module.exports = {
	// 以当前的 node 版本为基础，对代码进行转换
	presets: [['@babel/preset-env', {targets: {node: 'current'}}]], 
};
```

>需要 jest 支持 typescript（）

```shell
yarn add --dev @babel/preset-typescript

```

```javascript
// babel.config.js
module.exports = {

	presets: [

		['@babel/preset-env', {targets: {node: 'current'}}],

		'@babel/preset-typescript',

	],
};
```



# rollup

```shell
yarn add rollup --dev

yarn add @rollup/plugin-typescript --dev

// rollup.config.js
import typescript from '@rollup/plugin-typescript'

export default{
	input: "./src/index.ts",
	output: [
		{
			format: 'cjs',
			file: 'lib/gudie-mini-vue.cjs.js'
		},
		{
			format: 'esm',
			file: 'lib/gudie-mini-vue.esm.js'
		},
	],
	plugins: [typescript()]
}

// package.json

"script": {
	"build": "rollup -c rollup.config.js"
}

yarn build

yarn add tslib --dev


// tsconfig.json
"module": "esnext"



```


# monorepo

> [babel/packages - github](https://github.com/babel/babel/tree/main/packages)
> [vue3/packages - github](https://github.com/vuejs/core/tree/main/packages)
> 以上库都采用了monorepo的架构


```

优势：
	1. 共用基础设施，不用重复配置
	2. 存在依赖的项目之间方便调试
	3. 第三方库的版本管理更好控制

缺点：
	1. 项目粒度的权限管理
	2. 代码量庞大

















```





常见
	1. [npm](https://www.npmjs.com/)
	2. [yarn](https://yarnpkg.com/)
	3. [pnpm](https://pnpm.io/zh/)
	4. [larna](https://lerna.js.org/)
	5. [turborepo](https://turbo.build/)
	6. [nx](https://nx.dev/)
	7. [RushJS](https://rushjs.io/)

> 前三个包管理工具有基础的monorepo，但存在或多或少的问题
> 后四各自都针对不同的问题场景提供了解决方案
> 在真实环境中，需要针对不同项目场景去选用合适的工具


Vue3
1. 使用pnpm
	1. 利用软链接的方式，节约磁盘空间，提高了安装速度
	2. 创建非扁平化的node_modules文件夹结构
	3. 对monorepo的支持非常好
2. build命令
	1. `pnpm build reactivity` : 只打包`reactivity`
3. test
	1. 依赖build（如下，会先执行build，再执行test）
		1. `"test-e2e": "node scripts/build.js vue -f global -d && vitest -c vitest.e2e.config.ts",`




-F(--filter 简写): 指定当前命令在哪个文件夹下执行
`pnpm i @tiny-vue/shared -F @tiny-vue/reactivity`
reactivity -> shared





















# vitest


