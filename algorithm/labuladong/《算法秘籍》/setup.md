

## 2.0

- [x] 重构当前项目 ✅ 2024-04-19

bun
https://bun.sh/

bun天然支持typescript，以及内置test
当前对于labuladong来说，在使用上相较于vite来说更好

```bash

# 全局安装bun

curl -fsSL https://bun.sh/install | bash


# 初始化环境
bun init

# 使用bun:test来测试
bun test

```



### alias

```

// https://bun.sh/guides/runtime/tsconfig-paths

// tsconfig.json
{
	"compilerOptions": {
		"paths": {
			"@/*": ["./src/*"]
		}
	}
}


// usage
import { searchRange } from '@/34' // ./src/34.ts


```


### auto-import 

官网 google没找到文档能支持自动引入decsribe等，现阶段只能手写了


### test

```bash

# test global
bun test

# test single case
bun test case.ts

# or by test-name-pattern
bun test  --test-name-pattern xxx

# check coverage
bun test --coverage


```


### debug

![[Pasted image 20240419163923.png]]


`command + shift + P`

![[Pasted image 20240419163955.png]]

![[Pasted image 20240419164237.png]]


问题

如果出现以下情况，可以将断点打在expect上，或重新断点多次尝试，暂不知道原因
![[Pasted image 20240419164309.png]]



---


## 1.0

>环境准备
>vitest、typescript

```

npm init -y

pnpm add typescript

npx tsc --init

pnpm add vitest


```

```js
// tsconfig.json


{
	"compilerOptions": {
		"types": [
			"vitest/globals"
		],
	},
	"include": ["./src"]
}


```

```js
// vitest.config.js
// 全局引入 describe、it、expect 等

import { defineConfig } from 'vitest/config'

export default defineConfig({
	test: {
		globals: true,
	},
})



```

