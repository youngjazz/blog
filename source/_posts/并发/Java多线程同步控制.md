---
title: java多线程同步控制
date: 2017-03-14 18:45:12
tags: java并发
categories: 技术
---


### synchronized的功能扩展：重入锁

重入锁完全可以替代*synchronized*关键字，远优于*synchronized*，但*JDK*之后*synchronized*做了大量优化。差距不大

<!--more-->

```java
public class ReenterLock implements Runnable {

	public static ReentrantLock reentrantLock = new ReentrantLock();
	public static int i = 0;
	@Override
	public void run() {
		for (int j = 0; j < 10000000; j++) {
			reentrantLock.lock();
			reentrantLock.lock();//这就是之所以叫重入锁的原因（局限于一个线程），而不会和自己产生死锁
			try {
				i++;
			}finally {
//				reentrantLock.unlock();
				reentrantLock.unlock();
				reentrantLock.unlock();//不过，多少次加锁就要对应次数的释放锁
			}
		}
	}

	public static void main(String[] args) throws InterruptedException {
		ReenterLock lock = new ReenterLock();
		Thread t1 = new Thread(lock);
		Thread t2 = new Thread(lock);

		t1.start();t2.start();
		t1.join();t2.join();

		System.out.println(i);
	}
}
```

从上面的代码可以看出，重入锁有着显式操作过程。开发人会必须指定何时加锁，何时释放。突出一个灵活，但是在退出临界区时必须释放，否则其他的线程就无法进入。之所以叫**重入锁**是因为这种锁可以重复进入。当然这里的重复进入仅限一个线程。注意的是同一个线程多次获取锁就要多次释放锁。

### 中断响应

在等待锁的过程中，程序有可能根据需要取消对锁的请求。比如你和朋友约好了一起打球，如果你等了半小时朋友还没到，突然接到电话说有急事不能如约，那你就没必要在等待了。

```java

/**
 * DESCRIPTION：中断响应
 */
public class IntLock implements Runnable {
	public static ReentrantLock lock1 = new ReentrantLock();
	public static ReentrantLock lock2 = new ReentrantLock();
	int lock;

	/**
	 * 控制加锁顺序，方便构造死锁
	 * @param lock
	 */
	public IntLock(int lock) {
		this.lock = lock;
	}

	@Override
	public void run() {
		try {
			if(lock==1){
				lock1.lockInterruptibly();//lockInterruptibly 在等待锁的过程中，可以响应中断
				try {
					Thread.sleep(500);
				}catch (InterruptedException e){
					e.printStackTrace();
				}
				lock2.lockInterruptibly();

			}else{
				lock2.lockInterruptibly();
				try {
					Thread.sleep(500);
				}catch (InterruptedException e){
					e.printStackTrace();
				}
				lock1.lockInterruptibly();
			}
		}catch (InterruptedException e){
			e.printStackTrace();
		}finally {
			if(lock1.isHeldByCurrentThread()){
				lock1.unlock();
			}
			if(lock2.isHeldByCurrentThread()){
				lock2.unlock();
			}

			System.out.println(Thread.currentThread().getId()+"：线程退出");
		}
	}

	public static void main(String[] args) throws InterruptedException {
		IntLock r1 = new IntLock(1);
		IntLock r2 = new IntLock(2);

		Thread t1 = new Thread(r1);
		Thread t2 = new Thread(r2);

		t1.start();t2.start();
		Thread.sleep(1000);

		//中断其中一个线程,这里t1真正执行完毕 t2线程放弃其任务而退出, 释放资源
		 t2.interrupt();
	}
}

```

线程t1、 t2启动后，t1先占用lock1，在占用lock2；t2先占用lock2，在请求占用lock1，因此很容易形成t1和t2之间相互等待。在代码63行主线程睡眠，t1、t2两线程处于死锁状态，66行t2被中断，故t2放弃了对lock1的申请，同事释放已获得lock2，这波操作是的t1可以真正顺利的完成任务。

### 锁申请等待限时

除了等待外部通知外，避免死锁的一种方式。有时候我们无法确定为何一个线程迟迟拿不到锁，可能是因为死锁，可能是因为饥饿。但如果我们给定一个等待时间，让线程自动放弃，那么对系统来说是很有意义的。

