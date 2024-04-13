





```ts


// 第一阶段的代码
// 有问题！
// 在校验对角时有问题



斜着的校验有问题

1 3 0 4 2

14 、 02 

  [
    ".Q...",
    "...Q.",
    "Q....",
    "....Q",
    "..Q.."],

1 4
0 3 -> 3

两个值 m n
距离k
m - n !== k




/*
 * @lc app=leetcode.cn id=51 lang=typescript
 *
 * [51] N 皇后
 */

// @lc code=start
function solveNQueens(n: number): string[][] {
	// n [0, 9]
	// 输入：n = 4
	// 输出：[[".Q..","...Q","Q...","..Q."],["..Q.","Q...","...Q",".Q.."]]
	// 解释：如上图所示，4 皇后问题存在两个不同的解法。
	let resultList: string[][] = []

	/**
	 * n: 4
	 *
	 * choice: 0, 1, 2, 3
	 *
	 *
	 * [
	 *  ".Q..",
	 *  "...Q",
	 *  "Q...",
	 *  "..Q."
	 * ]
	 *
	 * 1302
	 *
	 * 每次 push 的新值 比上一个的差值 > 1
	 *
	 */

	let choice = []
	for (let i = 0; i < n; i++) {
		choice.push(i)
	}

	func([], choice)

	function func(path: number[], choice: number[]) {
		if (path.length === n) {
			// path [1, 3, 0, 2]

			/**
			 *
			 * path [1, 3, 0, 2]
			 *
			 * [
			 *  ".Q..",
			 *  "...Q",
			 *  "Q...",
			 *  "..Q."
			 * ]
			 *
			 * 1 -> ".Q.."
			 *
			 *
			 */

			let result: string[] = []

			for (const i of path) {
				let str = new Array(n).fill('.')
				str[i] = 'Q'
				result.push(str.join(''))
			}

			resultList.push([...result])
			return
		}

		for (const iter of choice) {
			let currentPathLength = path.length
			if (
				path.includes(iter) ||
				Math.abs(iter - path[currentPathLength - 1]) <= 1
			)
				continue

			path.push(iter)

			func(path, choice)

			path.pop()
		}

		return
	}

	return resultList
}
// @lc code=end
export { solveNQueens }

































```






