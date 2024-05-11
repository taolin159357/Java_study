* 并发的级别主要有以下几种：

  1. **阻塞**：当一个线程是阻塞的，那么在其他线程释放资源之前，当前线程无法继续执行。例如，使用`synchronized`关键字或者其他重入锁时，线程会进入阻塞状态。如果线程试图执行后续代码但无法获得所需资源（如临界区的锁），它就会被挂起等待，直到拥有所需资源为止。

  2. **无饥饿（Starvation-Free）**：在并发环境中，线程之间可能存在优先级差异。为了避免优先级低的线程长时间得不到资源（即“饥饿”），公平锁被引入。公平锁保证线程按照时间先后顺序获取资源，从而避免饥饿现象。

  3. **无障碍（Obstruction-Free）**：无障碍是一种最弱的非阻塞调度。两个线程如果是无障碍的执行，那么他们不会因为临界区的问题导致一方被挂起。这种策略通常依赖于一个“一致性标记”来实现，大家一起修改共享数据，把数据改坏了，会回滚之前的操作以确保数据安全。但如果没有数据竞争发生，那么线程就可以顺利完成自己的工作，走出临界区。

  4. **无锁（Lock-Free）**：无锁并行都是无障碍的，并且它保证必然有一个线程能够在有限步内完成操作离开临界区。这意味着即使多个线程试图访问同一资源，至少有一个线程能够成功完成操作。

  5. **无等待（Wait-Free）**：无等待是在无锁的基础上进一步扩展的概念。它要求所有线程都必须在有限步内完成操作，即不存在任何线程需要等待其他线程完成操作的情况。


* Java内存模型（Java Memory Model，简称JMM）是一个抽象的概念，它并不真实存在，而是描述了一组规则或规范，通过这组规范定义了程序中各个变量的访问方式。JMM的主要目标是定义程序中每个变量的访问规则，即将变量存储在虚拟机中并从内存中取出变量。

  Java内存模型主要由两个区域组成：主内存（Main Memory）和工作内存（Working Memory，也叫本地内存Local Memory）。此外，Java内存模型还定义了五种操作，包括lock（锁定）、unlock（解锁）、read（读取）、load（载入）、use（使用）、store（存储）、write（写入）等，这些操作都必须在遵守JMM的规范下才能保证多线程操作的正确性。

  Java内存模型具有三大特性：

  1. **可见性**：指当一个线程修改了共享变量的值，其他线程能够立即知道这个修改。
  2. **原子性**：指一个操作或者多个操作要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。
  3. **有序性**：即程序执行的顺序按照代码的先后顺序执行。

  JMM的实现主要是通过重排序、三个同步原语（synchronize、volatile、final）、内存屏障构成的happen-before原则。重排序是指编译器和处理器为了优化程序性能对指令序列进行重新排序的手段。内存屏障是为了保证内存可见性，Java编译器在生成指令序列的适当位置会插入内存屏障指令来禁止特定类型的处理器重排序。

  此外，Java虚拟机（JVM）的内存模型也是Java内存管理的重要组成部分。JVM内存模型包括堆、方法区（元空间）、虚拟机栈（线程栈）、本地方法栈和程序计数器。堆是运行时数据区，所有类的实例和数组都是在堆上分配内存；方法区主要包括常量、静态变量、类信息、运行时常量池等；虚拟机栈为每个线程分配一块栈帧内存空间，用于存储局部变量和部分运算的中间结果。





##### 多线程

###### 1. Java并行程序基础

* Java 提供了创建线程的方法：
  1. 通过实现 Runnable 接口；
  2. 通过继承 Thread 类本身；
  3. 通过 Callable 和 Future （或FutureTask）创建线程；

```java
  public class CallableThreadTest implements Callable<Integer> {
      public static void main(String[] args)  
      {  
          CallableThreadTest ctt = new CallableThreadTest();  
          FutureTask<Integer> ft = new FutureTask<>(ctt);  
          for(int i = 0;i < 100;i++)  
          {  
              System.out.println(Thread.currentThread().getName()+" 的循环变量i的值"+i);  
              if(i==20)  
              {  
                  new Thread(ft,"有返回值的线程").start();  
              }  
          }  
          try  
          {  
              System.out.println("子线程的返回值："+ft.get());  
          } catch (InterruptedException e)  
          {  
              e.printStackTrace();  
          } catch (ExecutionException e)  
          {  
              e.printStackTrace();  
          }  
    
      }
      @Override  
      public Integer call() throws Exception  
      {  
          int i = 0;  
          for(;i<100;i++)  
          {  
              System.out.println(Thread.currentThread().getName()+" "+i);  
          }  
          return i;  
      }  
  }
```

* 如何理解实现Callable接口的方式创建多线程比实现Runnable接口创建多线程方式强大？

  * call()可以有返回值的。
  * call()可以抛出异常，被外面的操作捕获，获取异常的信息
  * Callable是支持泛型的

![](/Users/taolin/Documents/Java_study/86.png)

