

```ts



// 思路
思路完全不一样


分析后认为可以套用bfs，遍历二维数组，算出每一位上的最短step
但没有AC

思路：初始化二维数组 -1，遍历填入0
遍历计算 从0开始到每一个-1的最短路径，填入结果

不计算step
分组
queue 从0 -> 1 -> 2 -> ...
往上递增

current: 1
四个方向的值: [0, 0, -1, 0]
判断如果为 -1 则 赋值 currentValue + 1 => 1 + 1 => 2

[0, 0, 2, 0]





位置移动（上下左右）：
let dirs = [
	[0, 1],
	[0, -1],
	[1, 0],
	[-1, 0],
]





----------------------------------------

未AC思路


// 给定一个由 0 和 1 组成的矩阵 mat ，请输出一个大小相同的矩阵，其中每一个格子是 mat 中对应位置元素到最近的 0 的距离。

// 两个相邻元素间的距离为 1 。

  

/**
*
* [
* [0, 0, 0],
* [0, 1, 0],
* [1, 1, 1]
* ]
*
* [
* [0, 0, 0],
* [0, 1, 0],
* [1, 2, 1]
* ]
*
* 1. 当前为0则直接为0
* 2. 如果当前不是0，则需要在上下左右方向去找0，找到则返回最短路径（查询层数）
*
* 四个方向 边界判断 <0 || >row.length || >col.length
*
* 如果都没有找到 则返回-1
*
*/













```


```ts

// 1091.

超时？

/*
 * @lc app=leetcode.cn id=1091 lang=typescript
 *
 * [1091] 二进制矩阵中的最短路径
 */

// @lc code=start
function shortestPathBinaryMatrix(grid: number[][]): number {
	// 输入：grid = [[0,0,0],[1,1,0],[1,1,0]]
	// 输出：4

	// 输入：grid = [[0,1],[1,0]]
	// 输出：2

	// 输入：grid = [[1,0,0],[1,1,0],[1,1,0]]
	// 输出：-1

	let m = grid.length
	let n = grid[0].length

	if (grid[0][0] === 1 || grid[m - 1][n - 1] === 1) {
		return -1
	}

	let queue: number[][] = []
	let visited = new Set<number[]>()

	queue.push([0, 0])
	visited.add([0, 0])

	let dirs = [
		[0, 1],
		[0, -1],
		[-1, 0],
		[1, 0],
		[1, 1],
		[1, -1],
		[-1, 1],
		[-1, -1],
	]

	let step = 1
	while (queue.length) {
		const l = queue.length
		for (let i = 0; i < l; i++) {
			let [curI, curJ] = queue.shift()!

			if (curI === m - 1 && curJ === n - 1) {
				return step
			}

			dirs.forEach(([i, j]) => {
				let nextI = curI + i
				let nextJ = curJ + j

				if (
					nextI >= 0 &&
					nextI < m &&
					nextJ >= 0 &&
					nextJ < n &&
					grid[nextI][nextJ] === 0 &&
					!visited.has([nextI, nextJ])
				) {
					queue.push([nextI, nextJ])
					visited.add([nextI, nextJ])
				}
			})
		}
		step++
	}

	return -1
}
// @lc code=end
export { shortestPathBinaryMatrix }



这里超时是因为 visited 使用了 Set
将其改为二维数组进行存储即可

AC


/*
 * @lc app=leetcode.cn id=1091 lang=typescript
 *
 * [1091] 二进制矩阵中的最短路径
 */

// @lc code=start
function shortestPathBinaryMatrix(grid: number[][]): number {
	// 输入：grid = [[0,0,0],[1,1,0],[1,1,0]]
	// 输出：4

	// 输入：grid = [[0,1],[1,0]]
	// 输出：2

	// 输入：grid = [[1,0,0],[1,1,0],[1,1,0]]
	// 输出：-1

	let m = grid.length
	let n = grid[0].length

	if (grid[0][0] === 1 || grid[m - 1][n - 1] === 1) {
		return -1
	}

	let queue: number[][] = []
	let visited = new Array(m).fill(0).map(() => new Array(n).fill(false))

	queue.push([0, 0])
	visited[0][0] = true

	let dirs = [
		[0, 1],
		[0, -1],
		[-1, 0],
		[1, 0],
		[1, 1],
		[1, -1],
		[-1, 1],
		[-1, -1],
	]

	let step = 1
	while (queue.length) {
		const l = queue.length
		for (let i = 0; i < l; i++) {
			let [curI, curJ] = queue.shift()!

			if (curI === m - 1 && curJ === n - 1) {
				return step
			}

			dirs.forEach(([i, j]) => {
				let nextI = curI + i
				let nextJ = curJ + j

				if (
					nextI >= 0 &&
					nextI < m &&
					nextJ >= 0 &&
					nextJ < n &&
					grid[nextI][nextJ] === 0 &&
					!visited[nextI][nextJ]
				) {
					queue.push([nextI, nextJ])
					visited[nextI][nextJ] = true
				}
			})
		}
		step++
	}

	return -1
}
// @lc code=end
export { shortestPathBinaryMatrix }





```



