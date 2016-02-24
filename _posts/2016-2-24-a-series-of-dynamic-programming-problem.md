---
layout: post
comments: false
title: 一个系列动态规划问题
category: algorithm
tag: leetcode algorithm dynamic_programming
---

![crazy_professor](http://mforever78.qiniudn.com/crazy_professor.jpg)

Leetcode 碰到的一个成系列的动态规划问题，由简单到复杂，很有意思。

看最初版本。

> 121\. Best Time to Buy and Sell Stock
>
> Say you have an array for which the ith element is the price of a given stock on day i.
>
> If you were only permitted to complete at most one transaction (ie, buy one and sell one share of the stock), design an algorithm to find the maximum profit.

这道题非常直观也非常简单。想要通过一次买卖得到的利润，就是要找到一组 `i` 和 `j`，使得 `prices[j] - prices[i]` 最大，并且满足 `i < j`。因为第二个约束条件，我们不会傻到找一个最大值和最小值并且返回它们的差。

假设 `f[i]` 为到第 `i` 天为止可以拿到的最大利润。对于第 `i` 天，有两种选择，即在当天卖掉股票，或者在第 `i` 天之前已经卖掉了。那么 `f[i]` 就是这两种选择中的最大值。如果在第 `i` 天卖掉股票，那么问题就是在哪天买股票，只要维护一个到第 `i` 天为止股价的最小值 `minPrice` 就可以了，此时 `f[i] = prices[i] - minPrice`；如果在第 `i` 天股票已经卖出，则 `f[i] = f[i-1]`。

综上所述，令 `f[i]` 表示到第 `i` 天为止可以拿到的最大利润。状态转移方程为 `f[i] = max(f[i-1], prices[i] - minPrice)`，其中 `minPrice` 表示到第 `i` 天为止的最低股价，并且有 `minPrice = min(minPrice, prices[i])`。边界条件为 `f[0] = 0, minPrice = prices[0]`。最终结果为 `f[n-1]`。时间复杂度与空间复杂度均为 `O(n)`。

观察状态方程可以发现，`f[i]` 的值只与 `f` 数组中的 `f[i-1]` 有关。也就是说，在计算 `f[i]` 时，只要保留 `f[i-1]` 的值就好了，其他的值都可以不保存。据此可以优化空间复杂度。

令 `profit` 表示到第 `i` 天为止可以拿到的最大利润。状态转移方程为 `profit = max(profit, prices[i] - minPrice)`。`profit` 初始化为 `0`。最终结果即保存在 `profit` 中。时间复杂度为 `O(n)`，空间复杂度为 `O(1)`。

解决，看第二版。

> 122\. Best Time to Buy and Sell Stock II
>
> Say you have an array for which the ith element is the price of a given stock on day i.
>
> Design an algorithm to find the maximum profit. You may complete as many transactions as you like (ie, buy one and sell one share of the stock multiple times). However, you may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).

与初始版本的不同之处在于，股票可以被无限次买入买出，而不是只能进行一次买卖。需要注意的是，与真实股票买卖不同，这里的股票只能持有一份，也就是说，在卖出手中持有的股票之前是不能再次购买的。（所以其实用股票这个模型容易产生误解，这是题目描述得不科学的地方）

由于可以无限次买卖，上次的状态模型显然不能用了。考虑第 `i` 天的行为可以是买入、卖出，或什么都不做。如果要买入，显然至少在第 `i-1` 天的时候手中是没有持有股票的，而如果要卖出，则至少在第 `i-1` 天的时候手中必须持有股票，如果什么都不做，则没有限制。根据这样的分析，我们可以知道为了计算第 `i` 天的情况，我们（只）需要第 `i-1` 天时持有和未持有股票的情况。

据此可以抽象出状态。令 `hold[i]` 表示第 `i` 天时持有股票可以获得的最大利润，`free[i]` 表示第 `i` 天时未持有股票能获得的最大利润。第 `i` 天持有股票有两种情况，即第 `i-1` 天就已经持有，或在第 `i` 天买入。即 `hold[i] = max(hold[i-1], free[i-1] - prices[i])`。同理第 `i` 天未持有股票也有对应两种情况，即第 `i-1` 天未持有，或第 `i` 天卖出。即 `free[i] = max(free[i-1], hold[i-1] + prices[i])`。初始条件为 `hold[0] = -prices[0], free[0] = 0`。最终结果为 `free[n-1]`。时间复杂度和空间复杂度均为 `O(n)`。

