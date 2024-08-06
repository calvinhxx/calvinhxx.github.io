---
layout: post
title:  "Reentrancy and Thread-Safety"
date:   2024-08-05 00:00:00 +0000
categories: Thread 
---
- [可重入和线程安全](#可重入和线程安全)
  - [函数维度:](#函数维度)
    - [例子如下:](#例子如下)
  - [类维度:](#类维度)
    - [例子如下:](#例子如下-1)
  - [Qt中的线程安全与可重入](#qt中的线程安全与可重入)
  - [参考资料:](#参考资料)

# 可重入和线程安全

`可重入`和`线程安全`通常用于描述跨线程使用场景中的函数、类.

## 函数维度:

Ⓜ️️ `可重入函数`: 可以在函数任意执行序列被中断, 其他函数实例调用依旧执行正常.

🧵 `线程安全函数`: 可以跨线程访问, 对共享数据的操作都是原子的。

### 例子如下:

- 非可重入、线程不安全
  
```c++
int t;

void swap(int *x, int *y) {
  t = *x;
  *x = *y;
  // `my_func()` could be called here
  *y = t;
}

void my_func() {
  int x = 1, y = 2;
  swap(&x, &y);
}
```
- 非可重入、线程安全
  
```c++
#include <threads.h>
// `t` is now local to each thread
thread_local int t;

void swap(int *x, int *y) {
  t = *x;
  *x = *y;
  // `my_func()` could be called here
  *y = t;
}

void my_func() {
  int x = 1, y = 2;
  swap(&x, &y);
}
```
- 可重入、线程不安全
  
```c++
#include <threads.h>
int t;

void swap(int *x, int *y) {
  int s;
  // save global variable
  s = t;
  t = *x;
  *x = *y;

  // `my_func()` could be called here
  *y = t;
  // restore global variable
  t = s;
}

void my_func() {
  int x = 1, y = 2;
  swap(&x, &y);
}
```
- 可重入、线程安全
  
```c++
void swap(int *x, int *y) {
  int t = *x;
  *x = *y;

  // `my_func()` could be called here
  *y = t;
}

void my_func() {
  int x = 1, y = 2;
  swap(&x, &y);
}
```
`线程安全`函数始终`可重入`, `可重入`不总是`线程安全`.

## 类维度:

Ⓜ️️ `可重入类`: 类的不同实例, 能在不同线程安全调用成员函数.

🧵 `线程安全类`: 可以跨线程访问类数据和方法, 对共享数据的操作都是原子的。

**note:** 通常情况下C++类是可重入的, 线程安全类一般是可重入的, 可重入类不一定线程安全.

### 例子如下: 

- 线程不安全
  
```c++
class Counter
{
public:
    Counter() { n = 0; }

    void increment() { ++n; }
    void decrement() { --n; }
    int value() const { return n; }

private:
    int n;
};
```

- 线程安全
  
```c++
class Counter
{
public:
    Counter() { n = 0; }

    void increment() { QMutexLocker locker(&mutex); ++n; }
    void decrement() { QMutexLocker locker(&mutex); --n; }
    int value() const { QMutexLocker locker(&mutex); return n; }

private:
    mutable QMutex mutex;
    int n;
};
```

## Qt中的线程安全与可重入

常见的Gui相关类是非可重入、线程不安全的, 未标准可重入的类不能再夸线程场景中使用, 标注了可重入也需要注意线程安全问题.

![Image](/assets/images/ReentrancyAndThread-Safety/image.png)

<!-- ![alt text](image.png)
![alt text](image-1.png) -->

## 参考资料:

[ref blog](https://deadbeef.me/2017/09/reentrant-threadsafe)

[ref qt](https://doc.qt.io/qt-5/threads-reentrancy.html)