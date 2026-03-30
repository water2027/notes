进程是资源分配的单位, 线程是系统调度的单位. 

linux并不区分进程与线程. 通常使用fork来创建需要分配独立资源的线程(进程), 使用pthread_create创建需要共享资源的线程. 

> - fork里面也是调用clone, 但是不共享资源而是复制资源. 
> - pthread_create内部调用clone, 但是是共享资源. 

```c
clone(CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SIGHAND, 0);
```

当有相应的标志位时, 那么采用共享资源; 如果缺失, 那么采用复制资源. 

> linux实现了写时拷贝Copy-On-Write, 只有要写时, 复制才会发生. COW使得linux的fork相当的快. 

当执行一个程序时, 

```shell
./program
```

1. Shell进程调用一个fork创建一个新task. 
2. 新task调用exec加载program代码
3. 内核为这个task分配资源
	- 独立的地址空间
	- 文件描述符表
	- 信号处理器等
4. 这个task开始运行. 
> 一般来讲, 这个task就是常说的进程, 也可以叫主线程. 

线程是由已经存在的task创建的. 当program需要开一个线程时:

1. 当前task调用pthread_create
2. pthread_create内部使用clone
3. 内核创建新的task_struct
4. 新的task共享当前task的资源
5. 两个task同属于一个进程组