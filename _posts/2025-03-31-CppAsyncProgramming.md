---
layout: post
title:  "C++异步编程"
date:   2025-03-31 00:00:00 +0000
categories: C++
excerpt_image: /assets/images/CppAsyncProgramming/image.png
tags: [C++]
---

传统C++异步编程，一般结合回调函数来实现。

```c++
void asyncOperation() {
    std::this_thread::sleep_for(std::chrono::seconds(2));
}
void task(std::function<void()> callback) {
    callback();
}

std::thread t1(task, asyncOperation);
t1.join();
```

这种异步编程方式可以应用在基础、简单的场景，在业务体量复杂后会演变成下面这种形态，
```c++
asyncOperation([&](int result) {
    // 第一个回调
    asyncOperation2(result, [&](int result2) {
        // 第二个回调
        asyncOperation3(result2, [&](int result3) {
            // 第三个回调
            // 依此类推...
        });
    });
});
```
A、B、C、D、E、F、G....
B操作依赖A操作结果，C操作依赖B操作结果，D操作依赖C操作结果，E操作依赖D操作结果，F操作依赖E操作结果，G操作依赖F操作结果，依此类推...

这种形态会存在以下问题：
`回调地狱`、`异常处理复杂`、`状态管理复杂`

1. 回调地狱, 代码可读性差, 维护成本高。
2. 异常处理复杂, 需要处理多个回调中的异常, 逻辑会存在强耦合。
3. 状态管理复杂, 需要管理多个回调的状态, 对于AB=Ture，C=False这种形态的状态管理，需要额外维护状态。

为了解决这些问题，ModernC++引入了`std::async`、`std::packaged_task`、`std::future/std::promise`，可以更方便地实现异步编程, 解决上述棘手的问题。

简单介绍一下`std::async`、`std::packaged_task`、`std::future/std::promise`的使用:

1️⃣ `std::async`：是一个函数模板, 传递一个`可调用对象(callable object)[函数、lambda表达式、bind表达式或其他函数对象]`，创建一个异步任务，并返回一个`std::future`对象，用于获取异步任务的结果。

```c++
void asyncOperation() {
    std::this_thread::sleep_for(std::chrono::seconds(2));
}

std::future<void> result = std::async(std::launch::async, asyncOperation); // 启动异步操作
result.wait();
```

2️⃣ `std::packaged_task：`是一个类模板, 是对可调用对象的封装, 教传统可调用对象支持future对象的返回.

```c++
void asyncOperation() {
    std::this_thread::sleep_for(std::chrono::seconds(2));
}

std::packaged_task<void()> task(asyncOperation);
std::future<void> result = task.get_future();
std::thread t(std::move(task));
t.detach();
result.wait();
```

3️⃣ `std::future/std::promise`：Promise表示一个异步操作的承诺，可以用于设置异步操作的结果。Future表示一个异步操作的结果，可以用于获取异步操作的返回值。调用Get时阻塞等待Set唤醒。

```c++
promise<int> pv;
std::future<int> f = pv.get_future();

std::thread t3([&pv]
                {
                    std::cout << "thread id:" << std::this_thread::get_id() << std::endl;
                    std::this_thread::sleep_for(2000ms);
                    pv.set_value(3);
                });
t3.detach();
std::cout << "value: " << f.get() << std::endl;
```

完整的代码示例：
```c++ 
#include <iostream>
#include <future>
#include <thread>

using namespace std;

void asyncOperation() {
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout << "thread id:" << std::this_thread::get_id() << std::endl;
}
void task(std::function<void()> callback) {
    std::cout << "start task" << std::endl;
    callback();
    std::cout << "end task" << std::endl;
}

int main()
{
    // test callback task
    std::thread t1(task, asyncOperation);
    t1.join();

    // test async task
    std::cout << "waiting for async/await..." << std::endl;
    std::future<void> result = std::async(std::launch::async, asyncOperation); // 启动异步操作
    result.wait();
    std::cout << "end async task" << std::endl;

    // test packaged_task
    std::cout << "waiting for packaged_task..." << std::endl;
    std::packaged_task<void()> packaged_task(asyncOperation);
    std::future<void> packaged_result = packaged_task.get_future();
    std::thread t2(std::move(packaged_task));
    t2.detach();
    packaged_result.wait();
    std::cout << "end packaged_task" << std::endl;

    // test promise future
    promise<int> pv;
    std::future<int> f = pv.get_future();

    std::thread t3([&pv]
                   {
                       std::cout << "thread id:" << std::this_thread::get_id() << std::endl;
                       std::this_thread::sleep_for(2000ms);
                       pv.set_value(3);
                   });
    t3.detach();
    std::cout << "value: " << f.get() << std::endl;
    return 0;
}
```

可以看出通过`std::async`、`std::packaged_task`、`std::future/std::promise`等方案可以方便地实现异步编程。我们一起研究下这些组件底层的实现原理.

```c++
template <typename T>
class SimpleFuture {
public:
    SimpleFuture(SimplePromise<T>& p) : promise(&p) {}

    T get() {
        return promise->get_value(); // 获取值
    }

private:
    SimplePromise<T>* promise; // 关联的 promise
};

template <typename T>
class SimplePromise {
public:
    SimplePromise() : is_set(false) {}

    void set_value(const T& value) {
        std::lock_guard<std::mutex> lock(mutex);
        if (is_set) {
            throw std::runtime_error("Value already set");
        }
        this->value = value;
        is_set = true;
        cv.notify_all();
    }

    T get_value() {
        std::unique_lock<std::mutex> lock(mutex);
        cv.wait(lock, [this] { return is_set; });
        return value;
    }

private:
    T value;
    bool is_set;
    std::mutex mutex;
    std::condition_variable cv;
};

template <typename R, typename... Args>
class SimplePackagedTask {
public:
    SimplePackagedTask(std::function<R(Args...)> f) : func(f), is_set(false) {}

    void operator()(Args... args) {
        {
            std::lock_guard<std::mutex> lock(mutex);
            if (is_set) {
                return; // 任务已完成，退出
            }
            result = func(args...); // 执行任务
            is_set = true; // 设置完成标志
        }
        cv.notify_all(); // 通知等待的线程
    }

    std::future<R> get_future() {
        return std::move(fut); // 返回 future
    }

private:
    std::function<R(Args...)> func;
    R result;
    bool is_set;
    std::mutex mutex;
    std::condition_variable cv;
    std::promise<R> p;
    std::future<R> fut = p.get_future();
};

template <typename R, typename... Args>
SimpleFuture<R> SimpleAsync(std::function<R(Args...)> func, Args... args) {
    SimplePromise<R> promise;
    SimpleFuture<R> future(promise);

    // 启动新线程执行任务
    std::thread([func, args..., promise = std::move(promise)]() mutable {
        try {
            promise.set_value(func(args...)); // 设置结果
        } catch (...) {
            // 处理异常
        }
    }).detach();

    return future; // 返回对应的 future
}
```