
> 参考文档: 
>[Function.prototype.bind MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
>[函数绑定](https://zh.javascript.info/bind)

#### 1. 绑定this


#### 2. partially applied - 部分函数
>一个较为通用的函数，通过bind固定了某个入参，得到的新的较为不通用的函数，称为部分函数

##### demo
```javascript

function sum(a, b){
	return a * b
}

const double = sum.bind(null, 2)

console.log(double(3)) // 6
console.log(double(4)) // 8

```

这里通过损失了一部分的通用度，得到了一个在某种情况下使用更加便捷、函数名更加语义化的函数
换个角度，可以将sum作为double的更底层的抽象，在封装上可以得到更好的抽象，以及代码低耦合，保证单一职责，这样方便后续的维护以及更好去实现单测
#### 3. 不处理this

> 在一些情况下，不处理this的绑定，只处理绑定arguments，这里可以使用包装器，通过返回一个函数来处理

```javascript

function partial(func, ...argsBound){
	return function(..args){
		func.call(this, ...argsBound, ...args)
	}
}

let user = {
  firstName: "John",
  say(time, phrase) {
    alert(`[${time}] ${this.firstName}: ${phrase}!`);
  }
};

user.sayNow = partial(user.say, new Date().getHours() + ':' + new Date().getMinutes());

user.sayNow("Hello"); // [xx:xx] John: Hello

```

上述只是简单实现，绑定的arguments必须从最左开始，无法绑定特定某一个参数

> lodash中实现了[\_.partial](https://lodash.com/docs/4.17.15#partial)
> 这里可以通过`_`来表示一个placeholder(占位符)，这样就可以只绑定特定位置上的参数，而其他位置的参数可以通过正常传参使用

```javascript

let user = {
  firstName: "John",
  say(time, phrase, toPerson) {
    console.log(`[${time}] ${this.firstName} to ${toPerson}: ${phrase}!`);
  }
};

user.sayNow = _.partial(user.say, new Date().getHours() + ':' + new Date().getMinutes(), _, 'Jayden');

user.sayNow("Hello"); // [xx:xx] John to Jayden: Hello

```

扩展partial函数，让他也可以接受`_`参数作为placeholder

```javascript



function partial(func, ...argsBound){
	return function(...args){
		let argsLength = args.length - 1
		let handled = 0
		let newArgs = []

		argsBound.forEach(item=>{
			let curArg;
			if(item === '_'){
				curArg = args[handled]
				handled++
			}else{
				curArg = item
			}
			newArgs.push(curArg)
		})
		func.call(this, ...newArgs)
	}
}

let user = {
  firstName: "John",
  say(time, phrase, toPerson) {
    console.log(`[${time}] ${this.firstName} to ${toPerson}: ${phrase}!`);
  }
};

user.sayNow = partial(user.say, new Date(), '_', 'Jayden');

user.sayNow("Hello"); // [xx:xx] John to Jayden: Hello

// =======================================================




```

