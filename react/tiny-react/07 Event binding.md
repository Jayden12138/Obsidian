
```js

// 事件绑定

// App.jsx
function Counter({ num }) {
	function handleClick() {
		console.log('click')
	}
	return (
		<div>
			<span>Counter: {num}</span>
			<button onClick={handleClick}>click</button>
		</div>
	)
}


// React.js
function updateProps(dom, props) {
	// props
	Object.keys(props).forEach(key => {
		if (key !== 'children') {
			if (key.startsWith('on')) {
				// onClick => click
				const eventName = key.slice(2).toLocaleLowerCase()
				dom.addEventListener(eventName, props[key])
			} else {
				dom[key] = props[key]
			}
		}
	})
}





```