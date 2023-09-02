---
title: thread-sync-with-condition-variable
tags:
  - c++
  - thread
date: 2023-09-02 23:00:41
---


# c++ 条件变量 condition_variable
c++的condition variable是用来实现多线程同步的一种机制。

condition variable在使用时需要配合互斥锁：mutex，并且是用unique_lock 包装的:

```
std::mutex mtx;
std::unique_lock<std::mutex> lck(mtx);
```

condition variable可以用来阻塞一个或多个线程，直到有一个线程修改共享的condition variable。

如果一个线程要修改共享量，需要如下操作：
- 请求一个互斥量，通常是unique_lock  
- 占有互斥量后， 可以修改了。  
- 调用notify_once/notify_all（可以在释放锁喉完成）

如果想要等待condition variable需要：
- 获取用于保护共享变量的互斥锁上的 std::unique_lock<std::mutex>  
- 执行以下操作之一：
  - 检查条件，是否是否已经被更新和通知   
  - 在condition_variable上调用wait，wait_for或wait_until（自动释放锁并且阻塞线程，知道条件变量被notify，timeout expires或者线程被虚假唤醒:spurious wakeup,然后在返回前自动获得锁） 
  - 检查条件，不满足则继续等待

# 确保唤醒
为了确保被唤醒，和防止虚假唤醒，一般会在wait前有个循环，或直接将判断放到wait参数里面：
```
cv.wait(lk, []{return ready;});
或：while (ready) cv.wait();
```

# 示例：三个线程按次序打印：
```
bool complete[3] = { 0 , 0, true };
std::mutex mtx;
std::condition_variable produce, consume;
#define LOOP_TIMES 1000

void PrintThreadA() {
  for (int i = 0; i < LOOP_TIMES; i++)
  {
    std::unique_lock<std::mutex> lck(mtx);
    conda.wait(lck, [] {return complete[2]; });
    std::cout << "|";
    //std::this_thread::sleep_for(std::chrono::milliseconds(123));
    complete[0] = true;
    complete[2] = false;
    condb.notify_one();
  }
}

void PrintThreadB() {
  for (int i = 0; i < LOOP_TIMES; i++)
  {
    std::unique_lock<std::mutex> lck(mtx);
    condb.wait(lck, [] {return complete[0]; });
    std::cout << "-";
    complete[1] = true;
    complete[0] = false;
    condc.notify_one();
  }
}

void PrintThreadC() {
  for (int i = 0; i < LOOP_TIMES; i++)
  {
    std::unique_lock<std::mutex> lck(mtx);
    condc.wait(lck, [] {return complete[1]; });
    std::cout << "<";
    complete[2] = true;
    complete[1] = false;
    conda.notify_one();
  }
}

void TestThread() {
  std::thread A(PrintThreadA), B, C;
  do {
    if (complete[0])
      break;
  } while (true);
  B = std::thread{ PrintThreadB };
  C = std::thread{ PrintThreadC };
  A.join();
  B.join();
  C.join();
  std::cout << "sum : " << sum << std::endl;
  return;
}
```

> 输出：
![2023-09-02T225444](2023-09-02T225444.png)
