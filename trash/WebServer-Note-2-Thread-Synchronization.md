---
title: WebServer Note 2 Thread Synchronization
date: 2022-07-06
tags:
- C++
- Linux
categories: Project
---

> Resource Acquisition is Initialization.
>
> 将资源和对象的生命周期绑定，智能指针是最好的例子。

# 锁机制

实现多线程同步，通过锁机制，确保任一时刻只能有一个线程能进入关键代码段。

# 封装

locker.h，对Linux下三种锁进行封装，将锁的创建于销毁函数封装在类的构造与析构函数中，实现RAII机制。

## 信号量

PV操作；假设有信号量SV。信号量的取值可以是任何自然数；最简单的信号量是二进制信号量，只有0和1两个值。

> P，如果SV的值大于0，则将其减一；若SV的值为0，则挂起执行。
>
> V，如果有其他进行因为等待SV而挂起，则唤醒；若没有，则将SV值加一。

```cpp
class sem {
public:
    sem() {
        /*
         * @description  sem_init() initializes the unnamed semaphore at the
         * address pointed to by sem.
         * @param {sem_t *} address
         * @param {int} pshared pshared indicates whether this semaphore is to
         * be shared between the threads of a process, or between processes.
         * @param {int} value
         */
        if (sem_init(&m_sem, 0, 0) != 0) {
            throw std::exception();
        }
    }

    sem(int num) {
        if (sem_init(&m_sem, 0, num) != 0) {
            throw std::exception();
        }
    }

    ~sem() {
        /*
         * @description sem_destroy() destroys the semaphore at the address
         * pointed to by sem.
         * @param address
         */
        if (sem_destroy(&m_sem) != 0) {
            throw std::exception();
        }
    }

    bool wait() {
        /*
         * @description sem_wait() decrements (locks) the semaphore pointed to
         * by sem.
         * If the semaphore's value is greater than zero, then the
         * decrementproceeds, and the function returns, immediately.  If the
         * semaphore currently has the value zero, then the call blocks until
         * either it becomes possible to perform the decrement (i.e., the
         * semaphore value rises above zero), or a signal handler interrupts the
         * call.
         * @param address
         * @return return 0 on success; return -1 on error
         */
        if (sem_wait(&m_sem) != 0) {
            throw std::exception();
        }
        return true;
    }

    bool post() {
        /*
         * @description sem_post() increments (unlocks) the semaphore pointed to
         * by sem. If the semaphore's value consequently becomes greater than
         * zero, then another process or thread blocked in a sem_wait call
         * will be woken up and proceed to lock the semaphore.
         * @param address
         * @return return 0 on success; return -1 on error
         */
        if (sem_post(&m_sem) != 0) {
            throw std::exception();
        }
    }

private:
    sem_t m_sem;
};
```

## 互斥量

互斥锁；保护关键代码段，以确保独占式访问。当进入关键代码段，获得互斥锁将其加锁；离开关键代码段，唤醒等待该互斥锁的线程。

```cpp
class locker {
public:
    locker() {
        /*
         * @param mutexattr The pthread_mutex_init() function initialises the
         * mutex referenced by mutex with attributes specified by attr. If attr
         * is NULL, the default mutex attributes are used; the effect is the
         * same as passing the address of a default mutex attributes object.
         * Upon successful initialisation, the state of the mutex becomes
         * initialised and unlocked.
         */
        if (pthread_mutex_init(&m_mutex, NULL) != 0) {
            throw std::exception();
        }
    }
    ~locker() {
        if (pthread_mutex_destroy(&m_mutex) != 0) {
            throw std::exception();
        }
    }
    bool lock() {
        if (pthread_mutex_lock(&m_mutex) != 0) {
            throw std::exception();
        }
        return true;
    }
    bool unlock() {
        if (pthread_mutex_unlock(&m_mutex) != 0) {
            throw std::exception();
        }
        return true;
    }
    pthread_mutex_t *get() { return &m_mutex; }

private:
    pthread_mutex_t m_mutex;
};
```

## 条件变量

线程间通信；当某个共享数据达到某个值时，唤醒等待这个共享数据的线程。条件变量的封装值得一提；首先提示，以下注释中pthread_cond_wait必须在加锁的条件下使用，本身是没有问题的。

```cpp
// 条件变量的使用机制需要配合锁来使用
// 内部会有一次加锁和解锁
// 封装起来会使得更加简洁
bool wait() {
    int ret = 0;
    // wait前必须上锁
    // 即pthread_cond_wait必须在加锁的条件下使用
    pthread_mutex_lock(m_mutex);
    ret = pthread_cond_wait(&m_cond, m_mutex);
    pthread_mutex_unlock(m_mutex);
    if (ret != 0) {
        throw std::exception();
    }
    return true;
}
```

