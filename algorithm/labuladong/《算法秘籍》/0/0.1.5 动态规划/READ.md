

## 509 斐波那契数

### 暴力穷举

>状态转移方程 === 暴力解

```js


// 1. 暴力穷举


/*
 * @lc app=leetcode.cn id=509 lang=typescript
 *
 * [509] 斐波那契数
 */

// @lc code=start
function fib(n: number): number {
	// F(0) = 0，F(1) = 1
	// F(n) = F(n - 1) + F(n - 2)，其中 n > 1
	// 输入：n = 2
	// 输出：1
	// 解释：F(2) = F(1) + F(0) = 1 + 0 = 1

	// 1.
	// F(20)
	// = F(19)  + F(18)
	// = F(18) + F(17) + F(17) + F(16)
	// = ...
	return dp(n)
}

function dp(n: number): number {
	if (n === 1 || n === 0) {
		return n
	}
	return dp(n - 1) + dp(n - 2)
}
// @lc code=end

export { fib }



```

### 备忘录

```js




// 备忘录

/*
 * @lc app=leetcode.cn id=509 lang=typescript
 *
 * [509] 斐波那契数
 */

// @lc code=start
function fib(n: number): number {
	// F(0) = 0，F(1) = 1
	// F(n) = F(n - 1) + F(n - 2)，其中 n > 1
	// 输入：n = 2
	// 输出：1
	// 解释：F(2) = F(1) + F(0) = 1 + 0 = 1

	// 2.
	// F(5)
	// f5
	// = f4 + f3
	// = f3 + f2 + f2 + f1
	// = f1 + f0 + f1 + f1 + f0 + f1 + f0 + f1
	// let arr = [f0, f1]

	let arr = new Array(n + 1).fill(0)
	arr[0] = 0
	arr[1] = 1

	return dp(arr, n)
}

function dp(arr: number[], n: number): number {
	if (n === 1 || n === 0) {
		return n
	}

	// 备忘录
	if (arr[n] !== 0) return arr[n]
	arr[n] = dp(arr, n - 1) + dp(arr, n - 2)

	return arr[n]
}
// @lc code=end

export { fib }




```

### db table

>2 中，通过自顶向下 从底部递归 得出结果
>构建一个db table，自底向上 从底部最基础的值 递推得出最后结果


```js

[0, 1]

f0 f1 f2 f3 f4 f5 f6
[0, 1, 1, 2, 3, 5, 8, ...]


// dp table

/*
 * @lc app=leetcode.cn id=509 lang=typescript
 *
 * [509] 斐波那契数
 */

// @lc code=start
function fib(n: number): number {
	// F(0) = 0，F(1) = 1
	// F(n) = F(n - 1) + F(n - 2)，其中 n > 1
	// 输入：n = 2
	// 输出：1
	// 解释：F(2) = F(1) + F(0) = 1 + 0 = 1

	// 3.
	// 构建dp table
	// 自底向上 递推结果

	let arr = [0, 1]

	for (let i = 2; i <= n; i++) {
		arr[i] = arr[i - 1] + arr[i - 2]
	}

	return arr[n]
}
// @lc code=end

export { fib }




// dp table - shrik

// 对于 dp(n) 来说 有用的是 dp(n - 1) dp(n - 2)
// 其他的没有必要进行存储

/*
 * @lc app=leetcode.cn id=509 lang=typescript
 *
 * [509] 斐波那契数
 */

// @lc code=start
function fib(n: number): number {
	// F(0) = 0，F(1) = 1
	// F(n) = F(n - 1) + F(n - 2)，其中 n > 1
	// 输入：n = 2
	// 输出：1
	// 解释：F(2) = F(1) + F(0) = 1 + 0 = 1

	// 3.
	// 对于 dp(n) 来说 有用的是 dp(n - 1) dp(n - 2)
	// 其他的没有必要进行存储

	if (n === 1 || n === 0) return n

	let arr = [0, 1]
	let res = 0

	for (let i = 2; i <= n; i++) {
		let num_i_0 = arr[0]
		let num_i_1 = arr[1]

		res = num_i_0 + num_i_1

		arr[1] = res
		arr[0] = num_i_1
	}

	return res
}
// @lc code=end

export { fib }



```



## 322 零钱兑换

### 暴力穷举

超时了！

