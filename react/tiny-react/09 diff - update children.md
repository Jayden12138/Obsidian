

```js

// type 不一致
// 删除旧的 创建新的















```



![[Pasted image 20240328102456.png]]
当处理完bar后，new处理完毕，但此时old节点中还存在foo/bar 的sibling未处理

```js

children.forEach(child=>{
	...
})

while(oldFiber){
	// 这里就是没有处理的兄弟节点
}


```