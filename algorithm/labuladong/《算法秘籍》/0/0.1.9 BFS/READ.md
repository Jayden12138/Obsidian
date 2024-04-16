

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

















```