```ts

// 输入：coins = [1, 2, 5], amount = 11
// 输出：3
// 解释：11 = 5 + 5 + 1

1.
暴力穷举
容易超时

/*
 * @lc app=leetcode.cn id=322 lang=typescript
 *
 * [322] 零钱兑换
 */

// @lc code=start
function coinChange(coins: number[], amount: number): number {
	// 输入：coins = [1, 2, 5], amount = 11
	// 输出：3
	// 解释：11 = 5 + 5 + 1
	// 输入：coins = [2], amount = 3
	// 输出：-1

	return dp(coins, amount)
}

function dp(coins: number[], amount: number): number {
	// [1, 2, 5] 11
	if (amount == 0) return 0
	if (amount < 0) return -1

	let res = Infinity

	for (let i = 0; i < coins.length; i++) {
		let subProblem = dp(coins, amount - coins[i])
		if (subProblem == -1) continue
		res = Math.min(subProblem + 1, res)
	}

	return res === Infinity ? -1 : res
}
// @lc code=end
export { coinChange }


```

### 备忘录

将重复的 “子问题” 进行缓存其结果

O(kn)

```ts

// 2. 备忘录


// 输入：coins = [1, 2, 5], amount = 11
// 输出：3
// 解释：11 = 5 + 5 + 1


dp(10) 
= dp(9) + dp(8) + dp(5) 
= (dp(8) + dp(7) + dp(4)) + dp(8) + dp(5)


消除重复子问题


/*
 * @lc app=leetcode.cn id=322 lang=typescript
 *
 * [322] 零钱兑换
 */

// @lc code=start
function coinChange(coins: number[], amount: number): number {
	// 输入：coins = [1, 2, 5], amount = 11
	// 输出：3
	// 解释：11 = 5 + 5 + 1
	// 输入：coins = [2], amount = 3
	// 输出：-1
	let memos: number[] = new Array(amount + 1).fill(-999)

	function dp(coins: number[], amount: number): number {
		// [1, 2, 5] 11
		if (amount == 0) return 0
		if (amount < 0) return -1

		// 注意这里 不能重复计算
		if (memos[amount] !== -999) {
			return memos[amount]
		}

		let res = Infinity

		for (let i = 0; i < coins.length; i++) {
			let subProblem = dp(coins, amount - coins[i])
			if (subProblem == -1) continue
			res = Math.min(subProblem + 1, res)
		}

		memos[amount] = res === Infinity ? -1 : res

		return memos[amount]
	}

	return dp(coins, amount)
}

// @lc code=end
export { coinChange }



```


### 自底向上 递推

前两种都是 自顶向下 ，通过递归得出结果

自底向上

```ts

/**


	[1, 2, 5] 11
	
	
	
	dp(0)
	[0]

	dp1
	dp0 dp-1 dp-4
	min(dp0) + 1 = 1

	dp2
	dp1 dp0 dp-3
	min(dp1 dp0 dp-3) + 1
	min(1 0) + 1 = 1

	dp3
	dp2 dp1 dp-2
	min(dp2 dp1 dp-2) + 1
	min(1.   1   ) + 1 = 2

	[0, 1, 1, 2, 2, 1, 2, 2, 3, 3, 2]


*/

/*
 * @lc app=leetcode.cn id=322 lang=typescript
 *
 * [322] 零钱兑换
 */

// @lc code=start
function coinChange(coins: number[], amount: number): number {
	// 输入：coins = [1, 2, 5], amount = 11
	// 输出：3
	// 解释：11 = 5 + 5 + 1
	// 输入：coins = [2], amount = 3
	// 输出：-1

	// 自底向上

	/**
	 *
	 * dp0
	 * [0]
	 *
	 * dp0 dp1
	 * dp1 = min(dp0 + dp-1 + dp-4) + 1
	 * [0, 1]
	 */

	let dptbale: number[] = new Array(amount + 1).fill(amount + 1)

	dptbale[0] = 0

	for (let i = 1; i <= amount; i++) {
		// dptable[i]
		for (let j = 0; j < coins.length; j++) {
			// coins[j]
			let currentDp = i - coins[j]
			if (currentDp < 0) continue

			dptbale[i] = Math.min(1 + dptbale[i - coins[j]], dptbale[i])
		}
	}

	return dptbale[amount] === amount + 1 ? -1 : dptbale[amount]
}

// @lc code=end
export { coinChange }





```

