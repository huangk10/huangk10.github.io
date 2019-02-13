---
layout:     post
title:      Multiprocessing多进程(转载)
subtitle:   
date:       2019-02-13
author:     huangk
header-img: 
catalog: true
tags:
    - python
---



## Multiprocessing多进程

>https://cuiqingcai.com/3335.html
>
>http://www.cnblogs.com/kaituorensheng/p/4445418.html

thread多线程库。python中的多线程其实并不是真正的多线程，并不能做到充分利用多核CPU资源。

如果想要充分利用，在python中大部分情况需要使用多进程，那么这个包就叫做 multiprocessing。

借助它，可以轻松完成从单进程到并发执行的转换。multiprocessing支持子进程、通信和共享数据、执行不同形式的同步，提供了Process、Queue、Pipe、Lock等组件。

### Process

#### 基本使用

在multiprocessing中，每一个进程都用一个Process类来表示。

```python
Process([group [, target [, name [, kwargs]]]])
```

+ target表示调用对象，可以传入方法的名字
+ args表示被调用对象的位置参数元祖，比如target是函数a，他有两个参数m，n，那么args就传入(m, n)即可
+ kwargs表示调用对象的字典
+ names是别名，相当于给这个进程取一个名字
+ group分组，实际上不使用

```python
import multiprocessing
import time

def process(num):
    time.sleep(num)
    print('Process:', num)
    
if __name__ == '__main__':
    for i in range(5):
        p = multiprocessing.Process(target=process, args(i,))
        p.start()
    
    print('CPU number:' + str(multiprocess.cpu_count()))
    for p in multiprocessing.active_children():
        print('Child process name:' + p.name + ' id: ' + str(p.pid))
    
    print('Process Ended')
    
# other example
import multiprocessing
import time

def worker_1(interval):
    print "worker_1"
    time.sleep(interval)
    print "end worker_1"

def worker_2(interval):
    print "worker_2"
    time.sleep(interval)
    print "end worker_2"

def worker_3(interval):
    print "worker_3"
    time.sleep(interval)
    print "end worker_3"

if __name__ == "__main__":
    p1 = multiprocessing.Process(target = worker_1, args = (2,))
    p2 = multiprocessing.Process(target = worker_2, args = (3,))
    p3 = multiprocessing.Process(target = worker_3, args = (4,))

    p1.start()
    p2.start()
    p3.start()

    print("The number of CPU is:" + str(multiprocessing.cpu_count()))
    for p in multiprocessing.active_children():
        print("child   p.name:" + p.name + "\tp.id" + str(p.pid))
    print "END!!!!!!!!!!!!!!!!!"
```

调用start()方法即可启动多个进程了。另外你还可以通过 cpu_count() 方法还有 active_children() 方法获取当前机器的 CPU 核心数量以及得到目前所有的运行的进程。

#### 自定义类

可以继承Process类，自定义进程类，实现run方法

```python
from multiprocessing import Process
import time

class MyProcess(Process):
    def __init__(self, loop):
        Process.__init__(self)
        self.loop = loop
    
    def run(self):
        for count in range(self.loop):
            time.sleep(1)
            print('Pid: ' + str(self.pid) + ' LoopCount: ' + str(count))
if __name__ == '__main__':
    for i in range(2, 5):
        p = MyProcess(i)
        p.start()
```

我们可以把一些方法独立的写在每个类里封装好，等用的时候直接初始化一个类运行即可。

### deamon

在这里介绍一个属性，叫做deamon。每个线程都可以单独设置它的属性，如果设置为True，当父进程结束后，子进程会自动被终止。

用一个实例来感受一下，还是原来的例子，增加了deamon属性：

```python
from multiprocessing import Process
import time
 
 
class MyProcess(Process):
    def __init__(self, loop):
        Process.__init__(self)
        self.loop = loop
 
    def run(self):
        for count in range(self.loop):
            time.sleep(1)
            print('Pid: ' + str(self.pid) + ' LoopCount: ' + str(count))
 
 
if __name__ == '__main__':
    for i in range(2, 5):
        p = MyProcess(i)
        p.daemon = True
        p.start()
 
 
    print('Main process Ended!')
```

结果很简单，因为主进程没有做任何事情，直接输出一句话结束，所以在这时也直接终止了子进程的运行。

这样可以有效防止无控制地生成子进程。如果这样写了，你在关闭这个主程序运行时，就无需额外担心子进程有没有被关闭了。

不过这样并不是我们想要达到的效果呀，能不能让所有子进程都执行完了然后再结束呢？那当然是可以的，只需要加入join()方法即可。

```python
from multiprocessing import Process
import time
 
 
class MyProcess(Process):
    def __init__(self, loop):
        Process.__init__(self)
        self.loop = loop
 
    def run(self):
        for count in range(self.loop):
            time.sleep(1)
            print('Pid: ' + str(self.pid) + ' LoopCount: ' + str(count))
 
 
if __name__ == '__main__':
    for i in range(2, 5):
        p = MyProcess(i)
        p.daemon = True
        p.start()
        p.join()
 
 
    print('Main process Ended!')
```

在这里，每个子进程都调用了join()方法，这样父进程（主进程）就会等待子进程执行完毕。

### Lock