* > Thread中的常用方法：
  >
  >  1. start():启动当前线程；调用当前线程的run()
  >
  >  2. run(): 通常需要重写Thread类中的此方法，将创建的线程要执行的操作声明在此方法中。
  >
  >  3. stop():已过时。当执行此方法时，强制结束当前线程。
  >
  >     stop()方法在结束线程时，会直接终止线程，并且会立即释放这个线程所有的锁。而这些锁是用来维持对象一致性的。如果此时，写线程正在写入数据但还没有完成，并强行被stop()方法终止，那么对象就会被写坏，同时由于锁已经释放，另外一个等待该锁的读线程就会读取到这个不一致的对象，从而导致系统处于不一致的状态。此外，由于线程在终止时可能不会按照预期的方式释放资源，这也可能导致资源泄露和其他问题。
  >
  >     在Java中，有几种安全的方式来停止线程，而不是使用已经被废弃的`stop()`方法。以下是一些推荐的方法：
  >
  >     1. **使用标志位来停止线程**：在线程内部使用一个`volatile`布尔变量作为标志位，当需要停止线程时，改变这个变量的值。线程在执行过程中会检查这个标志位，如果发现需要停止，则执行清理工作并退出。
  >
  >        `volatile`关键字是一种轻量级的同步机制，用于确保变量的可见性（实时更新到内存）和有序性（禁止重排序）。但是，它并不能替代所有的锁机制，因为`volatile`不提供原子性保证。
  >
  >     2. **使用`interrupt()`方法**：当线程被中断时，如果它正在等待、睡眠或占用其他资源，那么`interrupt()`方法会唤醒它，并抛出一个`InterruptedException`。线程应该捕获这个异常，并执行适当的清理工作。
  >
  >     3. **使用`Future`和`ExecutorService`**：如果你使用的是`ExecutorService`来管理线程，那么可以使用`Future`对象来取消任务的执行。`Future.cancel(boolean mayInterruptIfRunning)`方法可以用来尝试取消任务的执行。如果参数`mayInterruptIfRunning`为`true`，那么会尝试中断正在执行的任务。
  >
  >  4. `isInterrupted()` 是 Java 中 `Thread` 类的一个实例方法，用于判断当前线程是否被中断。当一个线程调用 `interrupt()` 方法时，它会设置自身的中断状态为 `true`。`isInterrupted()` 方法则用于检查这个中断状态。
  >
  >     在多线程编程中，我们经常需要控制和管理线程的行为。`isInterrupted()` 方法可以帮助我们检查线程的中断状态，并据此做出相应的处理。比如，在一个线程的执行循环中，我们可以周期性地使用 `isInterrupted()` 方法来检查线程是否被中断，然后在合适的时候退出任务。
  >
  >     值得注意的是，`isInterrupted()` 仅会返回中断状态，而不会清除中断状态，与 `isInterrupted()` 不同的是，`Thread` 类还有一个静态方法 `interrupted()`，这个方法也会检查当前线程的中断状态，但会清除中断状态。这意味着，如果连续两次调用 `interrupted()`，并且在这两次调用之间没有其他线程调用 `interrupt()`，那么第二次调用将返回 `false`，因为第一次调用已经清除了中断状态。
  >
  >     ```java
  >     public class InterruptExample extends Thread {  
  >         public void run() {  
  >             try {  
  >                 while (!Thread.currentThread().isInterrupted()) {  
  >                     // 执行一些工作...  
  >                     // 模拟耗时操作  
  >                     Thread.sleep(1000);  
  >                 }  
  >             } catch (InterruptedException e) {  
  >                 // 清除中断状态（可选）  
  >                 // Thread.currentThread().interrupt();  
  >                 // 处理中断...  
  >                 System.out.println("Thread was interrupted.");  
  >             }  
  >             // 清理工作...  
  >         }  
  >
  >         public static void main(String[] args) throws InterruptedException {  
  >             InterruptExample example = new InterruptExample();  
  >             example.start();  
  >
  >             // 让主线程等待一段时间，然后中断示例线程  
  >             Thread.sleep(2000);  
  >             example.interrupt();  
  >         }  
  >     }
  >     ```
  >
  >  5. 线程的通信
  >
  >     * 涉及到的三个方法：
  >
  >       * wait()：一旦执行此方法，当前线程就进入阻塞状态，并释放同步监视器。
  >
  >       * notify()：一旦执行此方法，就会唤醒被wait的一个线程。如果有多个线程被wait，就唤醒优先级高的那个。
  >
  >       * notifyAll()：一旦执行此方法，就会唤醒所有被wait的线程。
  >
  >     * 说明：
  >
  >       wait()，notify()，notifyAll()三个方法必须使用在同步代码块或同步方法（synchronized）中。
  >
  >       wait()，notify()，notifyAll()三个方法的调用者必须是同步代码块或同步方法中的同步监视器。否则，会出现IllegalMonitorStateException异常
  >
  >       wait()，notify()，notifyAll()三个方法是定义在java.lang.Object类中。
  >
  >     * 面试题：sleep() 和 wait()的异同？
  >
  >       * 相同点
  >
  >         一旦执行方法，都可以使得当前的线程进入阻塞状态。
  >
  >       * 不同点
  >
  >         两个方法声明的位置不同：Thread类中声明sleep() , Object类中声明wait()
  >
  >         调用的要求不同：sleep()可以在任何需要的场景下调用。 wait()必须使用在同步代码块或同步方法中
  >
  >         关于是否释放同步监视器：如果两个方法都使用在同步代码块或同步方法中，sleep()不会释放锁，wait()会释放锁。
  >
  >
  >       ```java
  >     public class WaitNotifyExample {  
  >         private int count = 0;  
  >         private final Object lock = new Object();  
  >         
  >         // 增加计数的线程  
  >         private class IncrementThread extends Thread {  
  >             @Override  
  >             public void run() {  
  >                 for (int i = 0; i < 10; i++) {  
  >                     synchronized (lock) {  
  >                         while (count == 5) {  
  >                             try {  
  >                                 // 当count等于5时，当前线程等待  
  >                                 lock.wait();  
  >                             } catch (InterruptedException e) {  
  >                                 e.printStackTrace();  
  >                             }  
  >                         }  
  >                         count++;  
  >                         System.out.println("Incremented count to " + count);  
  >                         // 唤醒其他可能正在等待的线程  
  >                         lock.notify();  
  >                     }  
  >                     try {  
  >                         // 暂停一段时间，以便观察线程交替执行的情况  
  >                         Thread.sleep(100);  
  >                     } catch (InterruptedException e) {  
  >                         e.printStackTrace();  
  >                     }  
  >                 }  
  >             }  
  >         }  
  >         
  >         // 减少计数的线程  
  >         private class DecrementThread extends Thread {  
  >             @Override  
  >             public void run() {  
  >                 for (int i = 0; i < 10; i++) {  
  >                     synchronized (lock) {  
  >                         while (count == 0) {  
  >                             try {  
  >                                 // 当count等于0时，当前线程等待  
  >                                 lock.wait();  
  >                             } catch (InterruptedException e) {  
  >                                 e.printStackTrace();  
  >                             }  
  >                         }  
  >                         count--;  
  >                         System.out.println("Decremented count to " + count);  
  >                         // 唤醒其他可能正在等待的线程  
  >                         lock.notify();  
  >                     }  
  >                     try {  
  >                         // 暂停一段时间，以便观察线程交替执行的情况  
  >                         Thread.sleep(100);  
  >                     } catch (InterruptedException e) {  
  >                         e.printStackTrace();  
  >                     }  
  >                 }  
  >             }  
  >         }  
  >         
  >     //   在main方法中，我们创建了两个线程实例并启动它们。这两个线程将交替执行，增加和减少count的值，直到各自执行了10次。
  >         public static void main(String[] args) {  
  >             WaitNotifyExample example = new WaitNotifyExample();  
  >             Thread incrementThread = new Thread(example.new IncrementThread());  
  >             Thread decrementThread = new Thread(example.new DecrementThread());  
  >         
  >             // 启动线程  
  >             incrementThread.start();  
  >             decrementThread.start();  
  >         }  
  >     }
  >       ```
  >
  >  6. Java的`suspend()`和`resume()`方法是早期用于暂停和恢复线程执行的方法。它们定义在`Thread`类中，允许开发者直接控制线程的状态。然而，这两个方法现在已经被标记为过时（deprecated），并不推荐使用。
  >
  >     `suspend()`方法会暂停线程的执行，而`resume()`方法则会恢复被`suspend()`方法暂停的线程。这两个方法简单直观，看起来很方便，但在实际使用中存在很多隐患，主要因为它们可能导致线程死锁或其他不可预测的行为。具体来说，当调用一个线程的`suspend()`方法时，该线程会被立即暂停，但并不会释放它所持有的任何锁。这意味着如果线程在持有锁的情况下被暂停，那么其他需要这个锁的线程就会一直被阻塞，从而导致死锁。同样，当使用`resume()`方法恢复线程时，也无法保证线程能够立即获得它之前持有的锁，这同样可能导致不可预测的行为。
  >
  >     因此，为了避免这些问题，现代Java编程推荐使用`wait()`、`notify()`和`notifyAll()`方法，或者`Lock`和`Condition`接口来实现线程的暂停和恢复。这些方法提供了更精细的控制，并且能够更好地处理并发问题。
  >
  >  7. 在Java中，`join`方法用于等待一个线程完成其执行。当调用一个线程的`join`方法时，当前线程会被阻塞，直到被调用的线程执行完毕。这种方法对于控制线程的执行顺序非常有用，特别是在需要保证一个线程在另一个线程之前完成其任务的情况下。
  >
  >     ```java
  >     public class JoinExample {
  >         public static void main(String[] args) {
  >             // 创建一个线程对象
  >             Thread thread = new Thread(new MyRunnableTask());
  >     
  >             // 启动线程
  >             thread.start();
  >     
  >             // 等待线程完成
  >             try {
  >                 thread.join();
  >             } catch (InterruptedException e) {
  >                 e.printStackTrace();
  >                 // 如果当前线程在等待时被中断，则可以选择处理中断或者恢复中断状态
  >                 Thread.currentThread().interrupt();
  >             }
  >     
  >             // 线程完成后的操作
  >             System.out.println("主线程继续执行，子线程已结束。");
  >         }
  >     
  >         static class MyRunnableTask implements Runnable {
  >             @Override
  >             public void run() {
  >                 // 模拟一个耗时任务
  >                 for (int i = 0; i < 5; i++) {
  >                     System.out.println("子线程正在执行任务: " + i);
  >                     try {
  >                         Thread.sleep(1000); // 休眠1秒
  >                     } catch (InterruptedException e) {
  >                         e.printStackTrace();
  >                     }
  >                 }
  >                 System.out.println("子线程任务完成。");
  >             }
  >         }
  >     }
  >     ```
  >
  >     在这个例子中，`MyRunnableTask`实现了`Runnable`接口，并在`run`方法中模拟了一个耗时任务。当我们在`main`方法中启动这个线程后，调用`thread.join()`方法会让主线程等待，直到`thread`线程执行完毕。在`thread.join()`调用期间，主线程会阻塞，不会继续执行后续的`System.out.println`语句。
  >
  >     当`thread`线程执行完其`run`方法中的所有代码后，主线程才会从`thread.join()`调用中返回，然后执行后面的打印语句。这就保证了子线程在主线程继续执行之前完成了它的任务。
  >
  >     需要注意的是，如果在等待过程中主线程被中断（通过调用它的`interrupt`方法），`join`方法会抛出`InterruptedException`异常。在这种情况下，你可以选择处理这个异常，比如打印堆栈跟踪信息，或者恢复中断状态（通过再次调用`interrupt`方法），这样可以让线程有机会响应中断。
  >
  >  8. 在Java中，`yield`方法用于提示线程调度器当前线程愿意放弃对CPU的占用，以使其他等待的线程有机会得到执行。需要注意的是，`yield`方法并不会阻塞当前线程，也不会使线程进入等待状态，它只是向线程调度器表达一个建议，具体是否真正放弃CPU的占用权取决于线程调度器的实现。
  >
  >     由于`yield`的行为取决于具体的线程调度器，因此它不能用来精确控制线程的执行顺序。不过，在某些情况下，使用`yield`可以帮助提高系统的并发性能，尤其是在有大量相似优先级的线程竞争CPU资源时。
  >
  >     ```java
  >     public class YieldExample {
  >         public static void main(String[] args) {
  >             // 创建并启动两个线程
  >             Thread thread1 = new Thread(new YieldingTask("线程1"));
  >             Thread thread2 = new Thread(new YieldingTask("线程2"));
  >     
  >             thread1.start();
  >             thread2.start();
  >         }
  >     
  >         static class YieldingTask implements Runnable {
  >             private final String name;
  >     
  >             public YieldingTask(String name) {
  >                 this.name = name;
  >             }
  >     
  >             @Override
  >             public void run() {
  >                 for (int i = 0; i < 5; i++) {
  >                     System.out.println(name + "正在执行任务: " + i);
  >     
  >                     // 模拟一个耗时的操作
  >                     try {
  >                         Thread.sleep(100);
  >                     } catch (InterruptedException e) {
  >                         e.printStackTrace();
  >                     }
  >     
  >                     // 提示线程调度器放弃CPU占用
  >                     Thread.yield();
  >                 }
  >             }
  >         }
  >     }
  >     ```
  >
  >     在这个例子中，我们创建了两个线程`thread1`和`thread2`，它们都执行相同的`YieldingTask`任务。在`run`方法的循环中，每个线程执行一个模拟的耗时操作，然后调用`Thread.yield()`方法。这表示当前线程愿意放弃CPU的占用，以便其他线程可以得到执行的机会。
  >
  >     然而，需要注意的是，由于`yield`方法仅仅是建议性的，实际的行为取决于操作系统的线程调度策略。因此，在实际应用中，我们不应该依赖`yield`来控制线程的执行顺序或确保某个线程得到优先执行。如果需要精确控制线程的执行顺序，应该使用`join`方法或其他同步机制，如`synchronized`块、`Lock`接口或`Semaphore`等。
  >
  >  9. `Thread.sleep()` 方法在 Java 中用于使当前执行的线程休眠（暂停执行）指定的毫秒数。当休眠时间结束后，线程会自动恢复执行。
  >
  >     正常情况下，`Thread.sleep()` 方法在休眠完成后不会抛出任何异常。但是，如果在线程休眠期间发生中断（即调用了该线程的 `interrupt()` 方法），那么 `Thread.sleep()` 会抛出一个 `InterruptedException`。
  >
  >     因此，当使用 `Thread.sleep()` 时，通常建议将其放在一个 `try-catch` 块中，以便捕获并适当处理可能的 `InterruptedException`。
  >
  >  10. 其他方法
  >
  >     currentThread():静态方法，返回执行当前代码的线程
  >    
  >     getName():获取当前线程的名字
  >    
  >     setName():设置当前线程的名字
  >    
  >     isAlive():判断当前线程是否存活
  >
  > 
  >
  >   线程的优先级：
  >
  >  1. MAX_PRIORITY：10，MIN _PRIORITY：1，NORM_PRIORITY：5  -->默认优先级
  >
  >  2. 如何获取和设置当前线程的优先级：getPriority():获取线程的优先，setPriority(int p):设置线程的优先级
  >
  >  3. 说明：高优先级的线程要抢占低优先级线程cpu的执行权。但是只是从概率上讲，高优先级的线程高概率的情况下被执行。并不意味着只有当高优先级的线程执行完以后，低优先级的线程才执行。
  >
  > 
  >
  > * 在Java中，`setDaemon(boolean on)` 是 `Thread` 类的一个方法，用于设置线程为守护线程（daemon thread）或非守护线程（非daemon thread）。守护线程是一种在后台为其他线程（通常是用户线程）提供服务的线程。
  >
  >   当你调用线程的 `setDaemon(true)` 方法时，你就将该线程设置为守护线程。守护线程具有以下几个关键特性：
  >
  >   1. **在JVM退出时自动终止**：当所有非守护线程都已经结束运行时，无论守护线程是否还在运行，JVM都会退出。这意味着守护线程不应该执行任何重要的、需要在程序结束时完成的操作。
  >
  >   2. **优先级较低**：守护线程通常用于执行后台任务，如垃圾回收或一些系统级任务，它们的优先级通常较低。
  >
  >   3. **不能阻止JVM退出**：即使守护线程还在运行，只要所有非守护线程都已经结束，JVM仍然会退出。
  >
  >   4. **启动限制**：守护线程必须在启动任何非守护线程之前设置。一旦线程启动，就不能再改变其守护状态。
  >
  >   ```java
  >   public class DaemonThreadExample {
  >       public static void main(String[] args) {
  >           // 创建一个新线程，并将其设置为守护线程
  >           Thread daemonThread = new Thread(() -> {
  >               while (true) {
  >                   System.out.println("Daemon thread is running...");
  >                   try {
  >                       Thread.sleep(1000);
  >                   } catch (InterruptedException e) {
  >                       e.printStackTrace();
  >                   }
  >               }
  >           });
  >           daemonThread.setDaemon(true); // 设置线程为守护线程
  >           daemonThread.start(); // 启动线程
  >   
  >           // 主线程休眠2秒后结束，此时JVM中只有守护线程在运行
  >           try {
  >               Thread.sleep(2000);
  >           } catch (InterruptedException e) {
  >               e.printStackTrace();
  >           }
  >           System.out.println("Main thread is exiting...");
  >       }
  >   }
  >   ```
  >
  >   在这个例子中，我们创建了一个守护线程，它会在无限循环中运行并打印消息。然后主线程休眠2秒后结束。由于此时JVM中只剩下守护线程在运行，因此JVM会立即退出，即使守护线程还在循环中。
  >
  >   请注意，在实际应用中，应该避免在守护线程中执行重要的清理工作或需要持久化的操作，因为守护线程可能在任何时候被终止。
  >
  > * 在Java中，`ThreadGroup`类被设计用于将多个线程组合成一个线程组，以便于线程的管理和控制。然而，值得注意的是，随着Java的发展，线程组的使用已经变得相对不那么普遍了，因为现代的并发编程更多地依赖于并发库，如`java.util.concurrent`包中的工具。
  



