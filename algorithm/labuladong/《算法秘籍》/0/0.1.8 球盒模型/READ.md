>为了理解球盒模型，这里使用46题作为demo

# 球盒模型
## 盒 视角

### 1. js-includes

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

### 2. used

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

### 3. swap

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


## 球 视角



```ts


// 以球的视角？
// 设置了一个used来表示当前有哪个元素进行了选择位置


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

	let used: boolean[] = new Array(nums.length).fill(false)

	function backTrack(start: number) {
		if (used.every(v => v === true)) {
			resultList.push([...nums])
			return
		}

		for (let i = start; i < nums.length; i++) {
			swap(nums, i, start)
			used[start] = true

			backTrack(start + 1)

			swap(nums, i, start)
			used[start] = false
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


```ts

// 2. count
// 利用count 计数，当处理了nums.length - 1个元素时，可以确定当前一种结果已完成


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

	let count: number = 0

	function backTrack(start: number) {
		if (count === nums.length - 1) {
			resultList.push([...nums])
			return
		}

		for (let i = start; i < nums.length; i++) {
			swap(nums, i, start)
			count++

			backTrack(start + 1)

			swap(nums, i, start)
			count--
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