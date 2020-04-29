title: JDK中的timer正确的打开与关闭
tags:
  - Timer
categories:
  - Dev
  - Java
date: 2020-04-29 10:59:00
cover: true
---

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/timer.png)
<!-- more -->

## Timer和TimerTask

Timer是jdk中提供的一个定时器工具，使用的时候会在主线程之外起一个单独的线程执行指定的计划任务，可以指定执行一次或者反复执行多次。

TimerTask是一个实现了Runnable接口的抽象类，代表一个可以被Timer执行的任务。

## Timer的调度
```
import java.util.Timer;
import java.util.TimerTask;

public class TestTimer {
    
    public static void main(String args[]){
        new Reminder(3);
    }
    
    public static class Reminder{
        Timer timer;
        
        public Reminder(int sec){
            timer = new Timer();
            timer.schedule(new TimerTask(){
                public void run(){
                    System.out.println("Time's up!");
                    timer.cancel();
                    System.out.println("Time's shutdown!");
                }
            }, sec*1000);
        }
    } 
}
```
控制台输出
```
Time's up!
Time's shutdown!
```
从这个例子可以看出一个典型的利用timer执行计划任务的过程如下：
* new一个TimerTask的子类，重写run方法来指定具体的任务，在这个例子里，我用匿名内部类的方式来实现了一个TimerTask的子类
* new一个Timer类，Timer的构造函数里会起一个单独的线程来执行计划任务。

jdk的实现代码如下：
```
public Timer() {
        this("Timer-" + serialNumber());
    }

    public Timer(String name) {
        thread.setName(name);
        thread.start();
    }
```

## Timer的关闭
在JDK1.5以后，文档中有这么一句话：
对 Timer 对象最后的引用完成后，并且 所有未处理的任务都已执行完成后，计时器的任务执行线程会正常终止（并且成为垃圾回收的对象）。但是这可能要很长时间后才发生。

### System.gc()
系统默认当Timer运行结束后，如果没有手动终止，那么则只有当系统的垃圾收集被调用的时候才会对其进行回收终止。
因此，可以手动System.gc();
但是Sytem.gc()在一个项目中是不能随便调用的。因为一个tomcat只启动一个进程，而JVM的垃圾处理器也只有一个，所以在一个工程里运行System.gc也会影响到其他工程。 

### cancle()
首先看cancle方法的源码
```
public void cancel() {
        synchronized(queue) {
            thread.newTasksMayBeScheduled = false;
            queue.clear();
            queue.notify();  // In case queue was already empty.
        }
    }
```
没有显式的线程stop方法，而是调用了queue的clear方法和queue的notify方法，clear是个自定义方法，notify是Objec自带的方法，很明显是去唤醒wait方法的。

clear方法
```
 /**
  * Removes all elements from the priority queue.
  */
void clear() {
        // Null out task references to prevent memory leak
        for (int i=1; i<=size; i++)
            queue[i] = null;

        size = 0;
    }
```
clear方法很简单，就是去清空queue，queue是一个TimerTask的数组，然后把queue的size重置成0，变成empty.还是没有看到显式的停止线程方法，回到最开始new Timer的时候，看看new Timer代码：
```
public Timer() {
        this("Timer-" + serialNumber());
    }

   /**
     * Creates a new timer whose associated thread has the specified name.
     * The associated thread does <i>not</i>
     * {@linkplain Thread#setDaemon run as a daemon}.
     *
     * @param name the name of the associated thread
     * @throws NullPointerException if {@code name} is null
     * @since 1.5
     */
    public Timer(String name) {
        thread.setName(name);
        thread.start();
    }
```
看看这个内部变量thread:
```
 /**
  * The timer thread.
 */
 private TimerThread thread = new TimerThread(queue);
```
不是原生的Thread,是自定义的类TimerThread.这个类实现了Thread类，重写了run方法，如下：
```
public void run() {
        try {
            mainLoop();
        } finally {
            // Someone killed this Thread, behave as if Timer cancelled
            synchronized(queue) {
                newTasksMayBeScheduled = false;
                queue.clear();  // Eliminate obsolete references
            }
        }
    }
```
最后是这个mainLoop方法
```
   /**
     * The main timer loop.  (See class comment.)
     */
    private void mainLoop() {
        while (true) {
            try {
                TimerTask task;
                boolean taskFired;
                synchronized(queue) {
                    // Wait for queue to become non-empty
                    while (queue.isEmpty() && newTasksMayBeScheduled)
                        queue.wait();
                    if (queue.isEmpty())
                        break; // Queue is empty and will forever remain; die

                    // Queue nonempty; look at first evt and do the right thing
                    long currentTime, executionTime;
                    task = queue.getMin();
                    synchronized(task.lock) {
                        if (task.state == TimerTask.CANCELLED) {
                            queue.removeMin();
                            continue;  // No action required, poll queue again
                        }
                        currentTime = System.currentTimeMillis();
                        executionTime = task.nextExecutionTime;
                        if (taskFired = (executionTime<=currentTime)) {
                            if (task.period == 0) { // Non-repeating, remove
                                queue.removeMin();
                                task.state = TimerTask.EXECUTED;
                            } else { // Repeating task, reschedule
                                queue.rescheduleMin(
                                  task.period<0 ? currentTime   - task.period
                                                : executionTime + task.period);
                            }
                        }
                    }
                    if (!taskFired) // Task hasn't yet fired; wait
                        queue.wait(executionTime - currentTime);
                }
                if (taskFired)  // Task fired; run it, holding no locks
                    task.run();
            } catch(InterruptedException e) {
            }
        }
    }
}
```
可以看到wait方法，之前的notify就是通知到这个wait，然后clear方法在notify之前做了清空数组的操作，所以会break，线程执行结束，退出。