* 线程的同步

  * 同步代码块：synchronized(同步监视器){//需要被同步的代码}

  * 同步方法：synchronized放在方法声明中，关于同步方法的总结：

    * 同步方法仍然涉及到同步监视器，只是不需要我们显式的声明。
    * 非静态的同步方法，同步监视器是：this
    * 静态的同步方法，同步监视器是：当前类本身

    ```java
    //死锁
    public class ThreadTest {
    
        public static void main(String[] args) {
    
            StringBuffer s1 = new StringBuffer();
            StringBuffer s2 = new StringBuffer();
    
    
            new Thread(){
                @Override
                public void run() {
    
                    synchronized (s1){
    
                        s1.append("a");
                        s2.append("1");
    
                        try {
                            Thread.sleep(100);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
    
                        synchronized (s2){
                            s1.append("b");
                            s2.append("2");
    
                            System.out.println(s1);
                            System.out.println(s2);
                        }
                    }
                }
            }.start();
    
            new Thread(new Runnable() {
                @Override
                public void run() {
                    synchronized (s2){
    
                        s1.append("c");
                        s2.append("3");
    
                        try {
                            Thread.sleep(100);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        synchronized (s1){
                            s1.append("d");
                            s2.append("4");
    
                            System.out.println(s1);
                            System.out.println(s2);
                        }
                    }
                }
            }).start();
        }
    }
    ```

  * Lock锁（可重入锁）  --- JDK5.0新增

    * ```java
      class A{
      	private final ReentrantLock lock = new ReentrantLock();
      	public void m(){
      		lock.lock();
      		try{
      		//保证线程安全的代码
      		}finally{
      			lock.unlock();
      		}
      	}
      }
      ```

  * 面试题：synchronized 与 Lock的异同？

    * 相同：二者都可以解决线程安全问题

    * 不同：synchronized机制在执行完相应的同步代码以后，自动的释放同步监视器；Lock需要手动的启动同步（lock()），同时结束同步也需要手动的实现（unlock()）

  * 优先使用顺序：Lock，同步代码块（已经进入了方法体，分配了相应资源），同步方法（在方法体之外）

    1. 可重入锁提供中断处理的能力

       ```java
       import java.util.concurrent.locks.ReentrantLock;  
         
       public class LockInterruptiblyExample {  
           private final ReentrantLock lock = new ReentrantLock();  
         
           public void someMethod() {  
               try {  
                   // 尝试获取锁，允许在等待过程中被中断  
                   lock.lockInterruptibly();  
         
                   // 临界区代码  
                   System.out.println("Doing something with the lock...");  
         
                   // 模拟长时间运行的任务  
                   Thread.sleep(5000);  
         
               } catch (InterruptedException e) {  
                   // 线程在等待锁时被中断  
                   System.out.println("Thread was interrupted while waiting for the lock.");  
                   // 注意：如果希望上层代码能够感知到中断，需要重新设置中断状态  
                   // Thread.currentThread().interrupt();  
         
                   // 清理资源或执行其他中断逻辑  
         
               } finally {  
                   // 总是释放锁  
                   if (lock.isHeldByCurrentThread()) {  
                       lock.unlock();  
                   }  
               }  
           }  
        
       // someMethod() 方法尝试获取 ReentrantLock。如果锁不可用，它将等待，并且这个等待过程是可以被中断的。在 main() 方法中，我们启动了一个线程去执行 someMethod() 方法，然后让主线程等待一段时间并中断那个线程。当线程在等待锁时被中断时，它会捕获 InterruptedException 并输出一条消息。注意，在实际应用中，你可能希望在捕获到 InterruptedException 后重新设置中断状态，以便上层代码能够感知到中断。  
           public static void main(String[] args) throws InterruptedException {  
               LockInterruptiblyExample example = new LockInterruptiblyExample();  
         
               // 启动一个线程去执行someMethod方法  
               Thread thread = new Thread(() -> {  
                   example.someMethod();  
               });  
         
               thread.start();  
         
               // 让主线程等待一段时间，然后中断等待锁的线程  
               Thread.sleep(1000);  
               thread.interrupt();  
         
               // 主线程等待子线程结束（注意：这里只是演示，实际上子线程可能在捕获到InterruptedException后已经退出了）  
               thread.join();  
           }  
       }
       ```

    2. 可重入锁提供锁申请等待限时

       `tryLock()`方法还提供了几个重载版本，允许你指定一个等待时间（以超时形式），例如`tryLock(long timeout, TimeUnit unit)`，该方法会尝试获取锁，但是如果在指定的超时时间内没有成功获取锁，那么它会返回`false`。

       ```java
       import java.util.concurrent.locks.ReentrantLock;  
         
       public class TryLockExample {  
           private final ReentrantLock lock = new ReentrantLock();  
         
           public void someMethod() {  
               // 尝试获取锁，不等待  
               if (lock.tryLock()) {  
                   try {  
                       // 临界区代码  
                       System.out.println("Lock acquired. Doing something...");  
         
                       // 模拟长时间运行的任务  
                       // ...  
         
                   } finally {  
                       // 释放锁  
                       lock.unlock();  
                   }  
               } else {  
                   // 未能获取到锁  
                   System.out.println("Failed to acquire lock without waiting.");  
                   // 可以在这里执行其他逻辑，或者选择等待  
               }  
           }  
         
           public static void main(String[] args) {  
               TryLockExample example = new TryLockExample();  
         
               // 假设有两个线程同时调用someMethod()  
               // Thread t1 = new Thread(() -> example.someMethod());  
               // Thread t2 = new Thread(() -> example.someMethod());  
               // t1.start();  
               // t2.start();  
         
               // 在这个简化的示例中，我们仅在一个线程中调用someMethod()  
               example.someMethod();  
           }  
       }
       ```

    3. 公平锁：private final ReentrantLock lock = new ReentrantLock(true);  

  * 在Java中，`ReentrantLock`类除了提供基本的锁机制外，还提供了一个与锁相关联的`Condition`接口的实现，用于支持线程间的等待/通知机制。与内置锁（synchronized关键字）中的`wait()`和`notify()`/`notifyAll()`方法相比，`ReentrantLock`的`Condition`提供了更强大且更灵活的等待/通知功能。

    `Condition`接口的主要方法包括：

    1. `await()`：使当前线程等待，直到其他线程调用该条件的`signal()`或`signalAll()`方法。当前线程必须持有与`Condition`关联的锁。

    2. `await(long time, TimeUnit unit)`：使当前线程等待，直到其他线程调用该条件的`signal()`或`signalAll()`方法，或者超过了指定的等待时间。当前线程必须持有与`Condition`关联的锁。

    3. `signal()`：唤醒在此条件上等待的一个线程（如果有的话）。选择哪个线程是任意的。

    4. `signalAll()`：唤醒在此条件上等待的所有线程。

    使用`ReentrantLock`和`Condition`的一个典型场景是生产者/消费者问题。以下是一个简单的示例，展示了如何使用`ReentrantLock`和`Condition`来实现生产者/消费者模式：

    ```java
    import java.util.LinkedList;
    import java.util.Queue;
    import java.util.concurrent.locks.Condition;
    import java.util.concurrent.locks.ReentrantLock;
    
    public class ProducerConsumerExample {
    
        private final Queue<Integer> queue = new LinkedList<>();
        private final int capacity;
    
        private final ReentrantLock lock = new ReentrantLock();
        private final Condition notFull = lock.newCondition();
        private final Condition notEmpty = lock.newCondition();
    
        public ProducerConsumerExample(int capacity) {
            this.capacity = capacity;
        }
    
        public void produce(int item) throws InterruptedException {
            lock.lock();
            try {
                while (queue.size() == capacity) {
                    // 队列已满，等待
                    notFull.await();
                }
                // 生产数据
                queue.add(item);
                System.out.println("Produced: " + item);
                // 通知等待的消费者
                notEmpty.signal();
            } finally {
                lock.unlock();
            }
        }
    
        public int consume() throws InterruptedException {
            lock.lock();
            try {
                while (queue.isEmpty()) {
                    // 队列为空，等待
                    notEmpty.await();
                }
                // 消费数据
                int item = queue.remove();
                System.out.println("Consumed: " + item);
                // 通知等待的生产者
                notFull.signal();
                return item;
            } finally {
                lock.unlock();
            }
        }
    
        // ... 其他方法和主函数等
    }
    ```

    在这个例子中，我们创建了一个`ProducerConsumerExample`类，该类使用`ReentrantLock`和两个`Condition`对象（`notFull`和`notEmpty`）来控制生产者和消费者线程之间的同步。生产者线程在队列满时等待`notFull`条件，并在添加数据后通知`notEmpty`条件。消费者线程在队列空时等待`notEmpty`条件，并在移除数据后通知`notFull`条件。

    这种基于`ReentrantLock`和`Condition`的同步机制比使用`synchronized`和`wait()`/`notify()`/`notifyAll()`更加灵活和高效，因为它允许你在一个锁上定义多个等待/通知条件。

  * Java中的信号量（`Semaphore`）是一个基于计数的信号量，用于控制对多个共享资源的并发访问。它是`java.util.concurrent.Semaphore`类的一个实例，通常用于限制可以并发访问某些资源（物理或逻辑）的线程数。

    `Semaphore`内部维护了一个计数器，该计数器表示当前可用的许可数量。每个许可代表一个资源，当线程需要访问资源时，它会尝试获取一个许可。如果计数器的值大于0，则许可被成功获取，计数器的值减1；如果计数器的值为0，则线程必须等待，直到有可用的许可。

    一旦线程完成了对资源的访问，它应该通过调用`Semaphore`的`release()`方法来释放许可，使计数器的值加1。这样，其他等待的线程就可以获取许可并访问资源了。

    以下是`Semaphore`的一些主要用法和特性：

    1. **限制并发访问**：通过限制同时访问资源的线程数，可以避免系统过载，确保资源的稳定性和性能。

    2. **非阻塞和阻塞获取**：`Semaphore`提供了阻塞和非阻塞的方法来获取许可。例如，`acquire()`方法会阻塞直到获取到许可，而`tryAcquire()`方法会立即返回，表示是否成功获取到许可。

    3. **公平和非公平模式**：`Semaphore`可以配置为公平或非公平模式。在公平模式下，等待时间最长的线程会优先获取许可；而在非公平模式下，等待的线程会按照它们到达的顺序（或者由JVM调度器的顺序）来竞争许可。

    4. **释放许可**：与获取许可相对应，线程在完成对资源的访问后，应该调用`release()`方法来释放许可，以便其他线程可以访问。

    5. **结合其他同步机制**：`Semaphore`可以与其他同步机制（如锁、条件变量等）一起使用，以提供更复杂的并发控制。

    下面是一个简单的示例，演示了如何使用`Semaphore`来限制对共享资源的并发访问：


    ```java
    import java.util.concurrent.Semaphore;
    
    public class SemaphoreExample {
        private final Semaphore semaphore;
        private final int maxConcurrent = 3; // 最多允许3个线程同时访问
    
        public SemaphoreExample() {
            this.semaphore = new Semaphore(maxConcurrent);
        }
    
        public void accessResource() throws InterruptedException {
            // 获取许可
            semaphore.acquire();
            try {
                // 访问共享资源的代码
                System.out.println(Thread.currentThread().getName() + " is accessing the resource.");
                // 模拟资源访问时间
                Thread.sleep(1000);
            } finally {
                // 释放许可
                semaphore.release();
            }
        }
    
        public static void main(String[] args) {
            SemaphoreExample example = new SemaphoreExample();
            for (int i = 0; i < 5; i++) {
                new Thread(() -> {
                    try {
                        example.accessResource();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }, "Thread-" + (i + 1)).start();
            }
        }
    }
    ```
    在上面的示例中，我们创建了一个`Semaphore`实例，其计数器初始化为3。然后，我们创建了5个线程来模拟对共享资源的访问。由于`Semaphore`的限制，最多只有3个线程可以同时访问资源。其他线程必须等待直到有线程释放了许可。

  * 在Java中，你可以使用`ReentrantReadWriteLock`类来实现读写锁。这个类提供了读锁和写锁的分离，允许多个读者线程同时访问共享资源，但只允许一个写者线程独占访问。

    ```java
    import java.util.concurrent.locks.ReentrantReadWriteLock;
    
    public class SharedData {
        private final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
        private int data;
    
        // 读取数据
        public int readData() {
            rwl.readLock().lock(); // 获取读锁
            try {
                // 这里读取共享数据
                return data;
            } finally {
                rwl.readLock().unlock(); // 释放读锁
            }
        }
    
        // 修改数据
        public void writeData(int newData) {
            rwl.writeLock().lock(); // 获取写锁
            try {
                // 这里修改共享数据
                data = newData;
            } finally {
                rwl.writeLock().unlock(); // 释放写锁
            }
        }
    }
    
    public class ReadWriteLockExample {
        public static void main(String[] args) {
            SharedData sharedData = new SharedData();
    
            // 模拟多个线程读取数据
            // ...
    
            // 模拟一个线程写入数据
            new Thread(() -> {
                sharedData.writeData(42);
                System.out.println("Data written: " + sharedData.readData());
            }).start();
        }
    }
    ```

    在上面的示例中，`SharedData`类有一个整数类型的共享数据`data`。我们使用`ReentrantReadWriteLock`来保护这个数据，确保在读取数据时允许多个线程并发访问，而在写入数据时只允许一个线程独占访问。

    在`readData`方法中，我们使用`rwl.readLock().lock()`来获取读锁，并在`finally`块中确保释放读锁。在`writeData`方法中，我们使用`rwl.writeLock().lock()`来获取写锁，并在`finally`块中确保释放写锁。

    请注意，`try-finally`块的使用是非常重要的，因为它可以确保即使代码块中出现异常，锁也能被正确地释放。如果忘记释放锁，可能会导致死锁或其他并发问题。

  * `CountDownLatch` 是 Java 并发工具包 `java.util.concurrent` 中的一个类，它允许一个或多个线程等待一组其他线程完成操作。`CountDownLatch` 的典型用途是控制并发线程的同步，使得一组线程能够等待直到其他线程完成一组操作。

    `CountDownLatch` 的工作原理是，初始化时设定一个计数器（count），该计数器只能被减少（通过调用 `countDown()` 方法），当计数器值达到零时，等待的线程（通过调用 `await()` 方法的线程）会被释放，继续执行。

    ```java
    import java.util.concurrent.CountDownLatch;
    
    public class CountDownLatchExample {
    
        public static void main(String[] args) throws InterruptedException {
            int threadCount = 5; // 假设有5个线程需要等待
            CountDownLatch latch = new CountDownLatch(threadCount);
    
            // 启动5个线程
            for (int i = 0; i < threadCount; i++) {
                new Thread(() -> {
                    System.out.println("线程 " + Thread.currentThread().getId() + " 正在执行...");
    
                    // 模拟一些工作
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
    
                    // 任务完成，计数器减一
                    latch.countDown();
                    System.out.println("线程 " + Thread.currentThread().getId() + " 任务完成，计数器减少至 " + (threadCount - latch.getCount()));
                }).start();
            }
    
            // 主线程等待其他线程完成
            latch.await();
            System.out.println("所有线程任务完成，继续主线程...");
        }
    }
    ```

    在这个示例中，我们创建了5个线程，并使用 `CountDownLatch` 来等待它们完成。每个线程在其任务完成后调用 `latch.countDown()`，而主线程则调用 `latch.await()` 来等待所有线程完成。当所有线程都调用 `countDown()` 方法，且计数器的值达到零时，主线程将继续执行。

  * `CyclicBarrier` 是 Java 并发工具包 `java.util.concurrent` 中的一个类，它允许一组线程互相等待，直到所有线程都到达某个公共屏障点（barrier point）。当所有线程都到达屏障点时，它们才被允许继续执行。`CyclicBarrier` 的名字中的 "Cyclic" 指的是它可以被重复使用，一旦所有线程都通过了屏障点，屏障会被重置，从而等待下一轮线程的到达。

    `CyclicBarrier` 适用于需要让一组线程等待彼此都完成某个操作（如初始化、数据加载等）的场景。

    ```java
    import java.util.concurrent.BrokenBarrierException;
    import java.util.concurrent.CyclicBarrier;
    
    public class CyclicBarrierExample {
    
        public static void main(String[] args) {
            int partySize = 5; // 假设有5个线程参与
            CyclicBarrier barrier = new CyclicBarrier(partySize, () -> {
                System.out.println("所有线程都已到达屏障点，开始执行屏障任务...");
                // 在这里可以放置一些需要在所有线程到达后执行的代码
            });
    
            for (int i = 0; i < partySize; i++) {
                new Thread(() -> {
                    System.out.println("线程 " + Thread.currentThread().getId() + " 正在准备...");
    
                    // 模拟一些准备工作
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
    
                    try {
                        // 等待其他线程到达屏障点
                        barrier.await();
                        System.out.println("线程 " + Thread.currentThread().getId() + " 已通过屏障点，继续执行...");
                    } catch (InterruptedException | BrokenBarrierException e) {
                        e.printStackTrace();
                    }
                }).start();
            }
        }
    }
    ```

    在这个示例中，我们创建了5个线程，并使用 `CyclicBarrier` 来让它们等待彼此。当所有线程都到达屏障点时，指定的屏障任务会执行一次，然后所有线程被释放并继续执行后续代码。

    `BrokenBarrierException` 异常是 Java 中与 `CyclicBarrier` 类相关的一种运行时异常。当某个线程试图等待处于断开状态的 `CyclicBarrier` 时，或者 `CyclicBarrier` 进入断开状态而线程处于等待状态时，就会抛出此异常。

    这种异常通常发生在使用 `CyclicBarrier` 的多线程程序中，可能的原因包括：

    1. **线程中断**：等待在 `CyclicBarrier` 上的线程被中断。
    2. **超时等待**：在指定的等待时间内，有线程未能到达屏障点。
    3. **屏障重置**：在等待过程中，`CyclicBarrier` 的 `reset()` 方法被调用，导致屏障提前损坏。
    4. **屏障操作异常**：执行屏障点动作（即传递给 `CyclicBarrier` 构造函数的 `Runnable` 任务）时出现异常。

    为了避免抛出 `BrokenBarrierException` 异常，需要在使用 `CyclicBarrier` 及其相关方法时特别注意以下几点：

    1. 确保所有线程都能在预期的时间内到达屏障点。
    2. 避免在不需要的时候调用 `CyclicBarrier` 的 `reset()` 方法。
    3. 如果在屏障点动作中执行了可能抛出异常的操作，请确保妥善处理这些异常，避免它们影响 `CyclicBarrier` 的正常状态。

    当捕获到 `BrokenBarrierException` 异常时，通常需要根据具体情况采取相应的恢复措施，例如重新初始化 `CyclicBarrier`、重新调度线程等。

  * `LockSupport` 是 Java 并发包 `java.util.concurrent.locks` 中的一个工具类，它提供了一种在线程之间进行基本的线程阻塞和解除阻塞的方法。与 `wait()` 和 `notify()`/`notifyAll()` 机制不同，`LockSupport` 不依赖于对象监视器（monitor）或内部锁，因此它可以用于控制那些没有获得任何锁，或者不想与锁交互的线程。

    `LockSupport` 主要提供了两个方法：

    1. `park()`：阻塞当前线程，除非或直到它的许可（permit）可用。
    2. `unpark(Thread thread)`：为指定的线程提供许可，使其可以离开 `park()` 状态。

    注意，`unpark(Thread thread)` 方法是“先使用后支付”的模型。这意味着，如果线程还没有被 `park()`，调用 `unpark(Thread thread)` 会使该线程获得一个许可，但是直到该线程调用 `park()` 方法时，它才会真正使用（消耗）这个许可并阻塞。如果在线程调用 `park()` 之前多次调用 `unpark(Thread thread)`，那么该线程可以多次从 `park()` 状态中解除阻塞，而不需要再次调用 `unpark(Thread thread)`。


    ```java
    import java.util.concurrent.locks.LockSupport;
    
    public class LockSupportExample {
    
        public static void main(String[] args) {
            Thread thread = new Thread(() -> {
                System.out.println("线程开始...");
                // 当前线程进入阻塞状态
                LockSupport.park();
                System.out.println("线程被唤醒...");
            });
    
            thread.start();
    
            // 等待一段时间，确保线程已经进入 park 状态
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
    
            // 唤醒线程
            LockSupport.unpark(thread);
        }
    }
    ```
    在这个示例中，我们创建了一个新线程，该线程在启动后会打印一条消息，然后调用 `LockSupport.park()` 进入阻塞状态。主线程在启动新线程后休眠一段时间（确保新线程已经进入 `park` 状态），然后调用 `LockSupport.unpark(thread)` 唤醒新线程。新线程在被唤醒后会继续执行并打印另一条消息。
    
    需要注意的是，虽然 `LockSupport` 提供了线程阻塞和解除阻塞的功能，但它并不提供任何形式的线程同步机制。因此，在使用 `LockSupport` 时，你需要自己确保线程之间的同步和协作。

  * 在Google的Guava库中，`RateLimiter`是一个用于控制对资源的访问速率的工具类。它可以帮助你实现令牌桶（Token Bucket）算法，该算法允许突发流量（burst）并平滑地限制长期的平均速率。

    以下是如何使用Guava的`RateLimiter`的简单示例：

    1. 首先，确保你的项目中包含了Guava库。如果你使用Maven，可以在`pom.xml`中添加以下依赖：


    ```xml
    <dependency>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
        <version>30.1-jre</version> <!-- 请检查并使用最新版本 -->
    </dependency>
    ```
    2. 使用`RateLimiter`来限制访问速率：


    ```java
    import com.google.common.util.concurrent.RateLimiter;
    
    public class RateLimiterExample {
    
        public static void main(String[] args) {
            // 创建一个RateLimiter，每秒生成1个令牌（即限制速率为每秒1个请求）
            RateLimiter rateLimiter = RateLimiter.create(1.0);
    
            // 模拟10个并发用户尝试访问资源
            for (int i = 0; i < 10; i++) {
                new Thread(() -> {
                    while (true) { // 无限循环模拟持续请求
                        // 尝试获取一个令牌，如果当前没有可用令牌，则会阻塞直到有令牌可用
                        double waitTime = rateLimiter.acquire();
    
                        // 打印等待时间（如果等待时间为0，表示没有等待）
                        System.out.println(Thread.currentThread().getName() + " 获取令牌，等待时间：" + waitTime + "秒");
    
                        // 模拟访问资源（这里只是简单地打印一条消息）
                        System.out.println(Thread.currentThread().getName() + " 访问资源");
    
                        // 为了演示，休眠一段时间再发起下一次请求
                        try {
                            Thread.sleep(500);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }).start();
            }
        }
    }
    ```
    注意：
    
    * `RateLimiter.create(1.0)` 创建了一个速率为每秒1个令牌的`RateLimiter`。你可以根据需要调整这个速率。
    * `rateLimiter.acquire()` 尝试获取一个令牌。如果没有可用令牌，该方法会阻塞直到有令牌可用。它返回等待时间（以秒为单位），但通常我们可能不需要这个值。
    * 在实际场景中，你可能不会在无限循环中调用`rateLimiter.acquire()`。上面的示例只是为了演示其工作原理。
    * 如果你不想在没有可用令牌时阻塞，可以使用`rateLimiter.tryAcquire()`方法，它会立即返回一个布尔值来表示是否成功获取了令牌。
    * 你还可以使用`rateLimiter.acquire(long permits)`或`rateLimiter.tryAcquire(long permits, long timeout, TimeUnit unit)`方法来尝试获取多个令牌。



  * 使用线程池

    `Executors` 是 Java 中的一个实用工具类，位于 `java.util.concurrent` 包中。它提供了创建和管理线程池的静态工厂方法，使得线程池的创建和配置变得简单且方便。使用 `Executors` 可以隐藏线程池实现的复杂性，并帮助开发者更加高效地使用和管理线程。

    `Executors` 类提供了以下几种主要的线程池类型：

    1. **FixedThreadPool**：
       - `newFixedThreadPool(int nThreads)`：创建一个固定大小的线程池，其中包含指定数量的线程。线程数量是固定的，不会自动扩展。适用于执行固定数量的长期任务。

         ```java
         import java.util.concurrent.ExecutorService;  
         import java.util.concurrent.Executors;  
           
         public class ExecutorsExample {  
             public static void main(String[] args) {  
                 // 创建一个固定大小的线程池，包含3个线程  
                 ExecutorService executor = Executors.newFixedThreadPool(3);  
                   
                 // 提交任务到线程池...  
                 // executor.submit(...);  
                   
                 // 关闭线程池  
                 executor.shutdown();  
             }  
         }
         ```
       
    2. **CachedThreadPool**：
       - `newCachedThreadPool()`：创建一个可缓存的线程池。这个线程池的线程数量可以根据需要自动扩展，如果有可用的空闲线程，就会重用它们；如果没有可用的线程，就会创建一个新线程。适用于执行大量的短期异步任务。

         ```java
         ExecutorService executor = Executors.newCachedThreadPool();  
         // 提交任务到线程池...  
         // executor.submit(...);  
           
         // 关闭线程池  
         executor.shutdown();
         ```
       
    3. **SingleThreadExecutor**：
       - `newSingleThreadExecutor()`：创建一个单线程线程池。线程池中只包含一个线程，用于串行执行任务。适用于需要按顺序执行任务的场景。

         ```java
         ExecutorService executor = Executors.newSingleThreadExecutor();  
         // 提交任务到线程池...  
         // executor.submit(...);  
           
         // 关闭线程池  
         executor.shutdown();
         ```
       
    4. **ScheduledThreadPool**：
       - `newScheduledThreadPool(int corePoolSize)`：创建一个数量固定的线程池，支持执行定时性或周期性任务。

         ```java
         import java.util.concurrent.ScheduledExecutorService;  
         import java.util.concurrent.Executors;  
         import java.util.concurrent.TimeUnit;  
           
         public class ScheduledExecutorsExample {  
             public static void main(String[] args) {  
                 // 创建一个支持定时任务的线程池  
                 ScheduledExecutorService executor = Executors.newScheduledThreadPool(5);  
                   
                 // 提交定时任务到线程池...  
                 // executor.schedule(...);  
                 // executor.scheduleAtFixedRate(...);  
                 // executor.scheduleWithFixedDelay(...);  
                   
                 // 关闭线程池  
                 executor.shutdown();  
             }  
         }
         ```
       
    5. **SingleThreadScheduledExecutor**：
       - 这是一个特殊的 `ScheduledThreadPool`，其中线程池只包含一个线程，用于执行定时性或周期性任务。

    6. **WorkStealingPool**（Java 8 新增）：
       - `newWorkStealingPool(int parallelism)`：创建一个工作窃取线程池。这种线程池会并行处理任务，但不能保证执行顺序。如果不设置任何参数，则以当前机器处理器个数作为线程个数。

    使用 `Executors` 创建的线程池都实现了 `ExecutorService` 接口，因此可以使用 `ExecutorService` 的方法来管理线程池中的线程和任务。例如，可以使用 `submit()` 方法提交任务，使用 `shutdown()` 和 `shutdownNow()` 方法关闭线程池，以及使用 `awaitTermination()` 方法等待线程池关闭等。

    提交任务到线程池时，可以使用`execute()`方法提交不需要返回值的任务，或者使用`submit()`方法提交需要返回值的任务。`submit()`方法会返回一个`Future`对象，你可以通过这个对象来检查任务是否完成、等待任务完成并获取结果。

    最后，当不再需要线程池时，应该调用`shutdown()`方法来关闭它。这将启动线程池的关闭序列，线程池将不再接受新的任务，但是已经提交的任务将继续执行直到完成。如果需要立即停止所有正在执行的任务并关闭线程池，可以调用`shutdownNow()`方法。但是请注意，这可能会导致正在执行的任务抛出异常。

    * `Executors` 本身并不直接实现线程池，而是利用 `ThreadPoolExecutor` 类来创建和管理线程池。`ThreadPoolExecutor` 是 Java 线程池的核心实现类，它包含了一个线程池所需的所有核心功能。以下是 `ThreadPoolExecutor` 的一些关键组件和内部实现：

      1. **线程池状态（Pool State）**：
         - `ThreadPoolExecutor` 使用一个整数变量 `ctl` 来同时表示线程池状态和线程数量。这个变量通过位运算来同时存储两种信息。
         - 状态包括：RUNNING、SHUTDOWN、STOP、TIDYING、TERMINATED。这些状态决定了线程池是否可以接受新任务、是否可以处理队列中的任务等。

      2. **工作线程（Worker Threads）**：
         - 线程池中的每个工作线程都是由 `Worker` 类实现的。`Worker` 实现了 `Runnable` 接口，因此可以作为一个线程来运行。
         - `Worker` 在其 `run` 方法中循环从任务队列中获取任务并执行。如果队列为空，则尝试获取并终止一个空闲的线程；如果线程池已关闭，则退出循环。

      3. **任务队列（Work Queue）**：
         - 线程池使用一个阻塞队列来保存待执行的任务。当线程池中的线程数量达到核心线程数时，新提交的任务会被放入任务队列中等待执行。
         - 任务队列的类型可以根据需要进行配置，例如使用 `LinkedBlockingQueue`（无界队列）或 `ArrayBlockingQueue`（有界队列）等。

      4. **拒绝策略（Rejection Policy）**：
         - 当线程池无法处理新任务时（例如，线程池已关闭、线程池已满且队列已满），它会使用一个拒绝策略来处理这个任务。
         - Java 提供了四种内置的拒绝策略：`AbortPolicy`（直接抛出异常）、`CallerRunsPolicy`（调用者运行任务）、`DiscardPolicy`（丢弃任务）和 `DiscardOldestPolicy`（丢弃队列中最老的任务）。

      5. **线程工厂（Thread Factory）**：
         - 线程池使用一个 `ThreadFactory` 来创建新的工作线程。通过提供自定义的线程工厂，可以定制线程的创建过程，例如设置线程名称、优先级等。

      `Executors` 类提供了多种静态方法来创建不同配置的 `ThreadPoolExecutor` 实例。这些方法简化了线程池的创建过程，但同时也隐藏了 `ThreadPoolExecutor` 的很多配置细节。因此，在某些需要更精细控制线程池行为的场景中，建议直接使用 `ThreadPoolExecutor` 类进行创建和配置。

      使用 `ThreadPoolExecutor` 类来创建和配置线程池，你需要指定几个关键的参数。以下是一个如何创建 `ThreadPoolExecutor` 的示例，并解释了每个参数的含义：

      ```java
      import java.util.concurrent.BlockingQueue;
      import java.util.concurrent.LinkedBlockingQueue;
      import java.util.concurrent.ThreadPoolExecutor;
      import java.util.concurrent.TimeUnit;
      import java.util.concurrent.RejectedExecutionHandler;
      import java.util.concurrent.ThreadFactory;
      
      public class ThreadPoolExample {
      
          public static void main(String[] args) {
              // 线程池参数配置
              int corePoolSize = 5; // 核心线程数
              int maximumPoolSize = 10; // 最大线程数
              long keepAliveTime = 60L; // 空闲线程保持存活的时间
              TimeUnit unit = TimeUnit.SECONDS; // 时间单位
              BlockingQueue<Runnable> workQueue = new LinkedBlockingQueue<>(100); // 工作队列
              ThreadFactory threadFactory = Executors.defaultThreadFactory(); // 线程工厂
              RejectedExecutionHandler handler = new ThreadPoolExecutor.AbortPolicy(); // 拒绝策略
      
              // 创建 ThreadPoolExecutor 实例
              ThreadPoolExecutor executor = new ThreadPoolExecutor(
                      corePoolSize,
                      maximumPoolSize,
                      keepAliveTime,
                      unit,
                      workQueue,
                      threadFactory,
                      handler
              );
      
              // 提交任务到线程池
              for (int i = 0; i < 15; i++) {
                  Runnable worker = () -> {
                      System.out.println(Thread.currentThread().getName() + " is working.");
                      // 模拟任务执行
                      try {
                          Thread.sleep(1000);
                      } catch (InterruptedException e) {
                          Thread.currentThread().interrupt();
                      }
                  };
                  executor.execute(worker);
              }
      
              // 关闭线程池
              executor.shutdown(); // 等待任务执行完毕
              // 如果需要立即关闭，可以使用 executor.shutdownNow();
      
              // 注意：通常，你应该在应用程序的某个合适的地方调用 shutdown 或 shutdownNow 方法来关闭线程池。
          }
      }
      ```

      对于返回结果的任务：

      ```java
      Future<String> future = executor.submit(() -> {  
          // 执行任务并返回结果  
          return "Task result";  
      });  
        
      // 获取任务结果  
      try {  
          String result = future.get(); // 这将阻塞直到任务完成  
          System.out.println("Task result: " + result);  
      } catch (Exception e) {  
          e.printStackTrace();  
      }
      ```

      在这个示例中，我们配置了以下参数：

      - `corePoolSize`（核心线程数）：线程池中保持存活的最小线程数。即使这些线程处于空闲状态，它们也不会被销毁，除非设置了 `allowCoreThreadTimeOut` 为 `true`。

      - `maximumPoolSize`（最大线程数）：线程池中允许的最大线程数。当工作队列已满，并且当前线程数小于最大线程数时，会创建新的线程来处理任务。

      - `keepAliveTime`（空闲线程保持存活的时间）：当线程数大于核心线程数时，这是多余的空闲线程在终止前等待新任务的最长时间。

      - `unit`（时间单位）：`keepAliveTime` 参数的时间单位。

      - `workQueue`（工作队列）：用于保存等待执行的任务的阻塞队列。

      - `threadFactory`（线程工厂）：用于创建新线程的工厂。

        在Java中，你可以通过实现`ThreadFactory`接口来自定义线程工厂，该接口定义了一个方法`newThread(Runnable r)`，该方法用于创建新的线程。下面是一个如何自定义线程工厂的示例：

        ```java
        import java.util.concurrent.ThreadFactory;
        import java.util.concurrent.atomic.AtomicInteger;
        
        public class CustomThreadFactory implements ThreadFactory {
            private static final AtomicInteger poolNumber = new AtomicInteger(1);
            private final AtomicInteger threadNumber = new AtomicInteger(1);
            private final String namePrefix;
        
            public CustomThreadFactory(String poolName) {
                namePrefix = "pool-" + poolNumber.getAndIncrement() + "-thread-";
            }
        
            @Override
            public Thread newThread(Runnable r) {
                Thread thread = new Thread(r, namePrefix + threadNumber.getAndIncrement());
                // 设置线程为守护线程（如果需要的话）
                // thread.setDaemon(true);
                // 设置线程优先级（如果需要的话）
                // thread.setPriority(Thread.NORM_PRIORITY);
                return thread;
            }
        }
        ```

        在上面的示例中，我们创建了一个`CustomThreadFactory`类，它实现了`ThreadFactory`接口。我们使用了一个原子整数来生成唯一的线程编号，并将其附加到线程的名称上。

        然后，你可以在创建`ThreadPoolExecutor`时，将你的自定义线程工厂作为参数传递给它：

        ```java
        import java.util.concurrent.ThreadPoolExecutor;
        import java.util.concurrent.TimeUnit;
        import java.util.concurrent.BlockingQueue;
        import java.util.concurrent.LinkedBlockingQueue;
        
        public class ThreadPoolExample {
            public static void main(String[] args) {
                int corePoolSize = 5;
                int maximumPoolSize = 10;
                long keepAliveTime = 60L;
                TimeUnit unit = TimeUnit.SECONDS;
                BlockingQueue<Runnable> workQueue = new LinkedBlockingQueue<>(100);
                
                // 创建自定义线程工厂
                ThreadFactory threadFactory = new CustomThreadFactory("MyCustomPool");
                
                // 创建 ThreadPoolExecutor 实例
                ThreadPoolExecutor executor = new ThreadPoolExecutor(
                    corePoolSize,
                    maximumPoolSize,
                    keepAliveTime,
                    unit,
                    workQueue,
                    threadFactory
                    // 这里我们没有指定拒绝策略，所以使用默认的 ThreadPoolExecutor.AbortPolicy
                );
        
                // 提交任务到线程池（省略...）
        
                // 关闭线程池（省略...）
            }
        }
        ```

        现在，当你向线程池提交任务时，`ThreadPoolExecutor`会使用你的自定义线程工厂来创建新的线程。这些新线程的名称将以你指定的前缀和唯一的线程编号作为后缀。

        请注意，你还可以根据需要设置线程的其他属性，如守护线程状态或线程优先级，但这取决于你的具体需求。在上面的示例中，这些设置被注释掉了，但你可以根据需要取消注释并进行配置。

      - `handler`（拒绝策略）：当线程池无法处理新任务时所使用的策略。Java 提供了几种内建的拒绝策略，包括 `AbortPolicy`（直接抛出异常）、`CallerRunsPolicy`（调用者运行任务）、`DiscardPolicy`（丢弃任务）和 `DiscardOldestPolicy`（丢弃队列中最老的任务）。

      创建 `ThreadPoolExecutor` 实例后，你可以通过 `execute` 方法提交任务到线程池，或者通过 `submit` 方法提交返回 `Future` 的任务。当不再需要线程池时，应该调用 `shutdown` 或 `shutdownNow` 方法来关闭它。

    * 在 `ThreadPoolExecutor` 中，存在几个可以被用户重写的钩子方法（hook method），其中包括 `beforeExecute`、`afterExecute` 和 `terminated`。

      1. **beforeExecute**:
         - 这个方法在线程池中的线程开始执行一个任务之前被调用。
         - 它接收两个参数：一个 `Thread` 对象，表示执行任务的线程；一个 `Runnable` 对象，表示要执行的任务。
         - 这个方法可以在子类中重写，用于在任务执行前进行一些自定义操作，如日志记录、计时统计、更新线程本地变量等。
         - 默认情况下，这个方法不执行任何操作。
      2. **afterExecute**:
         - 这个方法在线程池中的线程完成一个任务（包括异常结束）之后被调用。
         - 它也接收两个参数：一个 `Runnable` 对象，表示已经执行完毕的任务；一个 `Throwable` 对象（可能为 `null`），表示任务执行过程中抛出的异常（如果没有异常则为 `null`）。
         - 这个方法可以在子类中重写，用于在任务执行后进行一些自定义操作，如清除线程本地变量、更新日志记录、收集统计信息等。
         - 默认情况下，这个方法也不执行任何操作。
      3. **terminated**:
         - 这个方法在 `shutdown` 或 `shutdownNow` 被调用，并且所有任务都已经完成（包括执行中的任务和队列中等待的任务）之后被调用。
         - 这个方法没有参数。
         - 可以在这个方法中进行一些清理工作，如关闭一些与线程池关联的资源等。
         - 注意，这个方法的调用发生在所有工作线程都终止之后，也就是说，它是线程池生命周期中的最后一个方法。

      这些钩子方法为用户提供了在线程池生命周期的关键阶段进行自定义操作的能力，使得 `ThreadPoolExecutor` 的使用更加灵活和强大。

      ```java
      import java.util.concurrent.BlockingQueue;  
      import java.util.concurrent.RejectedExecutionHandler;  
      import java.util.concurrent.ThreadPoolExecutor;  
      import java.util.concurrent.TimeUnit;  
        
      public class CustomThreadPoolExecutor extends ThreadPoolExecutor {  
        
          public CustomThreadPoolExecutor(int corePoolSize,  
                                          int maximumPoolSize,  
                                          long keepAliveTime,  
                                          TimeUnit unit,  
                                          BlockingQueue<Runnable> workQueue,  
                                          ThreadFactory threadFactory,  
                                          RejectedExecutionHandler handler) {  
              super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory, handler);  
          }  
        
          // 重写 beforeExecute 方法  
          @Override  
          protected void beforeExecute(Thread t, Runnable r) {  
              super.beforeExecute(t, r); // 可以选择调用父类的实现，或者省略  
              // 在这里添加你的自定义逻辑  
              System.out.println("Thread " + t.getName() + " is going to execute " + r.getClass().getSimpleName());  
          }  
        
          // 重写 afterExecute 方法  
          @Override  
          protected void afterExecute(Runnable r, Throwable t) {  
              super.afterExecute(r, t); // 可以选择调用父类的实现，或者省略  
              // 在这里添加你的自定义逻辑  
              if (t == null) {  
                  System.out.println("Task " + r.getClass().getSimpleName() + " completed successfully.");  
              } else {  
                  System.out.println("Task " + r.getClass().getSimpleName() + " threw an exception.");  
                  t.printStackTrace();  
              }  
          }  
        
          // 重写 terminated 方法  
          @Override  
          protected void terminated() {  
              super.terminated(); // 可以选择调用父类的实现，或者省略  
              // 在这里添加你的自定义逻辑  
              System.out.println("ThreadPoolExecutor is terminated.");  
              // 清理资源等  
          }  
      }
      ```

      在这个示例中，`CustomThreadPoolExecutor` 类继承了 `ThreadPoolExecutor`，并重写了 `beforeExecute`、`afterExecute` 和 `terminated` 方法。你可以在这些方法中添加你自己的逻辑，比如日志记录、性能监控、异常处理等。

      请注意，在重写这些方法时，你可以选择调用父类的实现（如示例中的 `super.beforeExecute(t, r)`），或者省略它，这取决于你的具体需求。在某些情况下，你可能只需要添加额外的逻辑，而不需要修改父类的默认行为。

      要使用这个自定义的线程池，你可以像使用普通的 `ThreadPoolExecutor` 一样创建它的实例，并提交任务到它。但是，你需要提供正确的构造函数参数来初始化线程池。

    * `ForkJoinPool` 是 Java 7 引入的一个特殊类型的线程池，专为执行分治算法（Divide-and-Conquer algorithms）而设计。它允许你将一个大的任务（任务通常被称为 `ForkJoinTask`）分解为更小的子任务，并将这些子任务分配给不同的线程来并行执行。一旦子任务完成，其结果会被合并回父任务，然后继续向上合并，直到所有任务都完成。

      ### 特性

      1. **工作窃取（Work-Stealing）**：当一个线程完成了其所有的任务，它会尝试从其他线程队列中“窃取”任务来执行，从而减少了线程间的空闲时间，提高了资源利用率。

      2. **适应性**：`ForkJoinPool` 的线程数量是动态的，可以随着任务数量的增加而增加，也可以随着任务数量的减少而减少。这种适应性有助于优化系统资源的使用。

      3. **递归任务**：非常适合执行递归分治算法，如快速排序、归并排序等。

      ### 使用方法

      1. **创建 `ForkJoinPool`**：你可以使用默认的 `ForkJoinPool`（即 `ForkJoinPool.commonPool()`），也可以创建一个具有特定并行级别的 `ForkJoinPool`。

      2. **定义 `ForkJoinTask`**：你需要扩展 `RecursiveAction`（对于没有返回值的任务）或 `RecursiveTask<V>`（对于有返回值的任务）来定义你的任务。

      3. **提交任务**：使用 `ForkJoinPool` 的 `invoke`、`submit` 或 `execute` 方法来提交你的 `ForkJoinTask`。

      ### 示例

      下面是一个简单的示例，演示了如何使用 `ForkJoinPool` 来计算从 1 到 N 的整数和：
      
      ```java
      import java.util.concurrent.ForkJoinPool;  
      import java.util.concurrent.RecursiveAction;  
        
      public class ForkJoinSum extends RecursiveAction {  
          private static final int THRESHOLD = 1000; // 设定阈值  
          private int[] array;  
          private int start, end;  
        
          public ForkJoinSum(int[] array, int start, int end) {  
              this.array = array;  
              this.start = start;  
              this.end = end;  
          }  
        
          @Override  
          protected void compute() {  
              if (end - start < THRESHOLD) {  
                  int sum = 0;  
                  for (int i = start; i < end; i++) {  
                      sum += array[i];  
                  }  
                  setRawResult(sum); // 设置任务的结果  
              } else {  
                  int split = (start + end) / 2;  
                  ForkJoinSum left = new ForkJoinSum(array, start, split);  
                  ForkJoinSum right = new ForkJoinSum(array, split, end);  
                  invokeAll(left, right); // 递归地分解任务  
                  int leftResult = left.getRawResult(); // 获取左子任务的结果  
                  int rightResult = right.getRawResult(); // 获取右子任务的结果  
                  setRawResult(leftResult + rightResult); // 合并结果  
              }  
          }  
        
          public static void main(String[] args) {  
              int[] array = ... // 初始化数组  
              ForkJoinPool pool = new ForkJoinPool(); // 创建ForkJoinPool  
              ForkJoinSum task = new ForkJoinSum(array, 0, array.length);  
              int result = pool.invoke(task); // 提交任务并获取结果  
              System.out.println("Sum: " + result);  
          }  
      }
      ```
    
    * Guava库对Java的并发API提供了许多有用的扩展和工具，包括线程池的使用。虽然Guava没有直接实现一个新的线程池类型，但它提供了一些实用类和方法，使线程池的使用更加简单和灵活。
    
      以下是Guava中与线程池相关的几个主要扩展：
    
      1. **`ListeningExecutorService` 和 `MoreExecutors`**：
         - `ListeningExecutorService` 是一个扩展了 `ExecutorService` 的接口，允许你为执行的任务添加监听器。这意味着当任务开始、完成、失败或取消时，你可以执行特定的操作。
         - `MoreExecutors` 类提供了一些静态方法来创建和配置 `ListeningExecutorService` 和其他有用的 `Executor` 实现。
    
      2. **`ThreadFactoryBuilder`**：
         - Guava的 `ThreadFactoryBuilder` 类允许你轻松地创建自定义的 `ThreadFactory`。这对于设置线程名称、守护线程状态、线程优先级等非常有用。
    
      3. **`ForwardingExecutorService`**：
         - 如果你想要包装或扩展一个现有的 `ExecutorService`，那么 `ForwardingExecutorService` 是一个很好的起点。它提供了所有方法的默认实现，你只需要覆盖你感兴趣的方法。
    
      4. **`Service` 和 `AbstractService`**：
         - Guava的 `Service` 接口和 `AbstractService` 类提供了一种更高级的方式来管理服务的生命周期（如启动、停止和故障转移）。虽然它们不直接涉及线程池，但它们经常与线程池一起使用，以控制后台任务的执行。
    
      5. **`Callable` 和 `Future` 的工具方法**：
         - Guava提供了一些实用的工具方法来处理 `Callable` 和 `Future`，例如 `Futures.transform()` 用于转换 `Future` 的结果，`Futures.allAsList()` 用于等待多个 `Future` 的完成等。
    
      6. **`RateLimiter`**：
         - 虽然不是直接关于线程池的，但Guava的 `RateLimiter` 类允许你控制代码段的执行速率。这对于限制线程池中的任务执行速率非常有用。
    
      7. **`Executors2`**：
         - 这是一个额外的工具类，提供了对 `ExecutorService` 的更多实用扩展，如 `Executors2.getUnchecked(Future<T>)`，它允许你将 `Future` 的异常作为运行时异常抛出。
    
      8. **`ListenableFuture` 和 `Futures`**：
         - Guava的 `ListenableFuture` 接口和 `Futures` 类提供了一种更强大和灵活的方式来处理异步计算的结果。你可以注册监听器来响应 `Future` 的完成，而无需显式轮询或阻塞。
    
      使用Guava的这些扩展和工具，你可以更轻松、更安全地管理和使用线程池。但是，请注意，它们并不替代Java的内置并发API；相反，它们是对其的补充和增强。



* Java的`Collections`类提供了一系列静态工厂方法，用于将非线程安全的集合（如`ArrayList`、`HashMap`等）转换为线程安全的集合。这些线程安全的集合通常是通过在原始集合的基础上添加同步（synchronization）来实现的。例如，`Collections.synchronizedList()`方法接受一个`List`对象作为参数，并返回一个线程安全的列表，该列表在修改操作（如`add`、`remove`等）时使用了内部锁来确保线程安全。类似地，`Collections.synchronizedMap()`方法也接受一个`Map`对象作为参数，并返回一个线程安全的映射。

  这些线程安全的集合在多个线程同时访问时，通过同步机制确保了对集合的修改操作的原子性。但是，需要注意的是，这些集合的迭代器（Iterator）并不是线程安全的。如果在迭代过程中有其他线程修改了集合，那么迭代器可能会抛出`ConcurrentModificationException`异常。另外，虽然`Collections`类提供的这些线程安全集合在某些情况下可以简化并发编程的复杂性，但它们并不是并发编程的唯一解决方案。对于更复杂的并发需求，可能需要使用更高级的并发容器，如`java.util.concurrent`包下的`ConcurrentHashMap`、`CopyOnWriteArrayList`等。这些容器提供了更高效的并发性能，并解决了`Collections`类提供的线程安全集合在迭代和并发修改方面的一些问题。

  JDK的并发容器是一组专为多线程环境设计的容器类，它们提供了线程安全的数据结构和操作，从而简化了并发编程的复杂性。这些容器类主要集中在`java.util.concurrent`包下，包括但不限于以下几种：

  1. **ConcurrentHashMap**：这是一个高效的并发HashMap，可以将其理解为一个线程安全的HashMap。它提供了高并发性能，同时保持了较低的锁竞争和较高的吞吐量。
  2. **CopyOnWriteArrayList**：这是一个线程安全的List实现，其特点是在修改操作时（如add、set等）会复制底层数组，因此在读操作时可以无需加锁，从而提供高效的读取性能。适用于读多写少的场景。
  3. **ConcurrentLinkedQueue**：这是一个基于单向链表实现的无界非阻塞线程安全队列。它使用CAS（Compare-and-Swap）自旋和volatile来保证线程安全，适用于高并发场景。
  4. **BlockingQueue**：这是一个支持阻塞操作的队列接口，常用于生产者-消费者模型中的数据共享通道。当队列为空时，消费者线程会阻塞等待；当队列满时，生产者线程会阻塞等待。
  5. **ConcurrentSkipListSet** 和 **ConcurrentSkipListMap**：它们利用跳表（SkipList）结构实现高效的并发Set和Map。跳表是一种随机化的数据结构，能够在O(log n)时间复杂度下进行查找、插入和删除操作。

  这些并发容器与普通容器（如ArrayList、HashMap等）相比，更注重在多线程环境下数据结构的操作性能，而不仅仅是线程安全。它们通过各种并发控制策略（如分段锁、CAS等）来减少锁竞争，提高并发性能。

  在选择合适的并发容器时，需要根据实际的使用场景进行考虑，如执行更多的读操作还是写操作、是否需要排序功能、是否考虑内存占用和扩展性等因素。

* Java的JMH（Java Microbenchmark Harness）是由OpenJDK团队开发的一款专业的基准测试工具，旨在提供一个可靠的测试框架，帮助Java开发者进行代码性能的评估和优化。JMH可以对代码进行微基准测试，以提供精确的性能数据，并帮助开发者发现潜在的性能问题。

  在Java中，使用JMH的方法主要包括一些特定的注解，如`@Benchmark`（用于标记测试方法）、`@State`（用于定义测试状态）、`@Setup`（用于执行初始化操作的方法）、`@TearDown`（用于执行清理操作的方法）等。这些注解和方法的结合，可以编写具有高度可靠性和准确性的微基准测试。

  要使用Java的JMH（Java Microbenchmark Harness）进行基准测试，你需要遵循以下步骤：

  ### 1. 添加JMH依赖

  首先，你需要将JMH的依赖添加到你的项目中。如果你使用Maven，可以在`pom.xml`文件中添加以下依赖：

  ```xml
  <dependencies>
      <!-- 添加JMH依赖 -->
      <dependency>
          <groupId>org.openjdk.jmh</groupId>
          <artifactId>jmh-core</artifactId>
          <version>你的JMH版本号</version>
      </dependency>
      <dependency>
          <groupId>org.openjdk.jmh</groupId>
          <artifactId>jmh-generator-annprocess</artifactId>
          <version>你的JMH版本号</version>
          <scope>provided</scope>
      </dependency>
  </dependencies>
  
  <!-- 添加Maven插件（可选，用于生成JMH测试代码） -->
  <build>
      <plugins>
          <plugin>
              <groupId>org.openjdk.jmh</groupId>
              <artifactId>jmh-maven-plugin</artifactId>
              <version>你的JMH版本号</version>
              <executions>
                  <execution>
                      <id>jmh-generate-sources</id>
                      <phase>generate-sources</phase>
                      <goals>
                          <goal>generate-sources</goal>
                      </goals>
                  </execution>
                  <execution>
                      <id>jmh-compile-sources</id>
                      <phase>compile</phase>
                      <goals>
                          <goal>compile</goal>
                      </goals>
                  </execution>
                  <execution>
                      <id>jmh-run-benchmarks</id>
                      <phase>test</phase>
                      <goals>
                          <goal>run</goal>
                      </goals>
                  </execution>
              </executions>
          </plugin>
      </plugins>
  </build>
  ```

  请确保将`你的JMH版本号`替换为最新的JMH版本号。

  ### 2. 编写基准测试

  接下来，你需要编写基准测试类。这个类通常包含一个或多个使用`@Benchmark`注解的方法，这些方法表示要测试的基准操作。例如：

  ```java
  import org.openjdk.jmh.annotations.*;
  
  @BenchmarkMode(Mode.Throughput) // 基准测试模式（吞吐量）
  @OutputTimeUnit(TimeUnit.SECONDS) // 输出时间单位（秒）
  @State(Scope.Thread) // 测试状态作用域（线程）
  public class MyBenchmark {
  
      @Benchmark
      public void testMethod() {
          // 这里放置要测试的代码
          // 例如：一个简单的循环或者方法调用
      }
  
      // 如果有需要，可以添加更多的基准测试方法
  }
  ```

  在这个例子中，`@BenchmarkMode`注解用于指定基准测试的模式（如吞吐量、平均时间等），`@OutputTimeUnit`注解用于指定输出时间单位，`@State`注解用于指定测试状态的作用域（在这个例子中，每个线程都有自己的测试状态）。

  ### 3. 运行基准测试

  如果你使用Maven，并且已经添加了Maven插件，那么你可以通过运行Maven的`test`阶段来执行基准测试：

  ```bash
  mvn clean test
  ```

  Maven插件会自动编译基准测试代码，并运行它们。测试结果将被输出到控制台和结果文件中。

  如果你没有使用Maven或者不想使用Maven插件，你也可以通过编写一个包含JMH主类的Java程序来运行基准测试。这个主类将使用JMH的API来执行基准测试。

  ### 4. 分析结果

  JMH将测试结果输出到控制台和结果文件中。你可以查看这些结果来了解你的代码在不同配置下的性能表现。JMH还提供了一些可视化工具，如JMH Visualizer，可以帮助你更直观地分析测试结果。

  请注意，基准测试是一个复杂的过程，需要仔细设计和执行才能获得准确和可靠的结果。你应该避免在基准测试中包含与测试目标无关的代码和依赖项，并尽可能减少测试过程中的噪声和干扰。



###### 2. 锁的优化

在Java中，锁是并发编程中不可或缺的一部分，用于同步访问共享资源，以防止数据不一致和其他并发问题。然而，不当地使用锁可能导致性能下降。以下是一些提高Java锁性能的建议：

1. **最小化锁的范围**：
   - 尽量减少需要同步的代码块的大小。将锁的范围限制在最小的临界区内，以减少线程间的竞争。

2. **使用读写锁（ReadWriteLock）**：
   - 当多个线程需要同时读取共享资源时，使用读写锁可以提高性能。读写锁允许多个线程同时读取，但在写入时只允许一个线程访问。

3. **避免嵌套锁**：
   - 嵌套锁可能导致死锁和性能下降。如果可能的话，避免在一个已经持有锁的线程中请求另一个锁。

4. **锁分段（Lock Striping）**：
   - 如果一个锁保护的资源可以被分割成多个独立的部分，那么可以考虑使用多个锁来保护这些部分。这样可以减少线程间的竞争。

5. **使用偏向锁（Biased Locking）**：
   - Java HotSpot VM的默认锁实现使用了偏向锁来优化无竞争的情况。在大多数情况下，偏向锁可以提高性能，因为它避免了锁的获取和释放开销。

6. **使用轻量级锁（Lightweight Locking）**：
   - 当线程尝试获取一个已经被其他线程持有的锁时，JVM会首先尝试使用轻量级锁。轻量级锁是一种自旋锁，它会让线程在一个循环中等待锁的释放，而不是立即阻塞。这可以减少线程切换的开销。

7. **减少锁的粒度**：
   - 将大锁分解为多个小锁，这样可以减少线程间的竞争。但是，需要小心避免死锁和性能下降。

8. **使用条件变量（Condition）**：
   - 如果线程需要在某个条件成立时才能继续执行，那么可以使用条件变量来等待这个条件。条件变量允许线程在等待时释放锁，从而减少线程间的竞争。

9. **使用原子类（Atomic Classes）**：
   - 对于一些简单的同步需求，可以使用Java的原子类（如AtomicInteger、AtomicLong等）来替代显式的锁。原子类提供了高效的线程安全操作，并且避免了锁的开销。

10. **使用锁升级策略**：
    - 一些高级的并发库（如Netty、Disruptor等）使用了锁升级策略来优化性能。这些策略根据线程访问的模式动态地选择使用哪种锁（偏向锁、轻量级锁、重量级锁等），以达到最佳的性能。

11. **避免死锁和活锁**：
    - 通过合理的锁顺序、超时等待、检测死锁等技术来避免死锁和活锁的发生。这些问题会严重降低系统的性能。

12. **使用无锁数据结构**：
    - 对于一些特定的应用场景，可以考虑使用无锁数据结构（如Disruptor中的环形缓冲区）来替代传统的基于锁的数据结构。无锁数据结构可以在没有锁的情况下实现线程安全，从而避免了锁的开销。

13. **优化JVM参数**：
    - 根据应用程序的特点和硬件环境，调整JVM的参数（如堆大小、垃圾收集器类型等）可以进一步提高锁的性能。

14. **使用并发容器**：
    - Java并发包（java.util.concurrent）提供了一系列并发容器（如ConcurrentHashMap、CopyOnWriteArrayList等），这些容器内部已经实现了高效的并发控制，可以简化并发编程并提高性能。

* `ThreadLocal` 是 Java 提供的一个类，它提供了线程局部变量。这些变量与其他普通变量的区别在于，每一个访问这个变量的线程都有其自己独立初始化的变量副本。`ThreadLocal` 在多线程环境下提供了保持线程封闭数据（thread-confined data）的一种解决方案，即一个线程内部的数据对其他线程是不可见的。

  ### 主要用途

  1. **线程安全的数据存储**：每个线程都可以独立地改变自己的副本，而不会和其他线程的副本冲突。
  2. **避免传递参数**：在复杂的函数调用中，有时需要传递很多参数，如果参数中有很多是线程独有的，那么传递起来会非常麻烦，这时可以使用 `ThreadLocal`。
  3. **数据库连接、用户信息等线程私有资源**：在Web应用中，经常需要为每个用户保存一些信息，比如数据库连接、用户信息等，这些信息通常是线程私有的，使用 `ThreadLocal` 可以方便地管理这些资源。

  ### 使用方法

  1. **创建 ThreadLocal 实例**：
     通常使用匿名内部类的方式创建一个 `ThreadLocal` 的实例。

     ```java
     ThreadLocal<String> threadLocal = new ThreadLocal<>();
     ```

  2. **设置值**：
     通过 `set(T value)` 方法为当前线程设置值。

     ```java
     threadLocal.set("当前线程的值");
     ```

  3. **获取值**：
     通过 `get()` 方法获取当前线程的值。

     ```java
     String value = threadLocal.get();
     ```

  4. **清除值**：
     如果不再需要某个线程中的值，可以通过 `remove()` 方法清除。

     ```java
     threadLocal.remove();
     ```

  ### 注意事项

  1. **内存泄漏**：由于 `ThreadLocal` 设计的初衷是线程封闭的，所以如果在使用 `ThreadLocal` 时，没有正确地调用 `remove()` 方法，或者线程长时间运行而不终止，那么可能会导致内存泄漏。因为 `ThreadLocal` 的值是保存在 `Thread` 对象的 `ThreadLocalMap` 中的，而 `Thread` 对象是由 JVM 管理的，它的生命周期通常很长，因此如果 `ThreadLocal` 不再需要时，应该显式地调用 `remove()` 方法。
  2. **线程安全性**：虽然 `ThreadLocal` 提供了线程局部变量，但并不意味着它是线程安全的。如果多个线程同时访问同一个 `ThreadLocal` 变量，并且该变量的值是一个可变对象（mutable object），那么仍然需要额外的同步机制来保证线程安全。
  3. **性能**：由于 `ThreadLocal` 需要为每个线程创建一个独立的副本，因此在使用 `ThreadLocal` 时需要权衡性能与便利性。如果线程数量很多，或者 `ThreadLocal` 变量占用的内存很大，那么使用 `ThreadLocal` 可能会导致内存占用过多。

* `java.util.concurrent.atomic` 是 Java 并发包中的一个重要部分，它提供了一组支持原子操作的类。原子操作是不可中断的操作，即在多线程环境下，这些操作在多线程并发执行时，线程之间不可能通过任何机制（包括 CPU 的中断指令）看到中间状态，或者说，这些操作要么全部完成，要么全部不完成，不会存在中间某个节点上执行的情况。

  `java.util.concurrent.atomic` 包中的类主要用于实现线程安全的数据结构，这些类通常用于在多线程环境中减少同步的开销。该包中的类主要可以分为以下几种类型：

  1. **原子变量**：
     - `AtomicBoolean`：原子布尔值。
     - `AtomicInteger`：原子整数。
     - `AtomicLong`：原子长整数。
     - `AtomicReference<V>`：原子对象引用。

     这些类提供了用于原子操作的常用方法，如 `get()`, `set()`, `compareAndSet()`, `incrementAndGet()`, `decrementAndGet()`, `addAndGet()` 等。

  2. **原子数组**：
     - `AtomicIntegerArray`：支持原子操作的 `int` 数组。
     - `AtomicLongArray`：支持原子操作的 `long` 数组。
     - `AtomicReferenceArray<E>`：支持原子操作的对象引用数组。

     这些类提供了对数组元素进行原子操作的方法。

  3. **原子更新器**：
     - `AtomicIntegerFieldUpdater<T>`：基于反射的原子更新器，用于原子更新某个类的 `volatile int` 字段。
     - `AtomicLongFieldUpdater<T>`：基于反射的原子更新器，用于原子更新某个类的 `volatile long` 字段。
     - `AtomicReferenceFieldUpdater<T,V>`：基于反射的原子更新器，用于原子更新某个类的对象的字段。

     这些类提供了一种在特定类的实例字段上执行原子更新的机制。

  4. **原子标记引用**：
     - `AtomicStampedReference<V>`：这是一个支持原子操作的引用类型，它带有一个整型的“邮票”或“标记”，可以用于解决 ABA（A-B-A）问题。

  5. **原子累加器**：
     - `LongAdder` 和 `LongAccumulator`：这些类提供了比 `AtomicLong` 更高效的并发累加操作，特别是在有大量并发更新操作的情况下。

  `java.util.concurrent.atomic` 包中的类在多线程编程中非常有用，它们提供了一种高效的线程安全机制，而无需使用传统的同步块或锁。然而，需要注意的是，虽然这些类提供了原子操作，但它们并不保证对对象本身的操作是原子的。如果对象本身的状态需要在多线程之间保持一致，可能需要使用其他同步机制。

* Disruptor是一个高性能、无锁并发框架，主要用于在生产者和消费者之间高效地传输数据。其核心思想是通过无锁设计和内存预分配，实现高性能的数据传输，同时保持低延迟。Disruptor使用环形缓冲区（ring buffer）作为数据传输的媒介，通过位运算来快速计算数据在缓冲区中的位置，从而减少了对锁的需求。Disruptor框架的缺点是，虽然它避免了使用传统的锁机制，但为了实现线程安全，它采用了CAS（Compare And Set/Swap）操作，这可能会增加一些额外的开销。然而，在大多数情况下，这种开销是可以接受的，因为Disruptor能够显著提高系统的整体性能和吞吐量。

  以下是一个简单的Disruptor框架的使用示例，展示了如何创建一个生产者、消费者和事件处理器。

  首先，我们需要定义事件类型（即生产者和消费者之间传输的数据类型）：


  ```java
  public class LongEvent {
      private long value;
  
      public LongEvent() {
      }
  
      public LongEvent(long value) {
          this.value = value;
      }
  
      public long getValue() {
          return value;
      }
  
      public void setValue(long value) {
          this.value = value;
      }
  }
  ```
  接下来，我们需要创建事件处理器（即消费者）：


  ```java
  public class LongEventHandler implements EventHandler<LongEvent> {
  
      @Override
      public void onEvent(LongEvent event, long sequence, boolean endOfBatch) throws Exception {
          System.out.println("Event: " + event.getValue());
      }
  }
  ```
  然后，我们需要设置Disruptor：


  ```java
  import com.lmax.disruptor.Disruptor;
  import com.lmax.disruptor.EventHandlerGroup;
  import com.lmax.disruptor.RingBuffer;
  import com.lmax.disruptor.dsl.DisruptorBuilder;
  import com.lmax.disruptor.dsl.ProducerType;
  
  public class DisruptorExample {
  
      public static void main(String[] args) throws Exception {
          
          // 设置事件工厂
          EventFactory<LongEvent> factory = new EventFactory<LongEvent>() {
              @Override
              public LongEvent newInstance() {
                  return new LongEvent();
              }
          };
  
          // 创建Disruptor实例
          Disruptor<LongEvent> disruptor = new DisruptorBuilder<LongEvent>()
                  .setEventFactory(factory)
                  .setProducerType(ProducerType.SINGLE)
                  .setRingBufferSize(1024)
                  .build();
  
          // 注册事件处理器
          EventHandlerGroup<LongEvent> handlers = disruptor.handleEventsWith(new LongEventHandler());
  
          // 启动Disruptor
          disruptor.start();
  
          // 获取环形缓冲区
          RingBuffer<LongEvent> ringBuffer = disruptor.getRingBuffer();
  
          // 模拟生产者发布事件
          for (long l = 0; l < 10; l++) {
              long sequence = ringBuffer.next();
              try {
                  LongEvent event = ringBuffer.get(sequence);
                  event.setValue(l);
              } finally {
                  ringBuffer.publish(sequence);
              }
          }
  
          // 等待一段时间，让事件处理器有足够的时间处理事件
          Thread.sleep(1000);
  
          // 关闭Disruptor
          disruptor.shutdown();
      }
  }
  ```
  在这个示例中，我们定义了一个名为`LongEvent`的事件类型，一个名为`LongEventHandler`的事件处理器，以及一个名为`DisruptorExample`的示例类，该类用于设置和启动Disruptor。在`DisruptorExample`类中，我们首先设置了事件工厂和Disruptor的配置（包括生产者类型和环形缓冲区大小），然后注册了事件处理器，并启动了Disruptor。接着，我们模拟了一个生产者发布事件的过程，将事件发布到环形缓冲区中。最后，我们等待一段时间让事件处理器有足够的时间处理事件，并关闭Disruptor。

* JDK的Future模式是一种并发编程模式，它允许异步执行代码并在未来获取其结果。在Future模式中，调用线程可以提交一个任务给另一个线程或线程池，并立即返回一个Future对象作为任务的代理。这个Future对象代表了尚未完成的任务，并允许调用线程在未来的某个时刻获取任务的结果。

  Future模式通常用于处理长时间运行的任务，例如网络请求或耗时的计算。通过使用Future模式，调用线程可以避免阻塞并继续执行其他任务，同时仍然能够获得任务的结果。

  在Java中，Future模式是通过Future接口来实现的。Java还提供了CompletableFuture类，它是Future接口的实现，并提供了更丰富的功能，例如异步回调和异常处理。

  在JDK中，Future模式的核心是Callable和Future这两个接口。Callable接口类似于Runnable接口，但它允许有返回值，并且可以抛出异常。当调用线程提交一个Callable任务给线程池时，线程池会返回一个Future对象，这个Future对象代表了这个任务的执行结果。调用线程可以通过调用Future对象的get()方法来获取任务的结果，如果任务还没有完成，get()方法会阻塞直到任务完成。

  另外，JDK还提供了FutureTask类，它实现了Runnable和Future接口，可以将Callable任务包装成Runnable任务提交给线程池执行，并返回一个Future对象来获取任务的结果。

  **CompletableFuture是Java 8中引入的一个功能强大的Future实现，用于异步编程**。它是Future接口的扩展，实现了CompletionStage接口，代表一个异步计算的结果。与传统的Future相比，CompletableFuture提供了更丰富的功能，如异步回调、异常处理、组合多个异步操作等。

  CompletableFuture的主要特点包括：

  1. **异步回调**：当CompletableFuture的计算结果完成时，可以通过thenApply、thenAccept、thenCompose等方法注册回调函数，这些回调函数会在结果完成时自动执行。
  2. **异常处理**：当CompletableFuture的计算过程中出现异常时，可以通过exceptionally方法注册异常处理函数，该函数会在异常发生时被调用。
  3. **组合多个异步操作**：可以使用thenCombine、thenAcceptBoth、runAfterBoth等方法将多个CompletableFuture组合在一起，实现更复杂的异步操作。
  4. **显式完成**：与Future不同，CompletableFuture提供了显式完成的方法，如complete、completeExceptionally等，可以在需要时主动设置结果或异常。

  在使用CompletableFuture时，通常需要将其与线程池（如ForkJoinPool）结合使用，以充分利用多核处理器的优势。通过合理地使用CompletableFuture，可以大大提高异步编程的效率和灵活性。

  下面是一个使用`CompletableFuture`的简单例子，它模拟了一个异步计算过程，并在计算完成后执行一些操作：

  ```java
  import java.util.concurrent.CompletableFuture;
  import java.util.concurrent.ExecutionException;
  
  public class CompletableFutureExample {
  
      public static void main(String[] args) throws ExecutionException, InterruptedException {
          // 创建一个异步计算任务，这里简单地模拟了一个耗时操作
          CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
              try {
                  // 假设这是一个耗时操作，比如网络请求或复杂计算
                  Thread.sleep(2000); // 模拟耗时2秒
                  return 42; // 假设计算结果为42
              } catch (InterruptedException e) {
                  // 如果被中断，则返回异常
                  throw new IllegalStateException(e);
              }
          });
  
          // 当计算完成后，打印结果
          future.thenAccept(result -> {
              System.out.println("异步计算结果为: " + result);
          });
  
          // 这里我们等待主线程一段时间，以确保异步操作有足够的时间完成
          // 在实际的生产代码中，通常不会这样做，而是让主线程继续执行其他任务
          Thread.sleep(3000);
  
          // 如果你想在主线程中直接获取结果（可能会阻塞），可以使用get方法
          // 注意：get方法会阻塞当前线程直到异步操作完成
          Integer finalResult = future.get();
          System.out.println("在主线程中获取的异步计算结果为: " + finalResult);
      }
  }
  ```

  在这个例子中，我们创建了一个`CompletableFuture`对象，并使用`supplyAsync`方法提交了一个异步任务。这个任务会模拟一个耗时操作（通过`Thread.sleep`实现），并在完成后返回一个整数结果。然后，我们使用`thenAccept`方法为异步操作注册了一个回调函数，当异步操作完成时，这个回调函数会被调用，并打印出结果。最后，我们让主线程等待一段时间（这里只是为了演示，通常不推荐这样做），然后使用`get`方法在主线程中直接获取异步操作的结果。注意，`get`方法会阻塞当前线程直到异步操作完成。

  在实际应用中，通常会使用`thenApply`、`thenCompose`等方法将多个异步操作组合在一起，以实现更复杂的异步编程逻辑。

* StampedLock是Java 8中引入的一个新的同步原语，它主要用于替代ReentrantLock以提供更高的并发性能，特别是在读操作远多于写操作的场景中。

  StampedLock是一个高性能的读写锁，它支持三种访问模式：写锁、乐观读和悲观读。这三种模式使得StampedLock能够在不同的使用场景下提供更高的吞吐量。

  1. **写锁（writeLock）**：这是一个独占锁，用于修改数据。在写模式下，所有的读和写操作都会被阻塞，直到写操作完成。
  2. **悲观读（readLock）**：这是一个共享锁，用于读操作。多个线程可以同时获得读锁，但写操作会被阻塞。悲观读模式假设最坏的情况，即认为读操作期间可能会有写操作发生，因此会锁定资源以确保数据的一致性。
  3. **乐观读（tryOptimisticRead）**：这是一种特殊的读模式，它不会阻塞其他线程的写操作。在乐观读模式下，线程会先尝试获取一个戳记（stamp），然后读取数据。如果在读取过程中没有发生写操作，那么这次读取就是成功的；如果发生了写操作，那么可以通过检查戳记来发现数据的不一致性，并重新执行读操作。乐观读模式假设最好的情况，即认为读操作期间不太可能有写操作发生，因此不会立即锁定资源。

  此外，StampedLock引入了一个名为“戳记”（stamp）的概念，它是一个长整型的数字，用于标识锁的版本。在读锁和写锁的操作中，都会返回一个戳记，可以用来检查在获取锁期间是否有写操作发生。

  总的来说，StampedLock通过引入乐观读和写锁的优化机制，提高了多线程环境下的并发性能，能够更好地利用多核处理器的优势，减少线程间的竞争和阻塞，从而提升系统的吞吐量和响应速度。

  以下是一个使用`StampedLock`的示例，我们将创建一个简单的`Point`类，并在获取和修改其坐标时使用`StampedLock`来保证线程安全。

  ```java
  import java.util.concurrent.locks.StampedLock;  
    
  public class Point {  
      private int x, y;  
      private final StampedLock stampedLock = new StampedLock();  
    
      public Point(int x, int y) {  
          this.x = x;  
          this.y = y;  
      }  
    
      // 使用乐观读锁来获取距离原点的距离  
      public double distanceFromOriginOptimistic() {  
          long stamp = stampedLock.tryOptimisticRead(); // 尝试乐观读  
          int currentX = x, currentY = y;  
          if (!stampedLock.validate(stamp)) { // 如果在读取过程中有写操作发生  
              // 切换到悲观读锁  
              stampedLock.readLock();  
              try {  
                  currentX = x;  
                  currentY = y;  
              } finally {  
                  stampedLock.unlockRead();  
              }  
          }  
          return Math.sqrt(currentX * currentX + currentY * currentY);  
      }  
    
      // 使用写锁来修改坐标  
      public void move(int newX, int newY) {  
          long stamp = stampedLock.writeLock(); // 获取写锁  
          try {  
              x = newX;  
              y = newY;  
          } finally {  
              stampedLock.unlockWrite(stamp); // 释放写锁  
          }  
      }  
  }
  ```

  在这个示例中，我们定义了一个`Point`类，它有两个字段`x`和`y`表示坐标。我们使用`StampedLock`来保证对这两个字段的并发访问安全。

  - `distanceFromOriginOptimistic()`方法首先尝试使用乐观读锁来获取坐标。如果在获取坐标的过程中有写操作发生（即`stampedLock.validate(stamp)`返回`false`），则切换到悲观读锁来重新获取坐标。
  - `move()`方法使用写锁来修改坐标。在修改坐标之前，它首先获取写锁，并在修改完成后释放写锁。

  注意，在实际应用中，乐观读锁的使用需要谨慎考虑，因为它假设在读取过程中不会有写操作发生。如果实际上写操作非常频繁，那么乐观读锁可能会经常失败，并导致性能下降。在这种情况下，使用悲观读锁可能更为合适。
