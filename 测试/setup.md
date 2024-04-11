


- [x] vite ✅ 2024-04-11
- [x] vue3 ✅ 2024-04-11
- [x] typescript ✅ 2024-04-11

- [x] vitest ✅ 2024-04-11

- [ ] vue-router

- [ ] pinia(vuex5)
- [ ] naive-ui
- [ ] cypress
- [ ] cypress-component-test
- [ ] alias
- [ ] ci && cd
- [ ] tailwind css
- [ ] lint + eslint + prettier
- [ ] axios
- [ ] 
- [ ] path









## vite

https://vitejs.dev/

```shell

# 使用vite 初始化 项目
pnpm create vite

pnpm i

pnpm dev

```

![[Pasted image 20240411143219.png]]


## vitest

https://vitest.dev/

```shell

pnpm add -D vitest


```

### auto import

https://vitest.dev/config/#globals

#### 方式一

```ts

// vitest.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
	test: {
		globals: true,
	},
})


// tsconfig.json
{ 
	"compilerOptions": 
	{ 
		"types": ["vitest/globals"] 
	} 
}

```

#### 方式二

unplugin-auto-import
https://github.com/unplugin/unplugin-auto-import

```ts

// vitest.config.ts
import { defineConfig } from 'vitest/config'
import AutoImport from 'unplugin-auto-import/vite'

export default defineConfig({
	plugins: [
		AutoImport({
			imports: ['vitest'],
			dts: true, // generate TypeScript declaration
		}),
	],
})


```


### vue router

https://router.vuejs.org/zh/

```



```









@types/node

https://www.npmjs.com/package/@types/node