那这归根结底是因为线程同时资源（输出操作）而导致的。

那怎么来避免这种问题？那自然是在某一时间，只能一个进程输出，其他进程等待。等刚才那个进程输出完毕之后，另一个进程再进行输出。这种现象就叫做“互斥”。

我们可以通过 Lock 来实现，在一个进程输出时，加锁，其他进程等待。等此进程执行结束后，释放锁，其他进程可以进行输出。

我们现用一个实例来感受一下：

```python
from multiprocessing import Process, Lock
import time
 
 
class MyProcess(Process):
    def __init__(self, loop, lock):
        Process.__init__(self)
        self.loop = loop
        self.lock = lock
 
    def run(self):
        for count in range(self.loop):
            time.sleep(0.1)
            #self.lock.acquire()
            print('Pid: ' + str(self.pid) + ' LoopCount: ' + str(count))
            #self.lock.release()
 
if __name__ == '__main__':
    lock = Lock()
    for i in range(10, 15):
        p = MyProcess(i, lock)
        p.start()
```

我们在print方法的前后分别添加了获得锁和释放锁的操作。这样就能保证在同一时间只有一个print操作。

### Semaphone

信号量，是在进程同步过程中一个比较重要的角色。可以控制临界资源的数量，保证各个进程之间的互斥和同步。

```python
from multiprocessing import Process, Semaphore, Lock, Queue
import time
 
buffer = Queue(10)
empty = Semaphore(2)
full = Semaphore(0)
lock = Lock()
 
class Consumer(Process):
 
    def run(self):
        global buffer, empty, full, lock
        while True:
            full.acquire()
            lock.acquire()
            buffer.get()
            print('Consumer pop an element')
            time.sleep(1)
            lock.release()
            empty.release()
 
 
class Producer(Process):
    def run(self):
        global buffer, empty, full, lock
        while True:
            empty.acquire()
            lock.acquire()
            buffer.put(1)
            print('Producer append an element')
            time.sleep(1)
            lock.release()
            full.release()
 
 
if __name__ == '__main__':
    p = Producer()
    c = Consumer()
    p.daemon = c.daemon = True
    p.start()
    c.start()
    p.join()
    c.join()
    print('Ended!')
```

### Pool

在利用Python进行系统管理的时候，特别是同时操作多个文件目录，或者远程控制多台主机，并行操作可以节约大量的时间。当被操作对象数目不大时，可以直接利用multiprocessing中的Process动态成生多个进程，十几个还好，但如果是上百个，上千个目标，手动的去限制进程数量却又太过繁琐，此时可以发挥进程池的功效。
Pool可以提供指定数量的进程，供用户调用，当有新的请求提交到pool中时，如果池还没有满，那么就会创建一个新的进程用来执行该请求；但如果池中的进程数已经达到规定最大值，那么该请求就会等待，直到池中有进程结束，才会创建新的进程来它。

#### 使用进程池（非阻塞）

```python
#coding: utf-8
import multiprocessing
import time

def func(msg):
    print "msg:", msg
    time.sleep(3)
    print "end"

if __name__ == "__main__":
    pool = multiprocessing.Pool(processes = 3)
    for i in xrange(4):
        msg = "hello %d" %(i)
        pool.apply_async(func, (msg, ))   #维持执行的进程总数为processes，当一个进程执行完毕后会添加新的进程进去

    print "Mark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~"
    pool.close()
    pool.join()   #调用join之前，先调用close函数，否则会出错。执行完close后不会有新的进程加入到pool,join函数等待所有子进程结束
    print "Sub-process(es) done."
```

函数解释：

- apply_async(func[, args[, kwds[, callback]]]) 它是**非阻塞**，apply(func[, args[, kwds]])是**阻塞**的（理解区别，看例1例2结果区别）
- close()    关闭pool，使其不在接受新的任务。
- terminate()    结束工作进程，不在处理未完成的任务。
- join()    主进程阻塞，等待子进程的退出， join方法要在close或terminate之后使用。

执行说明：创建一个进程池pool，并设定进程的数量为3，xrange(4)会相继产生四个对象[0, 1, 2, 4]，四个对象被提交到pool中，因pool指定进程数为3，所以0、1、2会直接送到进程中执行，当其中一个执行完事后才空出一个进程处理对象3，所以会出现输出“msg: hello 3”出现在"end"后。因为为非阻塞，主函数会自己执行自个的，不搭理进程的执行，所以运行完for循环后直接输出“mMsg: hark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~”，主程序在pool.join（）处等待各个进程的结束。

#### 使用进程池（阻塞）

```python
#coding: utf-8
import multiprocessing
import time

def func(msg):
    print "msg:", msg
    time.sleep(3)
    print "end"

if __name__ == "__main__":
    pool = multiprocessing.Pool(processes = 3)
    for i in xrange(4):
        msg = "hello %d" %(i)
        pool.apply(func, (msg, ))   #维持执行的进程总数为processes，当一个进程执行完毕后会添加新的进程进去

    print "Mark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~"
    pool.close()
    pool.join()   #调用join之前，先调用close函数，否则会出错。执行完close后不会有新的进程加入到pool,join函数等待所有子进程结束
    print "Sub-process(es) done."
```