与上题相同，本题也可以将空间优化成 `O(1)`。令 `holdLast, hold` 表示到第 `i-1, i` 天持有股票可以获得的最大利润，`freeLast, free` 表示到第 `i-1, i` 天卖出股票可以获得的最大利润。则

```c+
hold = max(holdLast, freeLast - prices[i]);
free = max(freeLast, holdLast + prices[i]);
holdLast = hold;
freeLast = free;
```

初始条件为 `holdLast = -prices[0], freeLast = 0`。最终结果保存在 `free` 中。

解决，再来看一个番外篇。

> 309\. Best Time to Buy and Sell Stock with Cooldown
>
> Say you have an array for which the ith element is the price of a given stock on day i.
>
> Design an algorithm to find the maximum profit. You may complete as many transactions as you like (ie, buy one and sell one share of the stock multiple times) with the following restrictions:
>
> - You may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).
> - After you sell your stock, you cannot buy stock on next day. (ie, cooldown 1 day)

这道题在上题的基础上加了一个 CD 时间的限制。Easy stuff。

状态表示和上题完全一样，不同的是第 `i` 天买入的状态要从第 `i-2` 而不是 `i-1` 天转移而来。即 `hold[i] = max(hold[i-1], free[i-2] - prices[i])`。边界条件为 `hold[0] = -prices[0], hold[1] = max(hold[0], -prices[1]), free[0] = 0, free[1] = max(hold[0] + prices[1], free[0])`。最终结果保存在 `free[n-1` 中。时间和空间复杂度均为 `O(n)`，同理空间复杂度可以优化到 `O(1)`。

解决。然而还没完，来看第三版。

> 123\. Best Time to Buy and Sell Stock III
>
> Say you have an array for which the ith element is the price of a given stock on day i.
>
> Design an algorithm to find the maximum profit. You may complete at most two transactions.
>
> **Note:**
>
> You may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).

和第二题不同的地方在于，这道题限制最多只能进行两次交易。

由于题目限制在买入之前必须未持有股票，也就是说，两次交易是不会交叉的，即两次交易的时间段不会产生重叠。既然如此，我们就可以把这看成两个独立的交易，从而把问题转化成最初版本的问题。

具体解释。令 `f[i]` 表示从第 `0` 天到第 `i` 天可以获得的最大利润，`g[i]` 表示从第 `i` 天到第 `n-1` 天可以获得的最大利润。则 `f[i] = max(f[i-1], prices[i] - minPrice), g[i] = max(g[i+1], maxPrice - prices[i])`。其中 `g[i]` 从后往前计算，记录未来最高价，枚举确定要不要第 `i` 天买入。最后枚举分界点，找到 `f[i] + g[i+1]` 的最大值即为所求，思路还是比较清晰。

解决。然而系列之所以为系列就是因为版本真的很多。来看最后一版。

> 188\. Best Time to Buy and Sell Stock IV
>
> Say you have an array for which the ith element is the price of a given stock on day i.
>
> Design an algorithm to find the maximum profit. You may complete at most k transactions.
>
> **Note:**
>
> You may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).

只允许进行两次交易的用特殊方法解决，那么允许 k 次交易的该用什么方法呢？

其实题做到这里，思路继承下来还是不难的。只需要加一维把交易次数记下来就可以了。`free[i][j]` 表示第 `i` 天未持有股票，并且完成了 `j` 次交易，可以获得的最大利润。转移方程为 `free[i][j] = max(free[i-1][j], hold[i-1] + prices[i]), hold[i] = max(hold[i-1], free[i-1][j-1] - prices[i])`。最终结果保存在 `free[n-1][k-1]` 中。时间复杂度为 `O(k*n)`。

但是到这里还没有结束，对于题目给出的 `n` ，一个隐性的条件是我们最多只能完成 `n/2` 次交易，即一天买入，第二天即卖出。所以当 `k > n/2` 时，这道题就退化成了可以无限次买入卖出的第二题。事实上，由于时间复杂度依赖于 `k`，我们必须做这个判断，否则题目数据给一个非常大的 `k` 就将我们的程序卡掉了。

到这里这一系列的动态规划题就结束了，回头来看还是挺简单的。记下当时的思维过程，希望以后看的时候可以快速回忆起这类模型的解法。