对于这种写法，即将mutex隐藏在cond类中，可以理解，但是不能正常使用。

> Q：mutex用来保护什么？

通常在程序里，使用条件变量来表示等待”某一条件”的发生。虽然名叫”条件变量”，但是它本身并不保存条件状态，本质上条件变量仅仅是一种通讯机制：当有一个线程在wait某一条件变量的时候，会将当前的线程挂起，直到另外的线程发送信号notify通知其解除阻塞状态。

所以需要使用额外的共享变量保存条件状态；而由于这个变量会同时被不同的线程访问，因此需要一个额外的mutex保护它。

> A condition variable is always used in conjunction with a mutex. The mutex provides mutual exclusion for accessing the shared variable, while the condition variable is used to signal changes in the variable’s state.

条件变量总是结合mutex使用，条件变量就共享变量的状态改变发出通知，**mutex用来保护这个共享变量的**。

> **为什么pthread_cond_wait()需要mutex参数？**

则在wait的封装中使用mutex参数。使用条件变量的接口实现一个简单的生产者-消费者模型，avail就是保存条件状态的共享变量，它对生产者线程、消费者线程均可见。不考虑错误处理，先看生产者实现：

```cpp
pthread_mutex_lock(&mutex);
avail++;
pthread_mutex_unlock(&mutex);

pthread_cond_signal(&cond); /* Wake sleeping consumer */
```

因为avail对两个线程都可见，因此对其修改均应该在mutex的保护之下，再来看消费者线程实现：

```cpp
for (;;)
{
    pthread_mutex_lock(&mutex);
    while (avail == 0)
        pthread_cond_wait(&cond, &mutex);

    while (avail > 0)
    {
        /* Do something */
        avail--;
    }
    pthread_mutex_unlock(&mutex);
}
```

当”avail==0”时，消费者线程会阻塞在pthread_cond_wait()函数上。如果pthread_cond_wait()仅需要一个pthread_cond_t参数的话，此时mutex已经被锁，要是不先将mutex变量解锁，那么其他线程（如生产者线程）永远无法访问avail变量，也就无法继续生产，消费者线程会一直阻塞下去。因此pthread_cond_wait()需要传递给它一个pthread_mutex_t类型的变量。

pthread_cond_wait()函数分为3个部分：

1. 解锁互斥量mutex；
2. 阻塞调用线程，直到当前的条件变量收到通知；
3. 重新锁定互斥量mutex。

其中1和2是原子操作，也就是说在pthread_cond_wait()调用线程陷入阻塞之前其他的线程无法获取当前的mutex，也就不能就该条件变量发出通知。

> 虚假唤醒：判断条件放在了while中，而不是if，这是因为pthread_cond_wait()阻塞在条件变量上的时候，即使其他的线程没有就该条件变量发出通知notify()或者notifyAll()，条件变量也有可能会自己醒来，即pthread_cond_wait()函数返回，因此需要重新检查一下共享变量上的条件成不成立，**确保条件变量是真的收到了通知**，否则继续阻塞等待。

回到条件变量的封装问题。

从以上分析，可知pthread_cond_wait()使用时，必须遵守的几个法则：

1. 必须结合mutex使用，mutex用于保护共享变量，而不是保护pthread_cond_wait()的某些内部操作；
2. mutex上锁后，才能调用pthread_cond_wait()；
3. 条件状态判断要放在while()循环里，然后再pthread_cond_wait()。

```cpp
class cond
{
  public:
    bool wait()
    {
        int ret = 0;
        pthread_mutex_lock(&m_mutex);
        ret = pthread_cond_wait(&m_cond, &m_mutex);
        pthread_mutex_unlock(&m_mutex);
        return ret == 0;
    }

    bool signal()
    {
        return pthread_cond_signal(&m_cond) == 0;
    }

  private:
    pthread_mutex_t m_mutex;
    pthread_cond_t m_cond;
};
```

这段代码中，wait()函数直接将mutex隐藏到其实现里边，这里的mutex完全没发挥作用，没有保护任何的东西，仅仅是为了适配pthread_cond_wait()接口。

参考C++11标准库的接口设计，将mutex放出来，在wait的时候当参数传入。

```cpp
bool wait(pthread_mutex_t *m_mutex) {
    int ret = 0;
    // pthread_mutex_lock(m_mutex);
    ret = pthread_cond_wait(&m_cond, m_mutex);
    // pthread_mutex_unlock(m_mutex);
    if (ret != 0) {
        throw std::exception();
    }
    return true;
}
```

