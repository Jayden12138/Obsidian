

```js


// App.jsx
function Foo() {
	const [count, setCount] = CReact.useState(10)
	function handleClick() {
		setCount((c) => c + 1)
	}
	return (
		<div>
			foo
			{count}
			<button onClick={handleClick}>click</button>
		</div>
	)
}



```



