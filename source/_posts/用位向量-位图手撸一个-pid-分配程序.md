---
title: 用位向量/位图手撸一个 pid 分配程序
date: 2020-03-31 22:49:00
tags: [算法]
category: 码农
---
# 什么是位向量/位图？

位向量/位图是一个数据结构，可以利用小空间存储大量数据。

通俗概括一下：**位向量是一个二进制数，我们要存储的数字是几，就让位向量的第几位为 1。这样就能使用一个数，存储大量的数了。**

# 位向量有什么用？

看了上面的概括，大家可能不太明白这种数据结构到底有什么用。这里我举一个非常常见的场景：**进程 pid 的分配**。

如果不使用位向量，你想一想要怎么保存那么多进程的 pid？需要用一个 int 数组来保存吗？这样势必会造成大量的空间浪费，我们平时使用的电脑、手机等设备可能不在乎这点空间，但是在一些嵌入式设备中，这点空间就弥足珍贵了。

接下来让我们一起看看用位向量怎么实现 pid 分配的逻辑。

# pid 分配

## 代码实现

直接上代码：

需要注意的是，这个程序只是模拟了 pid 的分配逻辑，并没有兼顾实际的业务逻辑（比如 0～299 的 pid 保留给 daemon 进程）。

```java
// 保存上一个 pid
static int last = -1;
// 用来保存使用中的 pid 的位向量
static int pids;
// 已经使用了多少 pid
static int usedPid = 0;
// 最大 pid 数量
static int MAX_PID = 10;

static int allocPid() {
  if (usedPid == MAX_PID) {
    throw new RuntimeException("No pid resources!");
  }
  int pid = last;
  do {
    pid += 1;
    if (pid >= MAX_PID) {
      pid = 0;
    }
  } while ((1 << pid) == ((1 << pid) & pids));
  pids = pids | (1 << pid);
  last = pid;
  usedPid++;
  return pid;
}
```

从代码中可以看到，我们使用了三个变量来存储，分别是 `last`、`pids` 和 `usedPid`。

* `pids` 就是我们用来保存使用中的 pid 的位向量
* `last` 用来保存上一次分配的 pid，主要为了保持 pid 的递增状态
* `usedPid` 是用来记录已经使用了多少 pid 了，用来判断是否还有剩余 pid 可供使用。

整个代码的流程就是：

1. 先判断还有没有剩余的 pid 资源
2. pid 在上次分配的基础上进行自增，如果到了最大值就从头开始重新递增
3. 检查自增后的 pid 是否已经被使用了
4. 直到寻找到一个可用的 pid
5. 返回此 pid

## 测试验证

**测试代码：**

```java
public static void main(String[] args) {
  System.out.println(allocPid());
  System.out.println(allocPid());
  System.out.println(allocPid());
  System.out.println(allocPid());
  
  // 释放 pid 为 3 的程序
  pids = pids & ~(1 << 3);
  usedPid -= 1;
  
  System.out.println(allocPid());
  System.out.println(allocPid());
  System.out.println(allocPid());
  System.out.println(allocPid());
  
  // 释放 pid 为 6 的程序
  pids = pids & ~(1 << 6);
  usedPid -= 1;
  
  System.out.println(allocPid());
  System.out.println(allocPid());
  System.out.println(allocPid());
  System.out.println(allocPid());
  System.out.println(allocPid());
  System.out.println(allocPid());
  System.out.println(allocPid());
  System.out.println(allocPid());
}
```

在测试代码中我们一直在调用 `allocPid` 分配 pid，因为我们没有写释放的函数，所以就手动释放掉了 pid  `3` 和 `6` 。

**输出：**

```shell
0
1
2
3
4
5
6
7
8
9
3
6
Exception in thread "main" java.lang.RuntimeException: No pid resources!
	at com.android.signapk.Main.allocPid(Main.java:33)
	at com.android.signapk.Main.main(Main.java:25)

Process finished with exit code 1
```

从输出可以看出：pid 一直在递增，回收了 `3` 和 `6` 后 pid 也没有中断递增。递增了一圈后开始第二轮，重新使用了 `3` 和 `6`。最后实在没有 pid 资源了，抛出了 `No pid resources` 异常。

# 最后

这次的代码可以在 Gist 上找到：https://gist.github.com/T-Oner/2f6d634b111dc2d50c528d34f0e8b7a5

