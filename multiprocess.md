
Knowledges:

1. `*Unix`进程与线程
2. `OS`模块
3. `multiprocessing` 模块
4. 进程池
5. 进程间通信
6. 进程同步
7. 进程共享变量
8. 多进程的好处与弊端


# 一. Unix进程与线程

在之前写过这篇：[Python多线程编程(1)：基础](http://beginman.cn/python/2015/04/09/Python-learn9/)， 摘取点知识：

进程是计算机中已运行程序的实体，是程序的基本执行实体，是线程的容器。它有两种运行方式：同步（循序）和异步（平行）。

多任务的实现有3种方式：

- 多进程模式；
- 多线程模式；
- 多进程+多线程模式。

# 二. 多进程之os模块

>Unix/Linux操作系统提供了一个`fork()`系统调用，它非常特殊。普通的函数调用，调用一次，返回一次，但是`fork()`调用一次，返回两次，因为操作系统自动把当前进程（称为父进程）复制了一份（称为子进程），然后，分别在父进程和子进程内返回。

>子进程永远返回0，而父进程返回子进程的ID。这样做的理由是，一个父进程可以fork出很多子进程，所以，父进程要记下每个子进程的ID，而子进程只需要调用`getppid()`就可以拿到父进程的ID。

Python的`os`模块封装了常见的系统调用，其中就包括fork，可以在Python程序中轻松创建子进程：

    # multiprocessing.py
    import os

    print 'Process (%s) start...' % os.getpid()
    pid = os.fork()
    if pid==0:
        print 'I am child process (%s) and my parent is %s.' % (os.getpid(), os.getppid())
    else:
        print 'I (%s) just created a child process (%s).' % (os.getpid(), pid)


运行结果如下：

    Process (876) start...
    I (876) just created a child process (877).
    I am child process (877) and my parent is 876.

有了fork调用，一个进程在接到新任务时就可以复制出一个子进程来处理新任务，常见的Apache服务器就是由父进程监听端口，每当有新的http请求时，就fork出子进程来处理新的http请求。

在os模块的进程选项中([15.1.1. Process Parameters](https://docs.python.org/2/library/os.html#process-parameters))

# 三.multiprocessing模块
由于Windows没有fork调用，[multiprocessing模块](https://docs.python.org/2/library/multiprocessing.html)就是跨平台版本的多进程模块。

## 1.创建进程

    p = Process(name="processName", target=func, args=('arg1', ))
    p.start()
    p.join()


- Process类来代表一个进程对象
- `start()`方法启动线程
- `join()`方法等待子进程结束

## 2. 守护进程
    
    p.daemon = True
    p.start()
    p.join(1)
    ...等待n秒
    p.is_alive()

## 3. 终止进程
使用`terminate()`, **注意 terminate之后要join，使其可以更新状态**

    import multiprocessing
    import time

    def slow_worker():
        print 'Starting worker'
        time.sleep(0.1)
        print 'Finished worker'

    if __name__ == '__main__':
        p = multiprocessing.Process(target=slow_worker)
        print 'BEFORE:', p, p.is_alive()

        p.start()
        print 'DURING:', p, p.is_alive()

        p.terminate()
        print 'TERMINATED:', p, p.is_alive()

        p.join()
        print 'JOINED:', p, p.is_alive()

out:

    BEFORE: <Process(Process-1, initial)> False
    DURING: <Process(Process-1, started)> True
    TERMINATED: <Process(Process-1, started)> True
    JOINED: <Process(Process-1, stopped[SIGTERM])> False

## 4. 进程的退出状态

0,ok;其他数字，则错误

    import multiprocessing
    import sys
    import time

    def exit_error():
        sys.exit(1)

    def exit_ok():
        return

    def return_value():
        return 1

    def raises():
        raise RuntimeError('There was an error!')

    def terminated():
        time.sleep(3)

    if __name__ == '__main__':
        jobs = []
        for f in [exit_error, exit_ok, return_value, raises, terminated]:
            print 'Starting process for', f.func_name
            j = multiprocessing.Process(target=f, name=f.func_name)
            jobs.append(j)
            j.start()

        jobs[-1].terminate()

        for j in jobs:
            j.join()
            print '%15s.exitcode = %s' % (j.name, j.exitcode)


## 5. 日志

    import multiprocessing
    import logging
    import sys

    def worker():
        print 'Doing some work'
        sys.stdout.flush()

    if __name__ == '__main__':
        multiprocessing.log_to_stderr()
        logger = multiprocessing.get_logger()
        logger.setLevel(logging.INFO)
        p = multiprocessing.Process(target=worker)
        p.start()
        p.join()

out:

    Doing some work
    [INFO/Process-1] child process calling self.run()
    [INFO/Process-1] process shutting down
    [INFO/Process-1] process exiting with exitcode 0
    [INFO/MainProcess] process shutting down

## 四.进程池
进程池的出现就是为了解决大量进程管理的问题。`Pool`可以提供指定数量的进程，供用户调用，当有新的请求提交到pool中时，如果池还没有满，那么就会创建一个新的进程用来执行该请求；但如果池中的进程数已经达到规定最大值，那么该请求就会等待，直到池中有进程结束，才会创建新的进程来它。如下例子：

    import multiprocessing
    import time


    def f(x):
        print 'process:%s' % x
        time.sleep(1)

    if __name__ == '__main__':
        pool = multiprocessing.Pool(processes=3)
        process = []
        for i in range(10):
            res = pool.apply_async(f, (i, ))
            process.append(res)

        pool.close()
        pool.join()
        for i in process:
            print i, i.successful()


释义：

- `Pool(processes=3)`: 创建大小为3的线程池
- `pool.apply_async()`: 向进程池提交目标请求和参数
- `pool.close()`: 等待池中的worker进程执行结束再关闭pool, `terminate()`则是直接关闭
- `pool.join()`: 是用来等待进程池中的worker进程执行完毕，防止主进程在worker进程结束前结束。pool.join()必须使用在pool.close()或者pool.terminate()之后
- `successful()`: 整个调用执行的状态(成功则True)，如果还有worker没有执行完，则会抛出AssertionError异常。

在执行 ` ps aux | grep pool.py` 结果如下：

    fangpeng        74265   0.0  0.0  2466780   1324   ??  S     2:07下午   0:00.01 /usr/bin/python /Users/fangpeng/project/mqtt_demo/muti_process/produce.py
    fangpeng        74264   0.0  0.0  2466780   1392   ??  S     2:07下午   0:00.01 /usr/bin/python /Users/fangpeng/project/mqtt_demo/muti_process/produce.py
    fangpeng        74263   0.0  0.0  2466780   1316   ??  S     2:07下午   0:00.01 /usr/bin/python /Users/fangpeng/project/mqtt_demo/muti_process/produce.py
    fangpeng        74262   0.0  0.1  2479104   4596   ??  S     2:07下午   0:00.15 /usr/bin/python /Users/fangpeng/project/mqtt_demo/muti_process/produce.py

子进程始终是3个。

程序输出：

    process:0
    process:1
    process:2
    process:3
    process:4
    process:5
    process:6
    process:7
    process:8
    process:9
    <multiprocessing.pool.ApplyResult object at 0x105196090> True
    <multiprocessing.pool.ApplyResult object at 0x105196110> True
    <multiprocessing.pool.ApplyResult object at 0x1051961d0> True
    <multiprocessing.pool.ApplyResult object at 0x105196290> True
    <multiprocessing.pool.ApplyResult object at 0x105196350> True
    <multiprocessing.pool.ApplyResult object at 0x105196410> True
    <multiprocessing.pool.ApplyResult object at 0x105196510> True
    <multiprocessing.pool.ApplyResult object at 0x105196590> True
    <multiprocessing.pool.ApplyResult object at 0x105196650> True
    <multiprocessing.pool.ApplyResult object at 0x105196710> True

虽然进程池大小3，则在for 依次传入时，通过输出看出来，基本每3个为一组合。0，1，2是立刻执行的，而 3要等待前面某个task完成后才执行。**Pool的默认大小是CPU的核数**，如果拥有8核CPU，你要提交至少9个子进程才能看到上面的等待效果。

## 五.进程间通信
进程间通信方式有：

1. 管道( pipe )：管道是一种半双工的通信方式，数据只能单向流动，通常在父子进程关系的进程间使用.可参考[First Head C 进程间通信](https://github.com/BeginMan/BookNotes/blob/master/C/top10.md)
2. 有名管道(named pipe): 允许非父子进程关系之间的通信
3. 信号量
4. 信号 ( sinal ) 
5. 消息队列( message queue ) ： 消息队列是由消息的链表，存放在内核中并由消息队列标识符标识。消息队列克服了信号传递信息少、管道只能承载无格式字节流以及缓冲区大小受限等缺点。
6. 共享内存( shared memory ) ：共享内存就是映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但多个进程都可以访问。共享内存是最快的 IPC 方式，它是针对其他进程间通信方式运行效率低而专门设计的。
7. 套接字( socket )

Python的multiprocessing模块包装了底层的机制，提供了`Queue`、`Pipes`等多种方式来交换数据。

## 5.1 pipe
`pipe()`返回一对连接对象，代表了pipe的两端。每个对象都有`send()`和`recv()`方法。

    from multiprocessing import Process, Pipe

    def f(conn):
        conn.send(['pipe', {"name": "bm"}, None])   # send 多样性
        conn.close()

    if __name__ == '__main__':
        parent_conn, child_conn = Pipe()
        p = Process(target=f, args=(child_conn, ))
        p.start()
        p.join()

        print parent_conn.recv()

Pipe可以是单向(half-duplex)，也可以是双向(duplex)。`Pipe(duplex=False)` 创建单向管道 (默认为双向)。一个进程从PIPE一端输入对象，然后被PIPE另一端的进程接收，单向管道只允许管道一端的进程输入，而双向管道则允许从两端输入。

## 5.2 Queue & JoinableQueue

`mutiprocessing.Queue(maxsize)`创建，maxsize表示队列中可以存放对象的最大数量。

`Queue`用来在进程间传递消息，任何可以pickle-able的对象都可以在加入到queue。 

`JoinableQueue` 是 `Queue`的子类，增加了`task_done()`和`join()`方法。

- `task_done()`用来告诉queue一个task完成。一般地在调用`get()`获得一个task，在task结束后调用`task_done()`来通知Queue当前task完成。
- `join()` 阻塞直到queue中的所有的task都被处理（即task_done方法被调用）。

下面一个简单例子：

    # coding=utf-8
    __desc__ = '我们以Queue为例，在父进程中创建两个子进程，一个往Queue里写数据，一个从Queue里读数据'

    from multiprocessing import Process, Queue
    import time


    # 写数据进程执行的代码:
    def wirte(q):
        for i in ['E', 'X', 'O', 'S', 'B']:
            print 'Put %s to queue...' % i
            q.put(i)        # put 写入
            time.sleep(1)   # 假装花费很长时间做什么事，这里主要看输出效果，不至于唰一下子全输出了


    # 读数据进程执行的代码:
    def read(q):
        while 1:
            print 'Get %s from queue.' % q.get(True)

    if __name__ == '__main__':
        # 父进程创建Queue，并传给各个子进程：
        queue = Queue()     # 创建默认大小(cpu num) 队列
        pw = Process(target=wirte, args=(queue, ))      # write process
        pr = Process(target=read, args=(queue, ))       # read process

        pw.start()      # 启动子进程pw，写入:
        pr.start()      # 启动子进程pr，读取:

        pw.join()       # 等待pw结束
        pr.terminate()  # pr进程里是死循环，无法等待其结束，只能强行终止:


输出：

    Put E to queue...
    Get E from queue.
    Put X to queue...
    Get X from queue.
    Put O to queue...
    Get O from queue.
    Put S to queue...
    Get S from queue.
    Put B to queue...
    Get B from queue.


下面一个例子：

    # coding=utf-8
    __date__ = '15/10/12'
    __desc__ = 'do something...'

    import multiprocessing
    import time


    class Consumer(multiprocessing.Process):
        """从Process派生
            消费者
        """
        def __init__(self, task_queue, result_queue):
            multiprocessing.Process.__init__(self)
            self.task_queue = task_queue
            self.result_queue = result_queue

        def run(self):
            proc_name = self.name
            while True:
                next_task = self.task_queue.get()
                if next_task is None:
                    # None来表示task处理完毕。
                    print ('%s: Exiting' % proc_name)
                    self.task_queue.task_done()         # 通知Queue当前task完成
                    break
                print '%s, %s' % (proc_name, next_task)
                answer = next_task()        # __call__()
                self.task_queue.task_done()
                self.result_queue.put(answer)           # Task执行结果入result_queue队列
            return


    class Task(object):
        def __init__(self, a, b):
            self.a = a
            self.b = b

        def __call__(self, *args, **kwargs):
            time.sleep(2)     # 假装需要一段时间做的工作
            return "%s * %s = %s" % (self.a, self.b, self.a * self.b)

        def __str__(self):
            return '%s * %s' % (self.a, self.b)

    if __name__ == '__main__':
        # 建立通信队列
        tasks = multiprocessing.JoinableQueue()
        results = multiprocessing.Queue()

        # 开始消费者
        num_consumers = multiprocessing.cpu_count()         # CPU个数统计
        print ('Creating %d consumers' % num_consumers)
        consumers = [Consumer(tasks, results)
                     for i in range(num_consumers)]
        for w in consumers:
            w.start()

        # 入队作业，开始生产
        num_jobs = 10
        for i in range(num_jobs):
            tasks.put(Task(i, i))

        # Add a poison pill for each consumer
        for i in range(num_consumers):
            tasks.put(None)

        # Wait for all of the tasks to finish
        tasks.join()

        # Start printing results
        while num_jobs:
            result = results.get()
            print ('Result:', result)
            num_jobs -= 1


输出：

    Creating 4 consumers
    Consumer-2, 0 * 0
    Consumer-1, 1 * 1
    Consumer-3, 2 * 2
    Consumer-4, 3 * 3
    Consumer-3, 4 * 4
    Consumer-2, 5 * 5
    Consumer-1, 6 * 6
    Consumer-4, 7 * 7
    Consumer-2, 8 * 8
    Consumer-3, 9 * 9
    Consumer-1: Exiting
    Consumer-4: Exiting
    Consumer-2: Exiting
    Consumer-3: Exiting
    ('Result:', '2 * 2 = 4')
    ('Result:', '0 * 0 = 0')
    ('Result:', '1 * 1 = 1')
    ('Result:', '3 * 3 = 9')
    ('Result:', '4 * 4 = 16')
    ('Result:', '6 * 6 = 36')
    ('Result:', '5 * 5 = 25')
    ('Result:', '7 * 7 = 49')
    ('Result:', '8 * 8 = 64')
    ('Result:', '9 * 9 = 81')


**注意：**

queue和pipe的区别： pipe用来在两个进程间通信。queue用来在多个进程间实现通信。 


# 六. 进程同步
基础知识可参考：[操作系统(计算机)进程和线程管理](http://c.biancheng.net/cpp/u/xitong_2/), 相关概念有：

- **临界资源**： 一次仅允许一个进程使用的资源称为临界资源。许多物理设备都属于临界资源，如打印机等。此外，还有许多变量、数据等都可以被若干进程共享，也属于临界资源。
- **同步**： 同步亦称直接制约关系，它是指为完成某种任务而建立的两个或多个进程，这些进程因为需要在某些位置上协调它们的工作次序而等待、传递信息所产生的制约关系。进程间的直接制约关系就是源于它们之间的相互合作。例如，输入进程A通过单缓冲向进程B提供数据。当该缓冲区空时，进程B不能获得所需数据而阻塞，一旦进程A将数据送入缓冲区，进程B被唤醒。反之，当缓冲区满时，进程A被阻塞，仅当进程B取走缓冲数据时，才唤醒进程A。
- **互斥**： 互斥亦称间接制约关系。当一个进程进入临界区使用临界资源时，另一个进程必须等待, 当占用临界资源的进程退出临界区后，另一进程才允许去访问此临界资源。例如，在仅有一台打印机的系统中，有两个进程A和进程B，如果进程A需要打印时, 系统已将打印机分配给进程B,则进程A必须阻塞。一旦进程B将打印机释放，系统便将进程A唤醒，并将其由阻塞状态变为就绪状态。

**Python进程同步的方法基本与多线程相同。**

Python多进程同步方法有 Lock、Semaphore、Event实例，

- Lock,当多个进程需要访问共享资源的时候,lock避免访问冲突
- Semaphore用来控制对共享资源的访问数量
- Event用来实现进程间同步通信

## 6.1 Lock

    lock.acquire()
        try:
            fs = open(f,"a+")
            fs.write('Lock acquired directly\n')
            fs.close()
        finally:
            lock.release()

## 6.2 Semaphore

Semaphore用来控制对共享资源的访问数量，例如池的最大连接数。


    import multiprocessing
    import time 

    def worker(s,i):
        s.acquire()
        print(multiprocessing.current_process().name + " acquire")
        time.sleep(i)
        print(multiprocessing.current_process().name + " release")
        s.release()

    if __name__ == "__main__":
      
        s = multiprocessing.Semaphore(2)
        for i in range(5):
            p = multiprocessing.Process(target=worker, args=(s,i*2))
            p.start()


上面的实例中使用semaphore限制了最多有2个进程同时执行。

## 6.3 Event

    import multiprocessing
    import time

    def wait_for_event(e):
        """Wait for the event to be set before doing anything"""
        print ('wait_for_event: starting')
        e.wait()        # 如果wait(2)则在main下time.sleep(3)后就超时了，此时e.is_set()为False.
        print ('wait_for_event: e.is_set()->' + str(e.is_set()))

    def wait_for_event_timeout(e, t):
        """Wait t seconds and then timeout"""
        print ('wait_for_event_timeout: starting')
        e.wait(t)
        print ('wait_for_event_timeout: e.is_set()->' + str(e.is_set()))


    if __name__ == '__main__':
        e = multiprocessing.Event()
        w1 = multiprocessing.Process(name='block',
                                     target=wait_for_event,
                                     args=(e,))
        w1.start()

        w2 = multiprocessing.Process(name='non-block',
                                     target=wait_for_event_timeout,
                                     args=(e, 2))
        w2.start()

        time.sleep(3)
        e.set()
        print ('main: event is set')

    #the output is:
    #wait_for_event_timeout: starting
    #wait_for_event: starting
    #wait_for_event_timeout: e.is_set()->False
    #main: event is set
    #wait_for_event: e.is_set()->True


# 七.进程共享变量
因为进程拥有自己独立的命名空间和地址空间，无法通过一些全局变量来实现，multiprocessing提供了一些特殊的函数来实现共享变量：

1. Value,Array的方式 
2. Manager的方式

## 7.1 Value,Array

    from multiprocessing import Value, Array, Process


    def func(n, a, i):
        """
        共享变量
        n: Value实例，如:<Synchronized wrapper for c_double(0.0)>
        a: Array实例，如:<SynchronizedArray wrapper for <multiprocessing.sharedctypes.c_int_Array_10 object at 0x10cce3b90>>
        i: 进程数
        """
        n.value = 3.1415926
        for i in range(len(a)):
            a[i] = a[i] + i

    if __name__ == '__main__':
        num = Value('d', 0.0)               # 返回同步共享对象
        arr = Array('i', range(10))         # 返回同步共享数组
        ps = []
        for i in range(4):
            p = Process(target=func, args=(num, arr, i))
            ps.append(p)
            p.start()

        for i in ps:
            i.join()        # 如果不执行join则状态没有更新过来，此时，n.value:0.0, arr还是0-9

        print num.value
        print arr[:]

    #outprint:
    # 3.1415926
    # [0, 5, 10, 15, 20, 25, 30, 35, 40, 45]


**Value()和Array()都有两个参数第一个参数代表存放的值的类型(d:double, i:int)，第二个参数代表其值。**

## 7.2 Manager

    from multiprocessing import Process, Manager


    def func(d, l):
        d[1] = '1'
        d['2'] = 2
        d[0.25] = None
        d['name'] = 'man'
        l.reverse()

    if __name__ == '__main__':
        manager = Manager()
        d = manager.dict()
        l = manager.list(range(10))
        p = Process(target=func, args=(d, l))
        p.start()
        p.join()
        print d
        print l
        
    # outprint:
    # {0.25: None, 1: '1', '2': 2, 'name': 'man'}
    # [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]

Manager支持的更加丰富些，不过速度相对也慢一些。

# 八. 多进程的好处与弊端
可参考知乎：[Linux下多线程和多进程程序的优缺点，各个适合什么样的业务场景？](http://www.zhihu.com/question/24485648)

# 九. 总结

与多线程有相似的API，如Process对象与Thread对象的用法相同，也有`start()`, `run()`, `join()`的方法。与threading一样，multiprocessing也有`Lock`、`Event`、`Semaphor`、`Condition`类用与**同步**

但在使用这些共享API的时候，我们要注意以下几点:

- 在UNIX平台上，当某个进程终结之后，该进程需要被其父进程调用`wait`，否则进程成为僵尸进程(Zombie)。所以，有必要对每个Process对象调用`join()`方法 (实际上等同于wait)。对于多线程来说，由于只有一个进程，所以不存在此必要性。
- multiprocessing提供了threading包中没有的IPC(比如Pipe和Queue)，效率上更高。应优先考虑Pipe和Queue，避免使用Lock/Event/Semaphore/Condition等同步方式 (因为它们占据的不是用户进程的资源)。
- 多进程应该避免共享资源。在多线程中，我们可以比较容易地共享资源，比如使用全局变量或者传递参数。在多进程情况下，由于每个进程有自己独立的内存空间，以上方法并不合适。此时我们可以通过共享内存和Manager的方法来共享资源。但这样做提高了程序的复杂度，并因为同步的需要而降低了程序的效率。
- Process.PID中保存有PID，如果进程还没有start()，则PID为None。



# 十. 参考:

- [Python多进程multiprocessing使用示例](http://outofmemory.cn/code-snippet/2267/Python-duojincheng-multiprocessing-usage-example)
- [Python标准库10 多进程初步 (multiprocessing包)](http://www.cnblogs.com/vamei/archive/2012/10/12/2721484.html)
- [python之多进程multiprocessing](http://forlinux.blog.51cto.com/8001278/1423390)



