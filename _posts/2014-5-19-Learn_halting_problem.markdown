---
layout: post
comments: true
title: 停机问题学习笔记
category: Notes
tag: feynman
---

![Turing Machine](http://mforever78.qiniudn.com/turing_machine.jpg)

前段时间看到 Scott H Young 那篇介绍费曼技巧[^1]的文章，一下想到了程序员界流传甚广的小黄鸭调试法（Rubber Duck Debugging）[^2]。几天前晚上散步的时候，聊天儿时提起了罗素悖论，让我联想到图灵对停机问题精妙的解法。想拿这个试试费曼技巧，讲到一半发现卡壳了，自己把自己绕了进去。我想到刘未鹏说过的那种状态，学习一个算法，初看觉得太牛逼了，然后花十分钟理解了人家花了十年才想出的成果，再花十天完全忘记，只能想起那个算法十分精妙，该不会的还是不会。所以应该由本溯源地学习数学和算法，并且写一个博客记录下学习的过程。于是这篇就作为费曼技巧和写作学习实践的第一篇。

在计算机科学领域，最广为人知的问题是关于 P 与 NP 问题的讨论，但不论 P 还是 NP，它们只是对解决问题效率的讨论。其实还有一类问题，它们难到计算机永远无法解出，这类问题被称作不可判定问题（Undecidable Problem）[^3]，其中一个可能是最有名的问题就是停机问题。

停机问题提出了这样的疑问：是否存在一个程序，可以判定任何一个程序是否会停止。

要解决这个问题，我们先假设存在一个程序满足上述条件，比如：

```
int yourBrilliantProgram(char * program, inputType  input){
	if (program(input) stops)
		return 1;
	else //It loops forever
		return 0;}
```

其中，第一个 `if` 语句中的判断是人类智慧的结晶，它用了神奇的方法得知了 `program(input)` 是否停止。

接着我们就要进行破坏了，再来考虑这样一个程序：

```
int myDisgustingProgram(char * program){
	if (yourBrilliantProgram(program, program)){
		while(1);
		return 0; //Never reaches here	}
	else
		return 1;}
```

我的程序专门跟你作对，如果你说这个程序可以正常退出，那么我就作一个死循环；你说它是死循环的，我就正常退出给你返回一个`true`。

So far so good. 最精彩的地方来了，考虑 `yourBrilliantProgram(myDisgustingProgram, myDisgustingProgram)` 的返回值。这里有点绕，我们一步一步跟踪。

首先，进入 `yourBrilliantProgram` 的第一个判断语句，这里执行 `myDisgustingProgram(myDisgustingProgram)` ，进入 `if(yourBrilliantProgram(myDisgustingProgram, myDisgustingProgram))` 这一句。这里可能出现两个结果：

- 返回 `1`，说明 `myDisgustingProgram(myDisgustingProgram)` 正常退出，那么当前程序进入死循环，此时上一层的 `yourBrilliantProgram()` 应该检测到这个死循环，并返回 `0`，说明 `myDisgustingProgram(myDisgustingProgram)` 是个死循环。矛盾。
- 返回 `0`，说明 `myDisgustingProgram(myDisgustingProgram)` 死循环，那么当前程序返回 `1` 并且正常退出，又证明 `myDisgustingProgram(myDisgustingProgram)` 不是死循环。矛盾。

也就证明，根本不存在这样的程序可以判定任何一个程序是否停止。

[^1]: [Scott H Young 如何在十天内掌握线性代数](http://select.yeeyan.org/view/94114/329073)
[^2]: [小黄鸭调试法](http://zh.wikipedia.org/wiki/小黄鸭调试法)
[^3]: [不可判定问题列表](http://zh.wikipedia.org/wiki/不可判定问题列表)