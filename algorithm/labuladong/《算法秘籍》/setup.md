

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