## Listener中的Timer
很多业务中需要Timer一直执行，不会执行一次后就关闭，上面的例子中，timer调用cancel方法后，该timer就被关闭了。

监听器的实现方式有多种，这里我们说一下实现`ServletContextListener`接口。
该接口中有2个方法
```
public interface ServletContextListener extends EventListener {
    void contextInitialized(ServletContextEvent var1);

    void contextDestroyed(ServletContextEvent var1);
}
```
即上下文的初始化和销毁。

我们来看一个实例
Listener
```
public class MyListener implements ServletContextListener {

    private Log log = LogFactory.getLog(MyListener.class);

    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        Timer timer = new Timer();
        timer.schedule(new MyTask(),5000,5000);
    }

    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
    }
}
```
Task
```
public class MyTask extends TimerTask {

    @Override
    public void run() {
        System.out.println("timer 正在执行");
    }
}
```
这样当程序启动的时候，在监听器的初始化中，timer会梅5秒执行一次
```
timer 正在执行
timer 正在执行
timer 正在执行
timer 正在执行
```
此次程序中我们没有去调用timer的cancel方法，这样会存在一个问题，就是产生的timer一直不会被关闭，就像上面说的只有当系统的垃圾收集被调用的时候才会对其进行回收终止。

同时tomcat日志会打印错误
```
28-Apr-2020 14:23:24.892 警告 [http-nio-8080-exec-23] org.apache.catalina.loader.WebappClassLoaderBase.clearReferencesThreads Web应用程序[nyzft]似乎启动了一个名为[Timer-3]的线程，但未能停止它。这很可能会造成内存泄漏。线程的堆栈跟踪：[
 java.lang.Object.wait(Native Method)
 java.lang.Object.wait(Object.java:502)
 java.util.TimerThread.mainLoop(Timer.java:526)
 java.util.TimerThread.run(Timer.java:505)]
```
问题的原因就是我们没有手动去关闭timer，但是如果去调用cancel方法，真实的场景timer只会被执行一次，不符合业务要求。
因此可以通过listener的contextDestroyed去关闭timer
```
public class MyListener implements ServletContextListener {

    private Log log = LogFactory.getLog(MyListener.class);

    private Timer timer;

    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        timer = new Timer();
        timer.schedule(new MyTask(),5000,5000);
        System.out.println("执行");
    }

    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        timer.cancel();
        System.out.println("关闭");
    }
}
```
启动程序，过几秒钟后再关闭程序，查看控制台输出
```
执行
timer 正在执行
timer 正在执行
[2020-04-29 09:44:19,609] Artifact ssm-nyzft:war exploded: Artifact is deployed successfully
[2020-04-29 09:44:19,609] Artifact ssm-nyzft:war exploded: Deploy took 38,550 milliseconds
timer 正在执行
timer 正在执行
timer 正在执行
timer 正在执行
E:\Kit\Tomcat\tomcat8\apache-tomcat-8.5.39\bin\catalina.bat stop
Disconnected from the target VM, address: '127.0.0.1:52706', transport: 'socket'
Using CATALINA_BASE:   "C:\Users\Administrator\.IntelliJIdea2019.1\system\tomcat\Unnamed_ssm-nyzft_2"
Using CATALINA_HOME:   "E:\Kit\Tomcat\tomcat8\apache-tomcat-8.5.39"
Using CATALINA_TMPDIR: "E:\Kit\Tomcat\tomcat8\apache-tomcat-8.5.39\temp"
Using JRE_HOME:        "E:\Kit\JDK\JDK"
Using CLASSPATH:       "E:\Kit\Tomcat\tomcat8\apache-tomcat-8.5.39\bin\bootstrap.jar;E:\Kit\Tomcat\tomcat8\apache-tomcat-8.5.39\bin\tomcat-juli.jar"
29-Apr-2020 09:44:40.511 淇℃伅 [main] org.apache.catalina.core.StandardServer.await A valid shutdown command was received via the shutdown port. Stopping the Server instance.
29-Apr-2020 09:44:40.512 淇℃伅 [main] org.apache.coyote.AbstractProtocol.pause Pausing ProtocolHandler ["http-nio-8081"]
29-Apr-2020 09:44:40.638 淇℃伅 [main] org.apache.coyote.AbstractProtocol.pause Pausing ProtocolHandler ["ajp-nio-8009"]
29-Apr-2020 09:44:40.750 淇℃伅 [main] org.apache.catalina.core.StandardService.stopInternal Stopping service [Catalina]
关闭
```