```ts





js 太容易超时了
同样的代码，使用 Array.prototype.includes 来判断就会超时
重构为Set就好
但如果已经是Set了，还是不行，建议换方式优化，或者看是否可以改为下标获取 O(1)



/*
 * @lc app=leetcode.cn id=752 lang=typescript
 *
 * [752] 打开转盘锁
 */

// @lc code=start
function openLock(deadends: string[], target: string): number {
	// 输入：deadends = ["0201","0101","0102","1212","2002"], target = "0202"
	// 输出：6
	// 解释：
	// 可能的移动序列为 "0000" -> "1000" -> "1100" -> "1200" -> "1201" -> "1202" -> "0202"。
	// 注意 "0000" -> "0001" -> "0002" -> "0102" -> "0202" 这样的序列是不能解锁的，
	// 因为当拨动到 "0102" 时这个锁就会被锁定。

	// 输入: deadends = ["8888"], target = "0009"
	// 输出：1
	// 解释：把最后一位反向旋转一次即可 "0000" -> "0009"。

	// 输入: deadends = ["8887","8889","8878","8898","8788","8988","7888","9888"], target = "8888"
	// 输出：-1
	// 解释：无法旋转到目标数字且不被锁定。

	let deads = new Set(deadends)

	if (target === '0000') return 0
	if (deads.has('0000')) return -1

	// +1
	function forword(str: string, i: number) {
		let arr = str.split('')
		if (arr[i] === '9') {
			arr[i] = '0'
		} else {
			arr[i] = parseInt(arr[i]) + 1 + ''
		}
		return arr.join('')
	}

	// -1
	function backword(str: string, i: number) {
		let arr = str.split('')
		if (arr[i] === '0') {
			arr[i] = '9'
		} else {
			arr[i] = parseInt(arr[i]) - 1 + ''
		}
		return arr.join('')
	}

	let str = '0000'

	// target: '4000'
	// 0000 1000 2000 3000 4000

	/**
	 *
	 * 每一位都有两个选择 +1 和 -1
	 *
	 * target 0202
	 *
	 * [0, 0, 0, 0]
	 * [0, 1, 0, 0] [0, 9, 0, 0]
	 *
	 *
	 */

	let step = 0

	let queue: string[] = []
	let visited = new Set<string>()

	queue.push(str)
	visited.add(str)

	while (queue.length) {
		let l = queue.length
		for (let i = 0; i < l; i++) {
			let cur = queue.shift()!
			if (deads.has(cur)) continue

			//
			if (cur === target) return step

			//
			for (let j = 0; j < cur.length; j++) {
				let forwardStr = forword(cur, j)
				if (!visited.has(forwardStr)) {
					queue.push(forwardStr)
					visited.add(forwardStr)
				}

				let backwordStr = backword(cur, j)
				if (!visited.has(backwordStr)) {
					queue.push(backwordStr)
					visited.add(backwordStr)
				}
			}
		}
		step++
	}

	return -1
}
// @lc code=end
export { openLock }















```



```ts



使用 双向bfs
// 确实快了很多


双向bfs 使用条件，必须知道结束位置


/*
 * @lc app=leetcode.cn id=752 lang=typescript
 *
 * [752] 打开转盘锁
 */

// @lc code=start
function openLock(deadends: string[], target: string): number {
	// 输入：deadends = ["0201","0101","0102","1212","2002"], target = "0202"
	// 输出：6
	// 解释：
	// 可能的移动序列为 "0000" -> "1000" -> "1100" -> "1200" -> "1201" -> "1202" -> "0202"。
	// 注意 "0000" -> "0001" -> "0002" -> "0102" -> "0202" 这样的序列是不能解锁的，
	// 因为当拨动到 "0102" 时这个锁就会被锁定。

	// 输入: deadends = ["8888"], target = "0009"
	// 输出：1
	// 解释：把最后一位反向旋转一次即可 "0000" -> "0009"。

	// 输入: deadends = ["8887","8889","8878","8898","8788","8988","7888","9888"], target = "8888"
	// 输出：-1
	// 解释：无法旋转到目标数字且不被锁定。

	let deads = new Set(deadends)

	if (target === '0000') return 0
	if (deads.has('0000')) return -1

	// +1
	function forword(str: string, i: number) {
		let arr = str.split('')
		if (arr[i] === '9') {
			arr[i] = '0'
		} else {
			arr[i] = parseInt(arr[i]) + 1 + ''
		}
		return arr.join('')
	}

	// -1
	function backword(str: string, i: number) {
		let arr = str.split('')
		if (arr[i] === '0') {
			arr[i] = '9'
		} else {
			arr[i] = parseInt(arr[i]) - 1 + ''
		}
		return arr.join('')
	}

	let str = '0000'

	let step = 0

	let queue = new Set<string>()
	let queue2 = new Set<string>()
	let visited = new Set<string>()

	queue.add(str)

	queue2.add(target)

	while (queue.size && queue2.size) {
		let temp = new Set<string>()

		for (const cur of queue) {
			if (deads.has(cur)) continue

			//
			if (queue2.has(cur)) return step

			visited.add(cur)

			//
			for (let j = 0; j < cur.length; j++) {
				let forwardStr = forword(cur, j)
				if (!visited.has(forwardStr)) {
					temp.add(forwardStr)
				}

				let backwordStr = backword(cur, j)
				if (!visited.has(backwordStr)) {
					temp.add(backwordStr)
				}
			}
		}

		queue = queue2
		queue2 = temp

		step++
	}

	return -1
}
// @lc code=end
export { openLock }










```