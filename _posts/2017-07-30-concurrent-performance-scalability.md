---
layout: post
title: "【T】Java并发编程 — 性能与扩展性"
---

两个概念：
* 性能：用更少的资源做更多、更快的事。
* 扩展性（伸缩性）：硬件资源的增加与服务的性能质量成正比。

## 性能与伸缩性
并发的目的是提高性能，但是往往被忽略的是，并发的使用通常是增加了代码的复杂程度，并且更容易出现各种未知的问题。另外，在任何希望使用并发提升性能的场景下，首先要保证的是，你的程序是正确的。

通过并行化方式进行性能提升无非是找到代码中的串行部分与可以并行的部分，并将可以并行的部分通过多线程的方式并行化。这里先引入一个用于度量性能提升最终结果的Amdahl定律。

> Amdahl定律： 定律描述了在一个又并行和串行部分构成的系统中，通过增加硬件资源的方式（CPU）,理论上可以获得的加速比。 SpeedUp <= 1/(F+(1-F)/N), 其中F指串行比重，N指处理器个数。举个例子，如果串行部分占比10%，如果有10个处理器，那么加速比为 5.3，此时的利用率为53%。如果增加到100个CPU，那么加速比达到9.3（9%的利用率）。

通过引入多线程，最大限度的提升了并行性。但是并行开销的问题也不容小觑。

#### 并行开销

**上下文切换**

上下文切换指的是操作系统对线程的调度，以便所有线程都能够分配到CPU进行任务执行，当发生线程切换时，需要保存当前线程的上下文并恢复目的线程的上下文，这个过程就是上下文切换。过多的线程阻塞（IO、锁）是导致上下文切换的最主要的原因。

## 锁

#### 锁的优化

锁会导致更多的串行与更频繁的上下文切换，而这两者是性能与扩展性的最大的障碍。

一个锁的竞争性通常由：
* 访问频率
* 持有时间
这两方面决定。而这两点的优化也是服务提升的关键点。

通常的方案是通过减小同步块大小和减小锁的粒度来解决如上的两个问题。而针对于后者，一般通过分拆锁（将对无关资源的访问的锁拆分为不同的锁）和分离锁（将大锁按照分块的方式拆分为多个锁，如ConcurrentHashMap的实现）两种方式实现。

#### 进一步谈谈锁

最初，JDK提供了synchronized和volatile来提供同步支持，后来为了扩展synchronized，ReentrantLock被提出。ReentrantLock在如下方面拓展了同步块语义：
* 无条件。lock().
* 可轮询。tryLock().
* 可定时。tryLock(long time, TimeUnit unit).
* 可中断。lockInterruptibly().

如上，ReentrantLock是作为synchronized的拓展，在使用便利性而言，后者更能避免忘记释放锁的问题。在Java6之后，被诟病的synchronized性能问题已经基本可以忽略。所以，如果synchronized可以满足需要，那么它应该作为首选。
