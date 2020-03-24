### AQS(AbstractQueuedSynchronizer)

* 什么是AQS?  
    * 基于队列的同步器(入口等待队列+条件等待队列,FIFO)
    * 用来实现代码级别的锁(谁能对state变量成功加1，就代表获取锁成功)
    * 抽象模版基类
    
* 什么是同步？什么是互斥？  
    * 互斥：当一个线程运行某段公共资源时，其它线程不能再运行该公共资源
    * 同步：再互斥的基础上，但是必须严格按照先后顺序依次来运行公共资源
    * 同步互斥都排他，但是互斥任务无序，同步不需按顺序执行

* AbstractQueuedSynchronizer内部几个重要成员
    ```java
        /**
         * 等待队列头节点
         */
        private transient volatile Node head;
    
        /**
         * 等待队列尾节点
         */
        private transient volatile Node tail;
    
        /**
         * state值
         */
        private volatile int state;
    ```

* 来看看ReentrantLock的锁如何使用

    * 加锁示例:
        ```java
             ReentrantLock lock = new ReentrantLock();
             try {
                 //加锁
                 lock.lock();
                 //业务逻辑
             } finally {
                 //解锁
                 lock.unlock();
             }
        ```
    * new ReentrantLock()干了什么？
        ```java
            private final Sync sync;
    
            public ReentrantLock() {
                   //默认是非公平锁
                   sync = new NonfairSync();
               }
           
            abstract static class Sync extends AbstractQueuedSynchronizer{}
           /**
           * Sync是AbstractQueuedSynchronizer的一个抽象子类，
           * 下面有两个实现:FairSync公平锁和非公平锁NonfairSync
           */
        ```
    
    * lock.lock()干了什么？
        ```java
           public void lock() {
               sync.lock();
           }
     
           /**
           * ReentrantLock直接将lock指令委派给内部的Sync
           */
            static final class NonfairSync extends Sync {
                    private static final long serialVersionUID = 7316153563782823691L;
            
                    /**
                     * Performs lock.  Try immediate barge, backing up to normal
                     * acquire on failure.
                     */
                    final void lock() {
                      /**
                       * CAS操作将State设置成1
                       * 1.CAS成功，说明加锁成功，将独占线程变量改成当前线程
                       * 2.CAS失败，执行acquire指令     
                       */
                        if (compareAndSetState(0, 1))
                            setExclusiveOwnerThread(Thread.currentThread());
                        else
                            acquire(1);
                      /**
                      * 非公平锁上来就尝试cas设置state值，这就是不公平的体现
                      */
                    }
            
                    protected final boolean tryAcquire(int acquires) {
                        return nonfairTryAcquire(acquires);
                    }
                }
              //来看看AbstractQueuedSynchronizer的acquire内部逻辑
              public final void acquire(int arg) {
                  /**
                  * 三行代码分四步操作
                  * 1. tryAcquire 尝试获取锁(state加1操作)，获取到直接结束流程
                  * 2. addWaiter 1失败后,创建当前线程节点尝试入队，
                  *  2.1 如果尾节点存在，尝试一次CAS将当前节点设置成尾节点，成功就表示入队ok，如果CAS失败进行enq(node)自旋+CAS操作
                  *  2.2 如果尾节点不存在，直接进行enq(node)自旋+CAS操作
                  *  2.3 addWaiter结束后返回当前节点
                  * 3. acquireQueued 基于等待队列的尝试获取锁
                  *  3.1 如果当前节点是第二个节点，则再进行一次tryAcquire，如果获取锁成功，head节点后移，结束
                  *  3.2 如果当前节点不是第二个节点，执行shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt()进行阻塞(LockSupport#park)
                  * 4. selfInterrupt 中断自己
                  */
                  if (!tryAcquire(arg) &&
                      acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
                      selfInterrupt();
                  }
        ```
    * lock.unlock()做了什么？
        ```java
           public void unlock() {
                  sync.release(1);
              }
        
           public final boolean release(int arg) {
               if (tryRelease(arg)) {
               /**
                 * 尝试解锁成功
                 * 通知当前节点的后继节点,也就是解除后继节点阻塞(LockSupport#unpark)
                */
               Node h = head;
               if (h != null && h.waitStatus != 0)
                   unparkSuccessor(h);
               return true;
               }
               return false;
           }
         
           protected final boolean tryRelease(int releases) {
               int c = getState() - releases;
               /**
                 * 校验解锁线程是不是加锁的线程
               */
               if (Thread.currentThread() != getExclusiveOwnerThread())
                   throw new IllegalMonitorStateException();
               boolean free = false;
               /**
                * State减到0时,说明锁被释放，排它锁的占有线程设置成空，返回true
               */
               if (c == 0) {
                   free = true;
                   setExclusiveOwnerThread(null);
               }
               /**
               * 更新state，这里没有用到CSA操作，因为不是加锁线程就抛异常了
               */
               setState(c);
               return free;
           }
        ```