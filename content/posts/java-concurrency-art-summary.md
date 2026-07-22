+++
date = '2024-03-23'
title = '《Java 并发编程的艺术》个人总结'
categories = ['Notes']
tags = ['Java', '并发编程']
+++

# 《Java 并发编程的艺术》个人总结

## 为什么读这本书

工作中写并发代码的机会不少——线程池、缓存、异步处理——但遇到诡异的可见性问题或死锁时，往往只能靠加 `synchronized` 碰运气。这本书系统地梳理了 Java 内存模型和并发工具的设计原理，读完确实对调试并发问题有了更清晰的思路。

## 核心知识点梳理

### 1. 并发编程的挑战

书中开篇说的三个核心问题：
- **上下文切换**：线程多了，CPU 花在切换上的时间比干活还多
- **死锁**：互相等对方的锁
- **资源限制**：硬件瓶颈不解决，加再多线程也没用

经验上，**线程数 ≈ CPU 核数 × 2（取决于 IO 密集还是计算密集）** 是一个比较可靠的起点。

### 2. Java 内存模型（JMM）

这是全书最重要的一章。JMM 控制的是**一个线程对共享变量的写入何时对另一个线程可见**。核心概念：
- **happens-before 规则**：程序顺序、volatile、锁、传递性等
- **volatile**：保证可见性和禁止指令重排序，但不保证原子性
- **锁**：`synchronized` 和 `ReentrantLock` 同时保证原子性和可见性

并发编程中大部分诡异 bug 的根因都是**对 JMM 的理解不到位**——以为一个线程写完了另一个线程就一定能读到。

### 3. 并发基础组件

| 组件 | 核心作用 | 使用场景 |
|------|----------|----------|
| `volatile` | 轻量级可见性保证 | 状态标识位 |
| `synchronized` | 互斥 + 可见性 | 临界区保护 |
| `ReentrantLock` | 可中断/超时/公平锁 | 需要灵活锁控制的场景 |
| `AtomicInteger/Long/Reference` | CAS 无锁原子操作 | 计数器、状态机 |
| `CountDownLatch` | 等待多个线程完成 | 并行任务汇总 |
| `CyclicBarrier` | 多个线程互相等待 | 分阶段并行计算 |
| `Semaphore` | 流量控制 | 限流、资源池 |

### 4. 线程池

`ThreadPoolExecutor` 的核心参数关系：
- **corePoolSize/ maximumPoolSize**：核心线程和最大线程数
- **keepAliveTime**：非核心线程的空闲存活时间
- **workQueue**：任务队列满了才会创建新线程（从核心数涨到最大数）

实践中常见的问题是**队列设得太大**——线程池永远只用核心线程，失去了弹性伸缩的意义。另外 `Executors.newCachedThreadPool()` 要慎用，它允许无限创建线程。

### 5. 并发集合

- **ConcurrentHashMap**：分段锁（JDK7）→ CAS + synchronized（JDK8），读不加锁
- **CopyOnWriteArrayList**：写时复制，适合读多写少
- **BlockingQueue**：生产者-消费者模式的基础

### 6. 锁优化

JVM 对 `synchronized` 做了大量优化：
- **偏向锁**：同一个线程反复获取锁，直接放行
- **轻量级锁**：多线程交替执行，用 CAS 代替互斥
- **重量级锁**：真正的线程阻塞

这也是为什么现在不需要一味追求 `ReentrantLock`——JVM 的 `synchronized` 优化已经足够好，除非你需要锁中断或超时。

## 一点个人感悟

读完这本书最大的收获不是记住了多少 API，而是**理解了 JMM 的 happens-before 规则后，排查并发问题有了一个系统的思维框架**。不再是把 `volatile`、`synchronized`、`Lock` 当成"可能管用"的咒语到处乱加，而是能根据问题的性质（可见性？原子性？有序性？）针对性地选择工具。

建议读完这本书后，再配合《Java Concurrency in Practice》一起看，两本书互为补充。
