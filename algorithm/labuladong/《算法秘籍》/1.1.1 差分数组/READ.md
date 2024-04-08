
>区间内的差值不会改变
>改变的只有区间开头以及区间后下一个（如果区间后不存在则不需要进行修改，在recover阶段，会把以i开头之后的所有项进行修改）


关键代码

```ts


class DiffArr {
	diff: number[] = []
	constructor(arr: number[]) {
		this.diff = arr.slice(0)
		const l = arr.length

		for (let i = 1; i < l; i++) {
			this.diff[i] = arr[i] - arr[i - 1]
		}
	}

	increase(start: number, end: number, val: number) {
		this.diff[start] += val
		if (end + 1 < this.diff.length) {
			this.diff[end + 1] -= val
		}
	}

	recover() {
		const l = this.diff.length
		const res = this.diff.slice(0)
		for (let i = 1; i < l; i++) {
			res[i] = res[i - 1] + this.diff[i]
		}

		return res
	}
}


```


## case

### 370

![[Pasted image 20240408163512.png]]



