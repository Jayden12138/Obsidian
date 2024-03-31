



```js


// 1

	function handleAdd() {
		setTodos([...todos, { title: inputValue }])

		setInputValue('')
	}


这里需要重构，将

setTodos([...todos, { title: inputValue }])

抽离出来，

// 2



{...todos.map(todo => (
					<li className={todo.status}>
						{todo.title}
						<button onClick={() => removeTodo(todo.id)}>
							remove
						</button>
						{todo.status === 'active' ? (
							<button onClick={() => doneTodo(todo.id)}>
								done
							</button>
						) : (
							<button onClick={() => cancelTodo(todo.id)}>
								cancel
							</button>
						)}
					</li>
				))}

抽离出来

TodoItem













```