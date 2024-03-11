
> 视图更新是异步的


> 当响应式数据修改后，此时通过getCurrentInstance获取不到最新的视图数据，是因为此时还没有执行更新



```javascript


setup(){
	const count = ref(1)
	const instance = getCurrentInstance()
	function onClick() {
		for (let i = 0; i < 100; i++) {
			console.log('update')
			count.value = i
		}
	}
},
render(){
	return h('p', {}, 'count: ' + this.count)
}

// 不处理前，更新操作是同步的，只要数据改变一次，视图就会执行一次update，但这样不好，更新太多次了，这里可以只渲染最后一次，效果是一样的，所以改为异步

// 通过effect scheduler，每次执行update时，将当前update推入队列中，

// promise .then .catch .finially 都是异步的，只有同步代码执行完后，才会执行队列中的job


const queues = []
let isFlushPending = false
const p = Promise.resolve()

export function queueJobs(job){
	if(!queues.includes(job)){
		queues.push(job)
	}
	queueFlush()
}

function queueFlush(){
	if(isFlushPending) return
	isFlushPending = true
	nextTick(()=>{ flushJobs() })
}

function flushJobs(){
	isFlushPending = false
	let job
	while(job = queues.shift()){
		job()
	}
}


export function nextTick(fn){
	return fn ? p.then(fn) : p
}




```