```java
import java.util.concurrent.locks.ReentrantLock;

/**
 * DESCRIPTION：无参tryLock() 如果锁被其他前程占有,不会等待而是立即返回false
 * 这种模式不会引起线程等待,故不会发生死锁
 *
 */
public class TryLock implements Runnable {

    public ReentrantLock lock1 = new ReentrantLock();
    public ReentrantLock lock2 = new ReentrantLock();
    int                  lock;

    public TryLock(int lock) {
        this.lock = lock;
    }

    @Override
    public void run() {
        if (lock == 1) {
            while (true) {
                try {
                    if (lock1.tryLock()) {
                        try {
                            Thread.sleep(500);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }

                    if (lock2.tryLock()) {
                        System.out.println(Thread.currentThread().getId() + ": My job done");
                        lock2.unlock();
                        return;
                    }
                } finally {
                    lock1.unlock();
                }
            }
        } else {
            while (true) {
                if (lock2.tryLock()) {
                    try {
                        try {
                            Thread.sleep(500);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }

                        if (lock1.tryLock()) {
                            System.out.println(Thread.currentThread().getId() + ": my job done");
                            lock1.unlock();
                            return;
                        }
                    } finally {
                        lock2.unlock();
                    }
                }
            }
        }
    }

    public static void main(String[] args) {
        TryLock r1 = new TryLock(1);
        TryLock r2 = new TryLock(2);

        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r2);

        t1.start();
        t2.start();

    }

}

```



### 公平锁

大多数情况下，所得获得都是非公平的（比如使用synchronized关键字进行锁控制）。系统只是在这个锁的等待队列随机挑选一个。而公平锁不是这样，它会按照时间的先后顺序保证先到先得，后到后得。公平锁的一大特点就是**不会产生饥饿现象**

```java
public class FairLock implements Runnable{

	public static ReentrantLock fairLock = new ReentrantLock(true); //fair参数为true公平锁
	@Override
	public void run() {
		while (true){
			try {
				fairLock.lock();
				System.out.println(Thread.currentThread().getName()+"获得锁");
			} finally {
				fairLock.unlock();
			}
		}
	}

	public static void main(String[] args) {
		FairLock r1 = new FairLock();
		Thread t1 = new Thread(r1,"Thread_t1");
		Thread t2 = new Thread(r1,"Thread_t2");

		t1.start();t2.start();
	}
}
```

fair参数为true时表示锁是公平的。但是要实现公平锁就必须要维护一个有序队列，因此公平锁的成本比较高，性能相对非常低下。因此默认是非公平的。



### 重入锁的好搭档：Condition条件

如果理解了Object.wait()和Object.notify()方法的话，那么久很容易理解Condition对象了，它可以让线程在合适的时间等待，或者某一特定时刻得到通知。Condition接口的基本方法如下：

- await(): 当前线程等待，同时释放当前锁；当其他线程使用signal或signalAll，线程会重新获得锁并运行。或者当线程中断时，也能跳出等待，和Object.wait()很相似。
- awaitUninterruptibly: 和wait方法基本相同，只是不会再等待过程中响应中断。
- signal：唤醒一个等待中的线程。

```java
public class ReenterLockCondition implements Runnable {

	public static ReentrantLock lock = new ReentrantLock();
	public static Condition condition = lock.newCondition();

	@Override
	public void run() {
		try {
			lock.lock();
			condition.await();
			System.out.println("Thread is going on");
		} catch (Exception e) {
			e.printStackTrace();
		}finally {
			lock.unlock();
		}
	}

	public static void main(String[] args) throws InterruptedException {
		ReenterLockCondition rlc =new ReenterLockCondition();
		Thread t1 = new Thread(rlc);

		t1.start();
		t1.sleep(2000);

		lock.lock();
		condition.signal();//主线程main通知线程t1继续执行
		lock.unlock(); //释放相关的锁，谦让给被唤醒的线程，让他可以继续执行。如果无此行代码，虽然ti被唤醒，依然不能重新获得锁。
	}
}
```

在JDK内部，重入锁和Condition被大量使用，以ArrayBlockingQueue为例 看下其take，put的实现···



