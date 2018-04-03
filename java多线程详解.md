## Java多线程详解
- 进程：
	- 想要了解线程，必须先了解进程，因为线程是依赖于进程存在的。
		- 进程：程序运行时才会出现进程，我们可以通过windows任务管理器查看到进程的存在；
		- 进程就是系统进行资源分配和调用的独立单位，每一个进程都有其自己的内存空间和系统资源；
- 多进程：
	- 首先，单进程只能做一件事，当这件事没有做完事你不能做其他事；
	- 而多进程可以在**一段时间内**执行多个任务，并且提高CPU的使用率；
	- 但在这个**一段时间内**执行多个任务，它并不是同时进行的，因为CPU在某个时间点上只能做一件事，
	- 而又因为CPU在程序之间做着高效切换的速度，让我们几乎觉得就是同时进行的；
- 线程：
	- 同一个进程内执行多个任务，这其中的每一个任务即可看作为一根线程；
	- 线程：程序的执行单元，执行路径，CPU的最基本单位；
	- 单线程：如果程序只有一条执行路径或一个执行单元；
	- 多线程；程序有多条执行路径；
- 多线程：
	- **多线程的存在，不是为了提高程序的执行速度，而是为了提高程序的使用率**；
	- 程序的执行其实都是在抢CPU的资源，CPU的执行权。
	- 多个进程是在抢这个资源，而其中的某一个进程如果执行路径(线程)比较多，就会有更高的几率抢到CPU的执行权。
	- 我们是不敢保证哪一个线程能够在哪个时刻抢到，所以**线程的执行具有随机性**。
- 如何启动线程？
	- 一般来说，常用的有两种方式：1.继承`Thread`类；2.实现`Runnable`接口；
- 一、继承`Thread`类，创建一个类名为`MyThread`继承`Thread`：

		public class MyThread extends Thread {
		
			@Override
			public void run() {
				//书写你要执行的任务代码
			}
		
		}

- 创建一个有`main()`方法的类，创建MyThread对象并执行`start()`方法：

		public class MyThreadDemo {
			public static void main(String[] args) {
				MyThread thread = new MyThread();
				thread.start();
			}
		
		}

- 此时一条线程创建成功了，但是请往下看：


		public class MyThreadDemo {
			public static void main(String[] args) {
				MyThread thread = new MyThread();
				//同一根线程调用两次
				thread.start();
				thread.start();
			}
		
		}

- 此时会抛出`java.lang.IllegalThreadStateException`异常-->线程状态非法异常！
	- 原因：
		- 1.当你第一次执行调用并启动该线程时，此时线程已经启动，你再去调用它的启动方法是错误的；
		- 2.线程的生命周期是一个不可循环的过程，一个线程对象结束了就不能再次start；
	- 解决：
		- 分别创建两个对象，代表两个线程，然后分别启动它们即可；
- 如下：


		public class MyThreadDemo {
			public static void main(String[] args) {
				MyThread thread1 = new MyThread();
				MyThread thread2 = new MyThread();
				thread1.start();
				thread2.start();
			}
		
		}

- 疑问：
	- 我们发现在`MyThread`类中有继承了`Thread`了，也覆盖了它的`run()`方法，为何不能直接调用呢？
	- 在`MyThreadDemo`中，既然已经把对象new出来了，为什么不直接调用它的`run()`方法而是要调用`start()`方法呢？
- 原因：
	- run方法：如果通过new出来的对象去调用其run方法，那么这个**只能当作一个普通方法来使用**，系统并不会告诉JVM去创建一个新的线程，也就是说，只有一个主线程，必须也要等到其run方法内部的代码执行完毕后，才会继续往下执行代码；
	- start方法：通过该方法启动线程，系统会通知JVM去创建一个新的线程，真正实现了多线程；无需等待run()方法内部的代码执行完毕，此时start方法处于就绪状态，多根线程根据CPU分配的时间片去抢CPU的执行优先权，所以这也是为什么线程具有随机性的原因，当多个线程内的run方法都执行完毕，此时线程也就终止了；
- 我们来验证一下：

		public class Demo {
		    public static void main(String[] args) {
		        Thread t1 = new Thread(){
		            public void run() {
		                B();
		            }
		        };
		        t1.run();
		        System.out.println("AA");
		    }
		
		    static void B() {
		        System.out.println("BB");
		    }
		}
