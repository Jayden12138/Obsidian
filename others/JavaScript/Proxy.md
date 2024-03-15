
> [Proxy - MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)


```javascript


const rawObj = {
	name: "Jayden",
	age: 18
}

const proxyObj = new Proxy(rawObj, {
	get(target, key){
		console.log('proxy get')
		return Reflect.get(target, key)
	},
	set(target, key, value){
		console.log('proxy set')
		return Reflect.set(target, key, value)
	}
})






// proxy 代理对象
// 可以拦截对象的一些基础操作，类似get、set、delete等



```




## others

1. [[Reflect]]
2. 
