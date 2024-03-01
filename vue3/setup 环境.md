## init 
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



## rollup
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


