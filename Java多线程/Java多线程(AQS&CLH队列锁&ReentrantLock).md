### AbstractQueuedSynchronizer

队列同步器`AbstractQueuedSynchronizer（以下简称同步器或AQS）`，是用来构建锁或者其他同步组件的基础框架，它使用了一个`int`成员变量表示同步状态，通过内置的`FIFO队列`来完成资源获取线程的排队工作。

**AQS使用方式**



 ![1591604749760](../assets/Java多线程(AQS&CLH队列锁&ReentrantLock)/1591604749760.png)

