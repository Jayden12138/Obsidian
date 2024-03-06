通过一个全局变量，来保存currentInstance，在当调用setup时，赋值为当前instance，此时调用`getCurrentInstance()`可以获取到当前调用setup的instance
```javascript

// init 全局变量
let currentInstance;


function(){
	currentInstance = instance;
	
	// 调用组件setup
	setup() 
	
	// reset
	currentInstance = null
}


function getCurrentInstance(){
	return currentInstance
}

```