
使用上都是设置一个函数的this，并传入args，区别就是args的类型不同
apply可以接受一个类数组
call则需要spread语法来将数组进行展开传入

```javascript

function func(){...}

function wrap(){
	return function (...args){
		func.apply(this, args)
		// func.call(this, ...args)
	}
}

```



## 应用
> [debouncing-throttling - css-tricks](https://css-tricks.com/debouncing-throttling-explained-examples/)
> [debounce - lodash](https://lodash.com/docs/4.17.15#debounce)
> [throttle - lodash](https://lodash.com/docs/4.17.15#throttle)
### 防抖


### 节流



## 后记
### 1. args：类数组 & 可迭代对象

