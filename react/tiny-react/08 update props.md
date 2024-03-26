

```js

- 问题

1. 如何得到新的dom树
2. 如何找到老节点
3. 如果 diff props


- 思路

1. 如何得到新的dom树

执行 render 可以得到新的dom树

2. 如何找到老节点
3. 如果 diff props

/**

update props
需要获取到 旧的
如果从root开始遍历，有点x了
引入一个指针 alternate 指向对应的老节点，进而对比

*/


tag


















```