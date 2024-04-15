>为了理解球盒模型，这里使用46题作为demo

```ts

// 1. js-includes
// 这里使用了js中的includes方法，用来避免重复添加

/*
 * @lc app=leetcode.cn id=46 lang=typescript
 *
 * [46] 全排列
 */

// @lc code=start
function permute(nums: number[]): number[][] {
	// 输入：nums = [1,2,3]
	// 输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]

	let resultList: number[][] = []

	let temp: number[] = []

	function backTrack() {
		if (temp.length === nums.length) {
			resultList.push([...temp])
			return
		}

		for (let i = 0; i < nums.length; i++) {
			// 判断是否已存在 存在则跳过（避免重复）
			if (temp.includes(nums[i])) continue
			temp.push(nums[i])

			backTrack()

			temp.pop()
		}
	}

	backTrack()

	return resultList
}
// @lc code=end
export { permute }





```



```ts

// 2. used
// 这里使用used 来存储nums的使用情况，避免重复添加

/*
 * @lc app=leetcode.cn id=46 lang=typescript
 *
 * [46] 全排列
 */

// @lc code=start
function permute(nums: number[]): number[][] {
	// 输入：nums = [1,2,3]
	// 输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]

	let resultList: number[][] = []

	let temp: number[] = []

	let used: boolean[] = new Array(nums.length).fill(false)

	function backTrack() {
		if (temp.length === nums.length) {
			resultList.push([...temp])
			return
		}

		for (let i = 0; i < nums.length; i++) {
			if (used[i]) continue

			temp.push(nums[i])
			used[i] = true

			backTrack()

			temp.pop()
			used[i] = false
		}
	}

	backTrack()

	return resultList
}
// @lc code=end
export { permute }



```



```ts


// 3. swap
// 不太理解

/*
 * @lc app=leetcode.cn id=46 lang=typescript
 *
 * [46] 全排列
 */

// @lc code=start
function permute(nums: number[]): number[][] {
	// 输入：nums = [1,2,3]
	// 输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]

	let resultList: number[][] = []

	function backTrack(start: number) {
		if (start === nums.length) {
			resultList.push([...nums])
			return
		}

		for (let i = start; i < nums.length; i++) {
			swap(nums, i, start)

			backTrack(start + 1)

			swap(nums, i, start)
		}
	}

	function swap(nums: number[], i: number, j: number) {
		;[nums[i], nums[j]] = [nums[j], nums[i]]
	}

	backTrack(0)

	return resultList
}
// @lc code=end
export { permute }


```

以上都是以盒为视角




```ts






```