title: Unix编程入门2--线程同步
date: 2014-11-01 09:20:36
tags: [Code,Unix,Linux]
categories: Unix
---

## 意义
当一个线程可以修改的变量，其他的线程也可以读取或修改的时候，我们就需要进行线程同步，以确保线程在访问资源是不会获取无效的值。

## 几种线程同步涉及到的几种技术
### 互斥量(mutex)
互斥量从本质上讲是一把锁，在访问共享资源前对互斥量进行设置(加锁)，访问完成后释放(解锁)互斥量。这样可以确保同一时间只有一个线程访问资源，防止数据不一致的出现。
互斥量使用pthread_mutex_t数据类型表示，使用前需进行初始化，可将其设置为常量PTHREAD_MUTEX_INITIALIZER，也可通过调用pthread_mutex_init函数进行初始化。如果动态分配互斥量，在释放内存时需要调用pthread_mutex_destroy。
```c
#include <pthread.h>

int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *rerestrict attr);
/*	attr一般设为NULL表示以默认属性初始化互斥量 */
int pthread_mutex_init(pthread_mutex_t *mutex);

/* 函数返回：成功返回0，失败返回错误码 */
```
加锁和解锁：
```c
#include <pthread.h>

int pthread_mutex_lock(pthread_mutex_t *mutex);
/* 对互斥量加锁 */

int pthread_mutex_trylock(pthread_mutex_t *mutex);
/* 如果互斥量未加锁对互斥量加锁，不会组阻塞且返回0； 如已经加锁该操作就会失败，无法所互斥量并返回EBUSY */

int pthread_mutex_unlock(pthread_mutex_t *mutex);
/* 解锁 */

/* 函数返回：成功返回0，失败返回错误编号 */
```
当互斥量已经上锁时，调用线程将阻塞直到互斥量被解锁。

[例程](https://github.com/StromAI/GreedIsGood/blob/master/Unix_Introduction/Thread/Thread_mutex.c)
PS:
```C
#include <pthread.h>
#include <time.h>
int pthread_mutex_timedlock(pthread_mutex_t * restrict mutex,
							const struct timespec *restrict tsptr);
/* 成功返回0，失败返回错误编号 */
```
当该函数调用时，互斥量原语允许绑定线程阻塞时间(tsptr)，未超时时，其表现与pthread_mutex_lock一致对互斥量加锁，当超时后则不会对互斥量加锁，而是返回ETIMEDOUT。
该函数的使用主要是为了防止在死锁时所造成的永久阻塞。

###