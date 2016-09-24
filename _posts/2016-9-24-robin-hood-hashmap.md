---
layout: post
comments: false
title: Robin Hood Hashmap
category: security
tag: algorithm datastructure hash hashmap robinhood
---

在 feed 上看到一篇文章 \[1] 介绍了一种 Hashmap 的实现，称为 Robin Hood Hashmap。它也是一种基于开放定址（Open addressing）的哈希实现，但不同于以前被大家熟知的简单的线性探测（linear probing）或平方探测等等，它在发生冲突进行定址的时候，还维护了一种性质，据说可以使性能提升 20% 左右。出于好奇，我自己按照提出这一算法的原论文实现了一下，下面是实验过程和结果。

## 原理

为了理解 Robin Hood 的工作方式，首先要引用一个新概念，即 DIB（Distance to Initial Bucket）。DIB 故名思义指的是当前元素所在位置与它的初始位置之间的距离，这个初始位置就指的是经过 hash 运算后它的位置。

Robin Hood 的定址方式基于线性探测，但有一点不同，即它不是将新元素插到一个未被占用的地方，而是在线性探测的过程中，放在第一个 DIB 比它小的位置上去，而把原位置上的元素当做新元素继续依此规则向后查找，直到找到空位置为止。举个例子：

![RobinHood](https://o35qhjvld.qnssl.com/robinhood.png)

假设有如图的哈希函数，把六个字母按从左至右的顺序依次加到哈希表中。在加入前五个字母后，哈希表如上第二图所示。其中元素右下角的数字表示计算出的 DIB 值。当加入元素 x 时，有如下过程：

1. 计算得 `hash(x) = 1`。
2. `table[1]` 已经被占用，当前 `DIB(x) = 0`，小于 `DIB(b) = 1`。继续查找。
3. `table[2]` 已经被占用，当前 `DIB(x) = 1`，小于 `DIB(c) = 2`。继续查找。
4. `table[3]` 已经被占用，当前 `DIB(x) = 2`，大于 `DIB(d) = 0`。将 x 放入 `table[3]` 并且继续查找 d 的新位置。
5. `table[4]` 已经被占用，当前 `DIB(d) = 1`，等于 `DIB(f) = 1`。继续查找。
6. `table[5]` 为空，将 d 放入，结束。

### 查询

Robin Hood 的查询方式与插入对应。在哈希表中从初始哈希位置顺序查询即可。设当前位置为 `p`，查询的元素是 `key`，那么查询终止的条件就是 `DIB(P) < DIB(key)`。因为如果 `DIB(P) < DIB(key)` 成立的话，`key` 在插入时应该已经在这个位置上了。

### 删除

Robin Hood 有两种删除方式，按性质可以分为正常删除和懒惰删除。正常删除时，从被删除元素的位置向后查找，至到找到空位置或 `DIB(P) = 0` 的位置为止（不包含），将这一区间内的元素集体向左移动一个位置。这样的结果不但是要删除的元素被正常删除，且上述区间内的所有元素的 DIB 值都减少了 1。而懒惰删除则指的是给要删除的元素加一个标记，当再次插入新元素，而被删除的元素被置换出来的时候，不再继续置换，直接结束。

开始听下来感觉懒惰删除的表现可能更好一些，但其实正好相反 \[2]。

## 测试

为了测试 Robin Hood Hashmap 的综合性能，我们需要考虑它在高负载（即空位置很少）的情况下的插入、删除和查询的性能。我们令 hashmap 的大小为 333331 。然后把一个有 350k 个单词的单词表按顺序将 330k 左右（占 hashmap 的 90%）个单词作为 key，随机单词作为 value 加入 hashmap 中。然后随机从单词表中选择 key，进行 2000 次删除操作。最后再随机从单词表中选择 key，进行 300k 次查询操作。注意，由于没有把所有单词都作为 key 加入到 hashmap 中，可以认为删除和查询操作会涉及到 hashmap 中不存在的 key，这也是我们特意要考量的一个点。

我用 Go 语言实现了一个 Robin Hood Hashmap。为了与之进行比较，还实现了一个普通的 Linear probing Hashmap。两者使用相同的哈希函数：

```go
func hash(key string) (hashValue uint) {
	hashValue = 5381

	for _, c := range key {
		hashValue = ((hashValue << 5) + hashValue) + uint(c)
	}
	return hashValue
}
```

在实现过程中，需要特别注意以下几个点：

1. 先统一分配内存，而不是动态添加。
2. 每个 bucket 不但记录 key 和 value，同时为了计算 DIB，还需要记录 hash value。我们令 hash value 的未初始化值为 0，取值为 $(0, tableSize]$。这样只要看到 hash value 为 0 的 bucket，我们就可以确定它是未被占用的。具体做法是将 0 号 bucket 废除，下标从 1 开始计算。同时 `hashValue = rawHashValue % tableSize + 1`。
3. 线性探测时，有可能出现探测的位置越界的情况，所以我们从一开始就要假设这是一个循环数组。在位置加减时，要特殊处理。

代码保存在 [Github](https://github.com/MForever78/robinhood) 上。

## 结果

在 Ubuntu 16.04，Golang 1.6 版本下测试结果：

```bash
➜  robinhood /usr/bin/time ./robinhood > out
7.05user 2.97system 0:09.03elapsed 110%CPU (0avgtext+0avgdata 48404maxresident)k
0inputs+19272outputs (0major+9894minor)pagefaults 0swaps
➜  robinhood /usr/bin/time ./linear > out 
11.54user 3.10system 0:13.25elapsed 110%CPU (0avgtext+0avgdata 48624maxresident)k
0inputs+19272outputs (0major+9906minor)pagefaults 0swaps
➜  robinhood /usr/bin/time ./stdmap > out 
3.75user 2.67system 0:05.75elapsed 111%CPU (0avgtext+0avgdata 59816maxresident)k
0inputs+19272outputs (0major+10906minor)pagefaults 0swaps
```

其中 `linear` 是线性探测的方法。而 `stdmap` 是使用 Go 标准库中的 map，因为 hash 函数不同，而且 Go 标准库的实现使用了大量 unsafe pointer 以增加效率，不可直接比较，所以仅作为参考。顺带一提，Golang 的 map 在指定大小时，指定的是 bucket 数，而它的一个 bucket 中是可以放 8 个 key, value pair 的，在确定负载时要注意\[3]。

从结果来看，Robin Hood Hashmap 确实比 Linear probing 效率要高了 30% 左右。但是在有标准库的情况下，除了 interesting 之外，还有什么用呢 ╮(￣▽￣)╭

\[1]: [Robin Hood Hashing should be your default Hash Table implementation](http://www.sebastiansylvan.com/post/robin-hood-hashing-should-be-your-default-hash-table-implementation/)
\[2]: [Robin Hood hashing: backward shift deletion](http://codecapsule.com/2013/11/17/robin-hood-hashing-backward-shift-deletion/)
\[3]: [Source file src/runtime/hashmap.go](Source file src/runtime/hashmap.go)
