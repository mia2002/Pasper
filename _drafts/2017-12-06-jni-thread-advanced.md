---
layout: post
date: 2017-12-03
title: POSIX线程下篇
tags: Android
---

## 线程同步

由于运行在相同的进程空间，线程共享相同的内存和资源。这使得线程彼此通信和共享数据变得容易，但是有可能产生两种错误：由于并发修改共享资源产生线程干扰和内存不一致性，此时线程同步变得至关重要。线程同步机制确保两个并发运行的线程不同时执行代码的特定部分。

- 互斥锁(Mutexes)确保代码的互斥执行，即代码的特定部分不同时执行。
- 信号量(Semaphores)控制对特定数目可用资源的访问，如果没有可用资源，调用线程只是在信号量所涉及的资源上等待，直到资源可用。

## 互斥锁

### 初始化互斥锁

通过pthread\_mutex\_init函数或者PTHREAD\_MUTEX\_INITIALIZER宏。

```c
int pthread_mutex_init(pthread_mutex_t* mutex,const pthread_mutexattr_t* attr);
```

函数参数说明如下：

- pthread\_mutex\_t : 指向要初始化的互斥变量的指针。
- pthread\_mutexattr\_t : 指向为互斥锁定义属性的指针。设为NULL，将使用默认属性。

PS:如果采用默认属性，PTHREAD\_MUTEX\_INITIALIZER宏比pthread\_mutex\_init函数更合适。

```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
``` 

如果初始化成功，互斥锁被初始化并处于锁打开的状态，函数返回0，否则返回错误代码。

示例代码：

```c
static pthread_mutex_t mutex;

void init(){
if(0 != pthread_mutex_init(&mutex,NULL){
	//错误处理
}
```

### 锁定互斥锁

```c
int pthread_mutex_lock(pthread_mutex_t* mutex);
```

该函数带有一个指向互斥锁变量的指针。如果互斥锁已经被锁上，调用线程被挂起直到互斥锁被打开。如果成功，函数返回0，否则返回错误代码。

示例代码：

```c
if(0 != pthread_mutex_lock(&mutex)){
	//错误处理
}
```

### 解锁互斥锁

```c
int pthread_mutex_unlock(pthread_mutex_t* mutex);
```

该函数带有一个指向要解锁的互斥锁变量。调度策略决定解锁后执行哪个等待互斥锁的线程。如果成功，函数返回0，否则返回错误代码。

示例代码：

```c
if(0 != pthread_mutex_unlock(&mutex)){
	//错误处理
}
```

### 销毁互斥锁

```c
int pthread_mutex_destory(pthread_mutex_t* mutex);
```

一旦不在需要互斥锁，可以用pthread\_mutex\_destory函数销毁互斥锁。该函数带有一个指向要销毁的互斥锁变量的指针，视图销毁一个锁着的变量将返回不确定结果。

示例代码：

```c
if(0 != pthread_mutex_destroy(&mutex)){
	//错误处理
}
```

## 信号量

头文件：

```c
include <semaphore.h>
```

### 初始化信号量

```c
int sem_init(sem_t* sem,int pshared,unsigned int value);
```

三个参数分别是将被初始化的信号量变量指针、共享标志（flag）及其初始值。若成功，函数返回0，否则返回-1.

### 锁定信号量

一旦信号量被正确初始化，线程可以使用sem_wait函数减少信号量的数量。

```c
int sem_wait(sem_t* sem);
```

这个函数有一个信号量变量指针。如果信号量的值大于零，上锁成功，并且信号量的值也会相应递减。如果信号量的值是零，调用线程被挂起，直到另外一个线程通过解锁它增加了信号量的值。如果成功，函数返回0，否则返回-1。

### 解锁信号量

在临界区代码执行完成时，线程可以使用sem_post函数解锁信号量。

```c
int sem_post(sem_t* sem);
```

当使用sem_post函数解锁信号量以后，信号量的值会增加1。调度策略决定信号量解锁后执行哪个等待线程。如果成功，函数返回0，否则返回-1.

### 销毁信号量

```c
int sem_destroy(sem_t* sem);
```
一旦不再需要信号量，可以通过sem_destroy函数销毁它。

该函数有一个将被销毁的信号量变量指针。销毁一个另一个线程正在阻塞的信号量有可能导致未知行为。如果成功，函数返回0，否则返回-1.