### 允许多个线程同时访问：信号量Semaphore

**缘起：**无论是内部锁synchronized还是重入锁ReentrantLock, 一次只能允许一个线程访问一个资源，而要指定多个线程访问同一资源呢？

信号量就可以满足以上需求，在构造信号量时，必须指定信号量的准入数，即可同时申请多少个许可，即相当于同时又多少个线程访问同一资源。

```java
/**
 * DESCRIPTION：信号量：允许多个线程同时访问
 *
 *
 */
public class SemaphoreDemo implements Runnable{
	final Semaphore semaphore = new Semaphore(5);
	@Override
	public void run() {
		try {
			semaphore.acquire();

			//模拟耗时
			Thread.sleep(2000);
			System.out.println(Thread.currentThread().getId()+":done");

			semaphore.release();

		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

	public static void main(String[] args) {
		ExecutorService executorService = Executors.newFixedThreadPool(20);
		final SemaphoreDemo demo = new SemaphoreDemo();
		for (int i = 0; i < 20; i++) {
			executorService.execute(demo);
		}
	}
}
```

第七行申明了一个包含五个许可的信号量，意味着同时可以有5个线程进入代码段14-15。申请时使用acquire，离开时务必release，不然就会发生信号量泄露。



### ReadWriteLock读写锁

读写锁可以有效帮助减少锁竞争，以提升系统性能。

|        | 读     | 写   |
| ------ | ------ | ---- |
| **读** | 非阻塞 | 阻塞 |
| **写** | 阻塞   | 阻塞 |

```java
public class ReadWriteLockDemo {
	private static Lock lock = new ReentrantLock();
	private static ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();
	private static Lock readLock = readWriteLock.readLock();
	private static Lock writeLock = readWriteLock.writeLock();
	private int value;

	public static void main(String[] args) {
		final ReadWriteLockDemo demo = new ReadWriteLockDemo();
		Runnable readRunable = new Runnable() {
			@Override
			public void run() {
				try {
					demo.handleRead(readLock);
//					demo.handleRead(lock);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		};

		Runnable writeRunnable = new Runnable() {
			@Override
			public void run() {
				demo.handleWrite(writeLock,new Random().nextInt());
//				demo.handleWrite(lock,new Random().nextInt());
			}
		};

		for (int i = 0; i < 18; i++) {
			new Thread(readRunable).start();
		}

		for (int i =18; i < 20; i++) {
			new Thread(writeRunnable).start();
		}
	}

	public Object handleRead(Lock lock) throws InterruptedException {
		lock.lock();
		try {
			//模拟读操作
			Thread.sleep(1000);
			return value;
		}finally {
			lock.unlock();
		}
	}

	public void handleWrite(Lock lock,int index){
		lock.lock();
		try {
			//模拟写操作
			Thread.sleep(1000);
			value = index;

		} catch (InterruptedException e) {
			e.printStackTrace();
		}finally {
			lock.unlock();
		}
	}
}
```

如果使用普通锁代替读写锁，那么所有的读写线程之间必须相互等待。



### 倒计时器：CountDownLatch

这个工具是用来控制线程等待，它可以让某一线程等待直到倒计时结束，再开始执行。

```java
public class CountDownLatchDemo implements Runnable{
	static final CountDownLatchDemo demo = new CountDownLatchDemo();
	static final CountDownLatch end = new CountDownLatch(10);
	@Override
	public void run() {
		//模拟检查任务
		try {
			Thread.sleep(new Random().nextInt(10)*100);
			System.out.println("check complete");
			end.countDown(); //通知CountDownLatch，一个线程已经完成了任务
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

	public static void main(String[] args) throws InterruptedException {
		ExecutorService executorService = Executors.newFixedThreadPool(10);
		for (int i = 0; i < 21; i++) {
			executorService.submit(demo);
		}
		//等待检查
		end.await(); //CountDownLatch使用await方法，要求主线程等待所有10个检查任务全部完成，主线程才能继续执行
		//发射火箭
		System.out.println("Fire");
		executorService.shutdown();
	}
}
```



### 循环栅栏：CyclicBarrier

