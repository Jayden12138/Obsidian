jsx


```js

// 修改 App.js -> App.jsx

// 报错如下
Failed to load module script: Expected a JavaScript module script but the server responded with a MIME type of "text/jsx". Strict MIME type checking is enforced for module scripts per HTML spec.

// 浏览器不认识 jxs 文件
// 这里需要使用vite、webpack、babel等，对代码进行转换



```


vite

```js
// 使用vite 以支持jsx

pnpm create vite
// 创建一个vite项目，将前一节写的相关代码(App.js, main.js, core)带入




// App.jsx
import React from './core/React.js'

// const App = React.createElement('div', { id: 'app' }, 'hello world')
const App = <div id="app">hello world</div>
console.log(App)
/**
{
    "type": "div",
    "props": {
        "id": "app",
        "children": [
            {
                "type": "TEXT_ELEMENT",
                "props": {
                    "nodeValue": "hello world",
                    "children": []
                }
            }
        ]
    }
}
 */

const AppFun = () => {
	return <div id="app">hello world</div>
}
console.log(AppFun)
/**
const AppFun = () => {
	return <div id="app">hello world</div>
} 
vite 会将上述代码转换为
() => {
  return React.createElement("div", { id: "app" }, "hello world");
}
 */

export default App


// tip: 目前还不支持function component



```