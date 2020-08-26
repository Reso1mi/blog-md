---
title: 
 Golang踩坑：exec执行任务取消后不退出
tags: 
  [开源项目,Golang]
categories:
	[踩坑记录]
---

## 背景
在做项目的过程中利用os/exec包执行一些shell脚本，调试过程中发现我取消了context后go进程仍然阻塞不退出
## 分析
> go version go1.13.6 linux/amd64

在实现kill强杀功能时候发现的问题，无法杀死任务，即使kill了还是会等到任务执行完才会返回，在查资料的过程中发现这应该也算是golang本身的一个坑了，参考[issue23019](https://github.com/golang/go/issues/23019)

一开始是在Windows平台上运行测试的，以为是平台的原因但是在切换了Linux后问题依然存在，如下poc既可复现

```golang
package main

import (
	"context"
	"fmt"
	"os/exec"
	"time"
)

func main() {
	ctx, cancelFn := context.WithTimeout(context.Background(), time.Second*5)
	defer cancelFn()
	cmd := exec.CommandContext(ctx, "/bin/bash", "-c", "sleep 120; echo hello")
	output, err := cmd.CombinedOutput()
	fmt.Printf("output：【%s】err:【%s】", string(output), err)
}
```
在linux平台上通过go run执行上面的代码，然后`pstree -p`查看进程树
```golang
//由于进程树比较庞大，所以省略了一些无关的部分
systemd(1)─┬─...
           ├─sshd(17395)─┬─sshd(18450)───sftp-server(18453)
           │             ├─sshd(28684)───zsh(28687)───go(2661)─┬─demo3_cmdPit(2680)─┬─bash(2683)───sleep(2684)
           │             │                                     │                    ├─{demo3_cmdPit}(2681)
           │             │                                     │                    ├─{demo3_cmdPit}(2682)
           │             │                                     │                    └─{demo3_cmdPit}(2686)
           │             │                                     ├─{go}(2662)
           │             │                                     ├─{go}(2663)
           │             │                                     ├─{go}(2664)
           │             │                                     ├─{go}(2665)
           │             │                                     ├─{go}(2672)
           │             │                                     ├─{go}(2679)
           │             │                                     └─{go}(2685)
           │             └─sshd(29042)───zsh(29044)───pstree(2688)
```
可以看到这里我们的go进程`demo3_cmdPit(2680)`创建了`bash(2683)`进程去执行具体的shell指令，但是这里由于我们执行的命令是【`sleep 50; echo hello`】属于一组多条命令，所以这里shell会fork出子进程去执行这些命令，所以上面显示`sleep(2684)`是`bash(2683)`的子进程
>补充：除了多条命令会产生子进程外，还有一些情况会产生子进程，比如`&`后台运行，`pipe`管道，外部shell脚本等等，具体可以参考这篇文章[子shell以及什么时候进入子shell](https://www.cnblogs.com/f-ck-need-u/p/7446194.html)

然后再5s后ctx到期cancel后我们再次查看进程树
```golang
systemd(1)─┬─...
           ├─sleep(2684)
           ├─sshd(17395)─┬─sshd(18450)───sftp-server(18453)
           │             ├─sshd(28684)───zsh(28687)───go(2661)─┬─demo3_cmdPit(2680)─┬─{demo3_cmdPit}(2681)
           │             │                                     │                    ├─{demo3_cmdPit}(2682)
           │             │                                     │                    └─{demo3_cmdPit}(2686)
           │             │                                     ├─{go}(2662)
           │             │                                     ├─{go}(2663)
           │             │                                     ├─{go}(2664)
           │             │                                     ├─{go}(2665)
           │             │                                     ├─{go}(2672)
           │             │                                     ├─{go}(2679)
           │             │                                     └─{go}(2685)
           │             └─sshd(29042)───zsh(29044)───pstree(2708)
```
可以看到`bash(2683)`进程确实被kill了，但是它的子进程确并没有结束，而是变成了**孤儿进程**，被`init 1(systemd)`收养，但是我们看到这时进程`demo3_cmdPit(2680)`还没有退出，符合之前的预测，这里该进程仍然还有3条线程（协程）没有退出，直到sleep命令执行完才会返回，这里通过pstack是无法查看这几条线程的堆栈的，因为操作系统并不认识协程，而且G会在P上切换，所以我们暂时也不知道这几条线程在干嘛

>孤儿进程：父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。孤儿进程将被init进程(进程号为1)所收养，并由init进程对它们完成状态收集工作。
>
>僵尸进程：父进程使用fork创建子进程，如果子进程退出，而父进程并没有调用`wait`或`waitpid`获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中。这种进程称之为僵死进程。

当sleep执行完成后，程序会正常返回如下信息
```go
output：【】err:【signal: killed】#
```
因为是在Linux环境下没有IDE而且不太熟悉`gdb`所以这里我们用`pprof`查看了一下堆栈
```golang
import (
	"context"
	"fmt"
	"net/http"
	_ "net/http/pprof"
	"os/exec"
	"time"
)

func main() {
	go func() {
		err := http.ListenAndServe(":6060", nil)
		if err != nil {
			fmt.Printf("failed to start pprof monitor:%s", err)
		}
	}()
	ctx, cancelFn := context.WithTimeout(context.Background(), time.Second*5)
	defer cancelFn()
	cmd := exec.CommandContext(ctx, "/bin/bash", "-c", "sleep 50; echo hello")
	output, err := cmd.CombinedOutput()
	fmt.Printf("output：【%s】err:【%s】", string(output), err)
}
```
> 后面了解到其实直接发送`SIGQUIT`信号也可以看到堆栈，如 `kill -SIGQUIT <pid>`，go的工具链还是非常完善的，除了这些还有很多方法看堆栈


```go
curl http://127.0.0.1:6060/debug/pprof/goroutine\?debug\=2
...
goroutine 1 [chan receive]:
os/exec.(*Cmd).Wait(0xc000098580, 0x0, 0x0)
        /usr/lib/golang/src/os/exec/exec.go:509 +0x125
os/exec.(*Cmd).Run(0xc000098580, 0xc00006d710, 0xc000098580)
        /usr/lib/golang/src/os/exec/exec.go:341 +0x5c
os/exec.(*Cmd).CombinedOutput(0xc000098580, 0x9, 0xc0000bff30, 0x2, 0x2, 0xc000098580)
        /usr/lib/golang/src/os/exec/exec.go:561 +0x91
main.main()
		/usr/local/gotest/cmd/demo3_cmdPit/main.go:22 +0x176
...
goroutine 9 [IO wait]:
internal/poll.runtime_pollWait(0x7fa96a0f5f08, 0x72, 0xffffffffffffffff)
        /usr/lib/golang/src/runtime/netpoll.go:184 +0x55
internal/poll.(*pollDesc).wait(0xc000058678, 0x72, 0x201, 0x200, 0xffffffffffffffff)
        /usr/lib/golang/src/internal/poll/fd_poll_runtime.go:87 +0x45
internal/poll.(*pollDesc).waitRead(...)
        /usr/lib/golang/src/internal/poll/fd_poll_runtime.go:92
internal/poll.(*FD).Read(0xc000058660, 0xc0000e2000, 0x200, 0x200, 0x0, 0x0, 0x0)
        /usr/lib/golang/src/internal/poll/fd_unix.go:169 +0x1cf
os.(*File).read(...)
        /usr/lib/golang/src/os/file_unix.go:259
os.(*File).Read(0xc000010088, 0xc0000e2000, 0x200, 0x200, 0x7fa96a0f5ff8, 0x0, 0xc000039ea0)
        /usr/lib/golang/src/os/file.go:116 +0x71
bytes.(*Buffer).ReadFrom(0xc00006d710, 0x8b6120, 0xc000010088, 0x7fa96a0f5ff8, 0xc00006d710, 0x1)
        /usr/lib/golang/src/bytes/buffer.go:204 +0xb4
io.copyBuffer(0x8b5a20, 0xc00006d710, 0x8b6120, 0xc000010088, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0)
        /usr/lib/golang/src/io/io.go:388 +0x2ed
io.Copy(...)
        /usr/lib/golang/src/io/io.go:364
os/exec.(*Cmd).writerDescriptor.func1(0x0, 0x0)
        /usr/lib/golang/src/os/exec/exec.go:311 +0x63
os/exec.(*Cmd).Start.func1(0xc000098580, 0xc00000e3a0)
        /usr/lib/golang/src/os/exec/exec.go:435 +0x27
created by os/exec.(*Cmd).Start
        /usr/lib/golang/src/os/exec/exec.go:434 +0x608
```
这里我忽略了一些无关的协程，留下了这两条的堆栈信息

首先我们看一下第一条的堆栈信息，很明显这条就是我们的主`goroutine`，可以看到它正处于`[chan receive]`状态，也就是在等待某个chan传递信息，根据堆栈信息可以看到问题出在

 `/usr/lib/golang/src/os/exec/exec.go:509`

我们去看一下这里发生了什么

![mark](http://static.imlgw.top/blog/20200821/l2wEHHBXyTTh.png?imageslim)
果然这里在等待`c.errch`，那么这个errch是谁负责推送的呢？

我们再来看看另一条`goroutine`，这条`goroutine`处于`【IO wait】`状态，说明IO发生阻塞了，我们从下往上看堆栈，首先看一下是在哪里启动了这个协程
```go
created by os/exec.(*Cmd).Start
        /usr/lib/golang/src/os/exec/exec.go:434 +0x608
```
![mark](http://static.imlgw.top/blog/20200822/sh8qmlcDeD64.png?imageslim)
可以看到这个协程的作用就是向我们上面的主`goroutine`在等待的`c.errch`推送消息，那么这里`fn()`为什么没有返回呢？这个fn是在干嘛？继续翻一翻源码，发现这些函数`fn()`是在cmd.Start的时候创建的，用于处理shell的标准输入，输出和错误输出，而我们的程序就是卡在了这里，继续跟进上面的堆栈信息
```go
os/exec.(*Cmd).writerDescriptor.func1(0x0, 0x0)
        /usr/lib/golang/src/os/exec/exec.go:311 +0x63
```
![UTOOLS1598032678130.png](https://upload.cc/i1/2020/08/22/ls251C.png)
可以看到这里就是具体协程执行的任务，这里有一个IO操作，可想而知程序就是阻塞在了这里，再往下分析就是内部epoll的代码了，就不往下了，我们分析下这段代码的意义，首先创建了管道`Pipe`，然后在协程中将读端`pr`的数据copy到`w`中，很明显这里就是`goroutine`通过pipe读取子进程的输出，但是由于某些原因`Pipe`阻塞了，无法从读端获取数据

我们用`lsof -p`看一下该进程打开的资源
```go
$ lsof -p 27301
COMMAND   PID USER   FD      TYPE    DEVICE SIZE/OFF      NODE NAME
main    27301 root  cwd       DIR     253,1     4096   1323417 /usr/local/gotest/cmd/demo3_cmdPit
main    27301 root  rtd       DIR     253,1     4096         2 /
main    27301 root  txt       REG     253,1  7647232    401334 /tmp/go-build043247660/b001/exe/main
main    27301 root  mem       REG     253,1  2151672   1050229 /usr/lib64/libc-2.17.so
main    27301 root  mem       REG     253,1   141968   1050255 /usr/lib64/libpthread-2.17.so
main    27301 root  mem       REG     253,1   163400   1050214 /usr/lib64/ld-2.17.so
main    27301 root    0u      CHR     136,0      0t0         3 /dev/pts/0
main    27301 root    1u      CHR     136,0      0t0         3 /dev/pts/0
main    27301 root    2u      CHR     136,0      0t0         3 /dev/pts/0
main    27301 root    3u     IPv6 170355813      0t0       TCP *:6060 (LISTEN)
main    27301 root    4u  a_inode      0,10        0      5074 [eventpoll]
main    27301 root    5r     FIFO       0,9      0t0 170355794 pipe
```
可以看到确实打开了一个inode为170355794的管道，结合前面的分析，我们的shell在执行的时候`fork()`创建了子进程，但是kill的时候只kill了shell进程，而在`fork()`的时候会同时将当前进程的`pipe`的`fd[2]`描述符也复制过去，这就造成了`pipe`的写入端`fd[1]`被子进程继续持有（文件表引用计数不为0），而`goroutine`也会继续等待pipe写入端写入或者关闭，直到`sleep 50`执行完后才返回
## 原因总结
`golang`的`os/exec`包在和shell做交互的时候会和shell之间建立pipe，用于输入输出，获取shell返回值，但是在某些情况下，我们的shell会fork出子进程去执行命令，比如多条语句，&后台执行，sh脚本等，然而这里ctx过期后，仅仅kill了shell进程，并没有kill它所创建的子进程，这就导致了pipe的fd仍然被子进程持有无法关闭，而在`os/exec`的实现中`goroutine`会一直等待pipe关闭后才返回，进而导致了该问题的产生
## 解决方案
问题搞清楚了，就好解决了。在go的实现中只kill了shell进程，并不会kill子进程，进而引发了上面的问题
```golang
// Kill causes the Process to exit immediately. Kill does not wait until
// the Process has actually exited. This only kills the Process itself,
// not any other processes it may have started.
func (p *Process) Kill() error {
	return p.kill()
}
```
这里我们可以通过kill`进程组`的方式来解决，所谓进程组也就是父进程和其创建的子进程的集合，`PGID`就是进程组的ID，每个进程组都有进程组ID，这里我们可以通过`ps -axjf`命令查看一下
```go
PPID   PID  PGID   SID TTY      TPGID STAT   UID   TIME COMMAND
17395 20937 20937 20937 ?           -1 Ss       0   0:00  \_ sshd: root@pts/0
20937 20939 20939 20939 pts/0    25879 Ss       0   0:00  |   \_ -zsh
20939 25879 25879 20939 pts/0    25879 Sl+      0   0:00  |       \_ go run main.go
25879 25902 25879 20939 pts/0    25879 Sl+      0   0:00  |           \_ /tmp/go-build486839449/b001/exe/main
25902 25906 25879 20939 pts/0    25879 S+       0   0:00  |               \_ /bin/bash -c sleep 50; echo hello
25906 25907 25879 20939 pts/0    25879 S+       0   0:00  |                   \_ sleep 50
```
在知道`PGID`后可以根据这个ID直接Kill掉所有相关的进程，但是这里注意看上面，我们的`Shell`进程和它的子进程，以及我们`go进程`都是在一个进程组内的，直接kill会把我们的go进程也杀死，所以这里我们要让`Shell`进程和它的子进程额外开辟一个新的进程组，然后再kill

> 下面的代码中使用到了一些`syscall`包的方法，这个包的实现平台之间是有差异的，下面的代码只适用于Linux环境下

```golang
func RunCmd(ctx context.Context, cmd *exec.Cmd) ([]byte, error) {
	var (
		b   bytes.Buffer
		err error
	)
	//开辟新的线程组（Linux平台特有的属性）
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Setpgid: true, //使得Shell进程开辟新的PGID,即Shell进程的PID,它后面创建的所有子进程都属于该进程组
	}
	cmd.Stdout = &b
	cmd.Stderr = &b
	if err = cmd.Start(); err != nil {
		return nil, err
	}
	var finish = make(chan struct{})
	defer close(finish)
	go func() {
		select {
		case <-ctx.Done(): //超时/被cancel 结束
			//kill -(-PGID)杀死整个进程组
			syscall.Kill(-cmd.Process.Pid, syscall.SIGKILL)
		case <-finish: //正常结束
		}
	}()
	//wait等待goroutine执行完，然后释放FD资源
	//这个时候再kill掉shell进程就不会再等待了，会直接返回
	if err = cmd.Wait(); err != nil {
		return nil, err
	}
	return b.Bytes(), err
}
```
**效果演示**
![mark](http://static.imlgw.top/blog/20200825/gfz2xQ7nnKKg.gif)