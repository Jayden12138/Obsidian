

## 2.0

bun
https://bun.sh/

bun天然支持typescript，以及内置test
当前对于labuladong来说，在使用上相较于vite来说更好

```

# 全局安装bun

curl -fsSL https://bun.sh/install | bash


# 初始化环境
bun init

# 使用bun:test来测试



```

- [ ] 重构当前项目


alias

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