- 那么输出结果为："BB AA"；因为`t1.run()`实际上就是等待new Thread完毕后调用其内部的`run()`方法，而`run()`方法内部又调用了`B()`方法，从而先输出BB，当run执行完毕后再输出AA，所以实际上并不是一根线程；
- 那么，如果将`t1.run()`换成`t1.start()`呢？那么输出结果很明显是："AA BB"；因为，当程序执行到此处的时候，系统告诉JVM去创建了一个线程并处于就绪状态，代码继续往下执行，先输出AA；执行该线程得到CPU的时间片，开始执行`run()`方法，打印出了BB；

- 至于为什么，我们来看看源码：

	    public synchronized void start() {
	        /**
	         * This method is not invoked for the main method thread or "system"
	         * group threads created/set up by the VM. Any new functionality added
	         * to this method in the future may have to also be added to the VM.
	         *
	         * A zero status value corresponds to state "NEW".
	         */
	        if (threadStatus != 0)
	            throw new IllegalThreadStateException();
	
	        /* Notify the group that this thread is about to be started
	         * so that it can be added to the group's list of threads
	         * and the group's unstarted count can be decremented. */
	        group.add(this);
	
	        boolean started = false;
	        try {
	            start0();
	            started = true;
	        } finally {
	            try {
	                if (!started) {
	                    group.threadStartFailed(this);
	                }
	            } catch (Throwable ignore) {
	                /* do nothing. If start0 threw a Throwable then
	                  it will be passed up the call stack */
	            }
	        }
	    }
	
	    private native void start0();

- 从源码中可以看到，当一个线程启动的时候，如果它的状态不为0则抛出`IllegalThreadStateException`异常；反之，该线程会被加入到线程组内，并调用`start0()`方法，而此方法是由`native`修饰的私有方法，我们无法看到其内部的实现过程，因为Native Method是java调用非java代码的一个接口，它的底层是由C语言去实现的，而只有C或者C++才可以调用系统资源；
- 我们再来看看`run()`方法的源码：

		@Override
		    public void run() {
		        if (target != null) {
		            target.run();
		        }
		    }
- 要执行run方法，必须要是target不为空，那么target是什么？

	    /* What will be run. */
	    private Runnable target;
- target是一个由Runnable接口的一个变量；我们阅读源码也会发现，其实`Thread`类也是实现了`Runnable`接口的，那么我们实现`Thread`的`run()`方法其实就是实现了`Runnable`接口的`run()`方法；所以当你直接调用`run()`方法，实际上它并没有走到`start0()`，就不会创建出一个线程，就只是单纯的执行了一个普通的方法！

- 那么既然继承`Thread`类或者实现`Runnable`接口，最终都是实现`Runnable`的`run()`方法，那么为什么要这样继承`Thread`类而不直接实现`Runnable`接口就可以了呢？实现`Runnable`接口和继承`Thread`类之间有何区别？
- 我们先看看一段代码：
- 继承`Thread`类：
![](https://i.imgur.com/cIHPhpn.png)
- 然后输出的结果是：
![](https://i.imgur.com/bizNaHu.png)


- 接着我们看实现Runnable接口：
![](https://i.imgur.com/FSjrQlV.png)
- 输出的结果是这样的：
![](https://i.imgur.com/manBWlu.png)

- **结论：**
	- 很显然这两个结果是不同的，我们来分析下，继承`Thread`类来实现多线程，其实是创建了三个线程，并给这3个线程的每一根线程输出1-10的任务，这三根线程各自做各自的事情，各自完成各自的任务，互不干扰；
	- 因此我们可以认为：**通过继承Thread类来实现多线程，如果含有多根线程，线程与线程之间资源的互不干扰的，它们只负责完成分配的任务即可；**
	- 而通过实现`Runnabble`接口的，**其实是创建了三根线程，将输出1-10的这个任务平均分给了这三根线程，由这三根线程共同去完成；它们之间可以互相共享资源、同做一件事；**
- 如果你想用第二种方式实现第一种的效果，你可以这么做：
![](https://i.imgur.com/i5opPei.png)

- 总结：
	- 基于java类只能单继承，一个类可以实现多个接口的关系，**通过继承Thread类，不能一个实例创建多个线程，因此其实例内部的成员变量是不能共享的；但通过实现Runnable接口的实例，一个Runnable实例可以建立多个线程，因此其实例变量是可以共享的；**当然，继承Thread类也能够共享变量，能共享Thread类的static变量；
	- 实现Runnable接口所具有的优势：
		- 继承Thread类就要创建一个实例对象，当线程很多时，因为付出的代价太大，内存消耗巨大；
		- 避免Java单继承的问题；
		- 适合多线程处理同一资源；
		- 代码可以被多线程共享，数据独立，很容易实现资源共享；
	
