---
title: 
   由于cmd引发的血案
tags:
   [Bug]
categories:
   [踩坑记录]
date: 2019/12/8
cover: http://static.imlgw.top/4.jpg
---

## 前言

给教务处做一个工作量审核的web项目，前几天完成后部署在机房电脑上进行测试，但是测试的时候出现了Bug，会存在一个用户登录的时候，整个应用卡死，所有人都无法登陆，因为登陆模块是我写的，我当时就慌了（并不），然后赶紧去机房看看到底是啥情况

## FixBug

其实我首先想到的就是数据库的问题，但是看了我控制台的输出，发现代码其实根本就还没到数据库层！所以先排除了数据库的问题

然后我就想到了可能是GC的问题，然后我打开了 `jvisualvm`，准备查看当前的堆占用和GC情况，但是由于现在是正常的，看不出来什么，所以我们需要复现这个Bug，结果我们几个人在哪里搞了半天死活复现不了😂

> 在经过我们快一个小时坚持不懈的尝试下，终于复现了！

页面hang住，所有人都无法登陆，后台也并没有任何的错误信息，然后我赶紧去看了下 `jvisualvm` 发现堆并没有任何变化，GC也并没有发生！而且更诡异的是其他的已经登陆的人是可以正常的操作的！只是卡住了登陆的人，所以GC的问题也排除了

排除上面两个原因，剩下的就只有一个原因了，线程死锁！

在dump出线程快照后终于发现了问题所在

![mark](http://static.imlgw.top/blog/20191208/XKUriuCE9nTb.png?imageslim)

图中的这条线程卡在了 `PrintStream.println()` 上，而这个是我在Service打印的log信息，为什么会在这里卡住？？？这不科学啊，然后我看了 其他 tomcat的工作线程，发现还有好几个都是`BLOCK` 状态，都在等 `[0x0000004df8afd000]`这把锁，这个锁被另一个tomcat的工作线程 `http-bio-80-exec[2]` 所持有，而且它并不是`BLOCK`状态，而是 `RUNNABLE` 状态，这个线程正在执行 `java.io.FileOutputStream.writeBytes()` 方法，我们去看看`println`的源码

```java
/**
 * Prints a String and then terminate the line.  This method behaves as
 * though it invokes <code>{@link #print(String)}</code> and then
 * <code>{@link #println()}</code>.
 *
 * @param x  The <code>String</code> to be printed.
 */
public void println(String x) {
    synchronized (this) {
        print(x);
        newLine();
    }
}
```

可以看出在println方法确实是加了锁的，锁的对象就是当前的PrintStream实例对象，而占用这个锁的对象的线程则正在执行下面这个方法，是个本地方法我们看不到底层的细节

```java
/**
 * Writes a sub array as a sequence of bytes.
 * @param b the data to be written
 * @param off the start offset in the data
 * @param len the number of bytes that are written
 * @param append {@code true} to first advance the position to the
 *     end of file
 * @exception IOException If an I/O error has occurred.
 */
private native void writeBytes(byte b[], int off, int len, boolean append)
    throws IOException;
```

无奈，借助搜索引擎，果然查到了同样的问题 

[一个RUNNABLE状态的线程hang在了java.io.FileOutputStream.writeBytes方法上](https://my.oschina.net/u/1030459/blog/908007)

当然原问题是来自[StackOverflow](https://stackoverflow.com/questions/634102/log4j-is-hanging-my-application-what-am-i-doing-wrong) 

问题的根本原因：在CMD窗口点击了黑框之后，控制台就会被暂停![mark](http://static.imlgw.top/blog/20191208/1em5YGsoGyxT.png?imageslim)

，进入编辑模式，之后向控制台的输入内容都会被阻塞，但是正在输出的线程也并不会`BLOCK`，状态仍然是`RUNNABLE`，也就是上面所描述的情况，当你这个时候在CMD状态下按一下回车或者其他的键释放console，退出编辑模式![mark](http://static.imlgw.top/blog/20191208/4RXkNizHfNsP.png?imageslim)

这个线程又会继续往下执行，要解决这个问题可以调整cmd的设置，我这里其实无所谓，因为后面并不会在win上运行

至此问题就基本解决了（其实都不算问题），没想到还会被cmd给坑一把，不过增长了一点排查问题的能力也还是不错的😁