CyclicBarrier是另一种多线程并发控制工具，和CountDownLatch非常类似，它可以实现线程间的计算等待，但他的功能比CountDowLatch更加复杂且强大。

Cyclic意为循环，也就是这个计数器可以循环使用过，凑齐一批，计数器就会归零，然后继续凑下一批，而CountDownLatch只能用一次。CyclicBarrier更强大的是可以接受一个barrierActiion作为参数，即计数器技术完成后，系统会执行的动作。

```java
public class CyclicBarrierDemo {

	public static void main(String[] args) {
		final int N = 10;
		Thread[] allSoldier = new Thread[N];
		boolean flag = false;
		CyclicBarrier cyclicBarrier = new CyclicBarrier(N,new BarrierRun(flag,N));

		System.out.println("```````````````集合```````````````");
		for (int i = 0; i < N; i++) {
			System.out.println("士兵"+i+"报道");
			allSoldier[i] = new Thread(new Soldier("士兵"+i,cyclicBarrier));
			allSoldier[i].start();
		}
	}

	public static class Soldier implements Runnable{
		private String soldier;
		private final CyclicBarrier cyclicBarrier;

		public Soldier(String soldier, CyclicBarrier cyclicBarrier) {
			this.soldier = soldier;
			this.cyclicBarrier = cyclicBarrier;
		}

		void doWork(){
			try {
				Thread.sleep(Math.abs(new Random().nextInt()%10000));
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println(soldier+":任务完成");
		}

		@Override
		public void run() {
			//等待所有士兵到齐
			try {
				cyclicBarrier.await();
			} catch (InterruptedException e) {
				e.printStackTrace();
			} catch (BrokenBarrierException e) {
				e.printStackTrace();
			}
		}
	}

	public static class BarrierRun implements Runnable{
		boolean flag;
		int N;

		public BarrierRun(boolean flag, int n) {
			this.flag = flag;
			N = n;
		}

		@Override
		public void run() {
			if(flag){
				System.out.println("司令[士兵"+N+"个，任务完成]");
			}else{
				System.out.println("司令[士兵"+N+"个，集合完毕]");
				flag = true;
			}
		}
	}
}

```



### 线程阻塞工具 LockSupport

LockSupport可以在线程的任意位置让线程阻塞。和Thread.suspend()相比，它弥补了在在resume()之前线程不能继续执行的情况。和Object.wait()相比，它不需要先获得某个对象的锁，也不会抛出InterruptedException。

主要静态方法：
- park
- unpark
- 类似的还是有parkNano parkUntil等

```java
public class LockSupportDemo {
	public static Object u = new Object();
	static ChangeObjectThread t1 = new ChangeObjectThread("t1");
	static ChangeObjectThread t2 = new ChangeObjectThread("t2");

	public static void main(String[] args) throws InterruptedException {
		t1.start();
		Thread.sleep(100);
		t2.start();

		LockSupport.unpark(t1);
		LockSupport.unpark(t2);

		t1.join();
		t2.join();
	}

	public static class ChangeObjectThread extends Thread{
		public ChangeObjectThread(String name) {
			super(name);
		}

		@Override
		public void run() {
			synchronized (u){
				System.out.println("in "+getName());
			}
		}
	}
}
```

我们无法保证unpark在park之后执行，但是执行这段代码自始至终可以正常执行，不会因为park方法导致线程永久性的挂起。这是因为LockSupport使用了类似信号量的机制。他为每一个线程准备了一个许可，如果许可是可用状态，park（）函数会立即返回并消费这个许可（**将许可变为不可用**），如果不可用就阻塞。而unpark（）将许可将状态变为可用（和信号量不同的是许可只有一个，不可累加）。所以永远可以顺利结束。

同时，处于park（）挂起状态的线程不会像suspend那样还给出一个令人费解的Runnable的状态，它非常明确地给出一个WAITING状态，甚至还会标注是park（）引起的。park方法还可以接受一个对象，为当前线程设置阻塞对象，并且该对象会出现在线程Dump中。

除了定时阻塞功能外，LockSupport还支持中断影响，但是其他的接收中断函数不一样，LockSupport.park()不会抛出InterruptedException，它只会默默的返回。