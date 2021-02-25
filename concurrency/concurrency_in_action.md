# Concurrency-in-action
java并发编程实战
## 1 并发源头
CPU、内存、I/O设备三者速度不一致
* CPU增加缓存，以平衡内存速度
* OS增加进程、线程，以分时复用CPU，均衡CPU与I/O设备的速度
* 编译程序优化指令执行次序，使缓存能更有效利用
###  1.1缓存导致的可见性问题
每科CPU有自己的缓存，多个线程在不同CPU上执行时，线程操纵的是不同的CPU缓存，难以保证CPU缓存与内存的一致性      
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2020/08/18/20-41-44-c88351ef301602298730f2cf718c84b9-20200818204143-e0e48e.png)
```java
public class Task{
    private long count = 0;
    private void add10k(){
        int idx=0;
        while(idx++ < 1000){
            count+=1;
        }
    }
    public static long calc(){
        final Task task = new Task();
        Thread th1 = new Thread( () -> {
            test.add10k();
        });
        Thread th2 = new Thread( () -> {
            test.add10k();
        });
        th1.start();
        th2.start();
        th1.join();
        th2.join();
        return count;
    }
}
```
**假设线程A/B同时执行，同时将count=0读到各自的缓存中，执行完count+=1后，各自缓存中值为1，同时写入内存后，内存中为1，，而不是2。之后各自的CPU缓存中有了count值，两个线程都基于缓存中count计算，最后count值小于20000**.
### 1.2 线程切换导致的原子性问题
count+=1   
* 把变量count从内存加载到CPU寄存器
* 在寄存器进行+1操作  
* 将结果写入内存(也有可能写入缓存)   
任务切换发生在任何一条CPU执行执行完毕。
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2020/08/18/20-54-16-1daa9da1f448d8f5702f668fb212f5db-20200818205416-eda2c4.png)
### 1.3 编译优化导致的有序性问题
双重锁的单例模型
```java
public class Singleton{
    static Singleton instance;
    static Singleton getInstance(){
        if(instance == null){
            synchronized(Singleton.class){
                if(instance == null){
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
new操作
* 1 分配内存M
* 2 在内存M上初始化Singleton对象
* 3 M的地址赋值给instance变量   
编译器优化后
* 1 分配内存M
* 2 M的地址赋值给instance变量 
  *  **A在此处发生线程切换时，B线程获得的instance不为null，但是还未初始化完成，触发空指针异常**
* 3 在内存M上初始化Singleton对象
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2020/08/18/21-06-13-2aab54accc7b6cf8c82d7290d60a605a-20200818210612-41f833.png)
## 2 java内存模型
缓存的可见性，编译优化的有序性，可通过按需禁用缓存和编译优化。java内存模型规范JVM方法，volatile,synchronized,final关键字以及Happens-Before原则
### volatile
禁用缓存，对这个变量的读写，不能使用CPU缓存，必须从内存中读取或写入
### Happens-Before
前面一个操作的结果对于后续操作是可见的。Happens-Before约束了编译器优化，要求编译优化后遵守该原则。
* 程序的顺序性原则
    * 程序前面对某个变量的修改对后续操作可见
* volatile原则
  * 对一个volatile变量的写操作，Happens-Before于后续对这个volatile变量的读操作
* 传递性原则
  * A Happens-Before B,B Happens-Before C,那么A Happens-Before C.
   ```java
   class volatileexam{
       int x = 0;
       volatile boolean v = fale;
       public void writer(){
           x = 42;          
           v = true;  //顺序性原则，x=42  happens-before v=true 对v=true可见
       }
       public void reader(){
           if(v == true){  //volatile原则，v的写 happens-before v的读         
               // what is the value of X ???  //传递性原则  x=42的写 happens-before v的读
           }
       }
   }
   ```
* 管程中锁的规则
  * 对一个锁的解锁happens-before后续对这个锁的加锁
    ```java
    synchronized(this){//自动加锁
        if( this.x < 12){
            this.x = 12;
        }
    }//自动解锁  先获得锁的线程 对变量的修改 解锁后 可见与后获得锁的线程
    ```
* 线程start()原则
  * 主线程A启动子线程B后，子线程B能够看到主线程在启动子线程B前的操作
    ```java
    Thread B = new Thread( () -> {
        //线程A中启动线程B，即调用B.start方法，对var的修改，此处可见
        //
    });
    var = 77;
    B.start();
    ```
*  线程join原则
   *  主线程A调用B子线程，并等待子线程B返回，子线程B中对共享变量的操作，A线程可以看到
      ```java
      Thread B = new Thread( () -> {
          var=55;
      })//
      B.start();
      B.join();
      //此处可见var=55
      ```
* final
  * 指示这个变量生而不变，可以尽情优化
  * 对final类型变量重拍进行了约束，1.5以后，只需要保证构造函数没有逸出
* 线程中断原则
  * 对线程interrupt方法的调用先行发生于被中断线程的代码检测到中断的发生，可通过Thread.interrupted检测是否有中断发生
* 对象终结原则
  * 一个对象的初始化完成（构造函数执行完毕）先行发生于它的finalize方法的开始
## 3 互斥锁
* 同一时刻只有一个线程执行---互斥
* 称需要互斥执行的代码段为临界区
* 锁和锁保护的资源要关联
### synchronized
 * 当修饰静态方法时，锁定的是当前类的Class对象
 * 当修饰非静态方法时，锁定的是当前实例对象this
    ```java
    class X{
        synchronized( X.class) static void bar(){}
    }
    class X{
        synchronized(this) static void bar(){
            //临界区 互斥
        }
    }
    ```
    ```java
    class safecalc{
        long value = 0l;
        synchronized long get( ){ //执行完毕addOne 后，value值不一定对get方法可见，管程中锁的规则，只保证后续对这个锁加锁的可见性，所以要加synchronized !!!
            return value;
        }
        synchronized void addOne( ){
            value += 1;
        }
    }
    ```
* 一把锁可以锁多个资源！！，不能用多把锁保护一个资源
### 保护多个资源
**如果资源没有关系，那么声明不同的锁管理不同的资源，否则，找一个粒度更大的锁，能够覆盖所有要保护的资源。以及对于所有的访问路径，也要设置合适的锁**
#### 保护没有关联关系的资源
* 申请不同的锁对不同关联的资源进行保护
    ```java
    class Account{
        private final Object balLock = new Object();//不同锁管理不同资源，也可以this锁住所有资源 final  **确保了是同一个对象**！！！

        private  Integer balance;

        private final Object pwLock = new Object();

        private String password;

        void withdraw( Integer amt){
            synchronized(balLock){
                if (this.balance > amt ){
                    this.balance -= amt;
                }
            }
        }
        void updatePassword(String pw) {
            synchronized(pwLock){
                this.password = pw;
            }
        }
    }
    ```
#### 保护有关联关系的资源 
```java
class Account{
    private int balance;
    synchronized void transfer(Account target, int amt){//synchronized 锁住的是this，只保护了this.balance,当A转账B，B转账C，锁住的是 A.this B.this保护的是A.balance,B.balance
        if(this.balance > amt){
            this.balance -= amt;
            target.balance += amt;
        }
    }
}
```
* 只要锁能覆盖所有受保护的资源即可
   ```java
     class Account{
        private Object lock;
        private int balance;

        private Account(){}        //将默认构造private
        public Account(Object lock){ //创建Account时，传入同一个object lock
            this.lock = lock;
        }
       void transfer( Account target, int amt){
           synchronized(locak) {
             if(this.balance > amt){
            this.balance -= amt;
            target.balance += amt;
        }
       }
       }
    }
   ```
   ```java
       class Account{
        private int balance;
   
       void transfer( Account target, int amt){
           synchronized(Account.class) { //所有Account 对象共享一个Account.class 
             if(this.balance > amt){
            this.balance -= amt;
            target.balance += amt;
        }
       }
       }
    }
   ```
## 死锁
线程1 A转账B，线程2 B转账A，线程1获得A，等待获得  B，线程2 获得B，等待获得A。**一组互相竞争资源的线程因互相等待造成的永久阻塞现象**。
```java
class Account{
    public int balance;
    void transfer(Account target, int emt){
        synchronized(this){
            synchronized(target){
                if(this.balance > emt){
                    this.balance -= emt;
                    target.balance += emt;
                }
            }
        }
    }
}
```
死锁条件，只要破坏一个就可以避免死锁
* 互斥 共享资源只能被一个线程占有
* 占有且互相等待，线程T1已经占有X，在等待获得Y时，不释放X             --------可以一次性申请所有资源
* 不可抢占，其他线程不能强行抢占T1线程的占有资源      --------占用资源的线程进一步申请资源时，申请不到就释放获得的资源
* 循环等待，线程T1等待线程T2占有的资源，线程T2也等待线程T1占有的资源-------按照资源的序号，顺序或倒序申请资源
 ```java
 //占有且互相等待
 class Allocation{
     private List<Object> als = new ArrayList<>();
     synchronized boolean apply( Object from, Object to){
         //一次性 申请所有资源，否则 失败
         if(als.contains(from) || als.contains(to)){
             return false;
         }
         else{
             als.add(from);
             als.add(to);
         }
         return true;
     }
     synchronized void free(Object from, Object to){
         als.remove(from);
         als.remove(to);
     }
 }
 public Account{
     private Allocation actr; /////应为 单例 ！！！
     private int balance;
     void transfer(Account target, int emt){
         //一次性申请转出、转入账户，直到成功
         while(!actr.apply(this, target)){}
         try{
             synchronized(this){
                 synchronized(target){
                     //.....
                     if(this.balance > emt){
                         this.balance -= emt;
                         target.balance += emt;
                     }
                 }
             }
         }finally{
             actr.free(this,target);
         }
     }
 }
 ```

 ```java
 // 循环等待
 class Account{
     private int balance;
     public int id;
     void transfer(Account target, int emt){
         Account left=this;
         Account right=target;
         if(this.id < target.id){
                left=target;
                right=this;
         }
         synchronized(left){
             synchronized(right){
                 ////...................
             }
         }
     }
 }
 ```
## 06等待通知
线程首次获取互斥锁，当线程要求的条件不满足时，释放互斥锁，进入等待状态，当要求的条件满足时。通知等待的线程，重新获取互斥锁。
每个互斥锁有自己独立的等待队列。
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2020/08/20/21-02-10-e89b0f1533e58a49e23a966e063a11aa-20200820210210-589650.png)
* 首先等待队列中的一个线程获得锁进入临界区后，条件不满足，那么释放锁，并再次进入等待队列，其他等待队列中的线程可以获得锁。
* 当条件满足时，`notify` `notifyAll`可以通知等待队列中的线程，条件曾经满足过。
* 被通知的线程，再次获得锁，进入临界区，再次判断条件是否满足
* 如果锁住的是target，那么一定是`target.wait` `target.notify` `target.notifyAll`(这三者使用的前提是已经获得锁)
    ```java
    class Allocation{
        List<Object> als = new ArrayList<>();
        synchronized void apply(Object from,Object to){
            while(als.contains(from) || als.contains(to)){
                try{
                    wait();
                }catch(Exception e){
                }
            }
            als.add(from);
            als.add(to);
        }
        synchronized void free(Object from,Object to){
                als.remove(fomr);
                als.remove(to);
                notifyAll();
        }
    }
    ```
* 尽量使用`notifyAll`，通知等待队列所有线程。`notify`会通知等待队列中随机一个线程
* 范式 如下，尽量誊抄
    ```java 
    while(条件不满足){
        wait();
    }
    ```
## 07 安全、活跃性以及性能问题
### 安全性问题
线程安全程序，即解决原子性问题、可见性问题、有序性问题。只在**存在共享数据且该数据会发生变化，即有多个线程会同时读写数据的时候需要分析**
* 数据竞争
  * 多个线程访问数据，其中至少一个线程会写数据
* 竟态条件
  * 程序的执行结果依赖于线程执行的顺序
  * 程序的执行依赖于某个状态变量，当某个线程发现状态变量满足并执行时，其他线程同时修改了状态变量，并且状态变量条件不满足执行条件了。
    ```java
    if( 状态变量满足执行条件){
        执行操作；
    }
    ```
### 活跃性问题
某个操作无法继续执行下去，死锁、活锁、饥饿
* 活锁
  * 相互让步
  * 谦让时，随机等待一个时机
* 饥饿
  * 线程因无法访问所需资源而无法执行下去
  * 资源充足
  * 避免持有锁的线程长时间运行
  * 公平分配
    * 公平锁，线程的等待队列的线程的等待有顺序，先来的先获得资源
### 性能问题
* 阿姆达尔定律   
   $$ S=\frac{1}{((1-p)+\frac{p}{n})}$$
   n 为CPU核数，p为并行百分比，1-p为串行百分比，串行比为5%时，最高提高性能20%。
* 使用无锁的算法和数据结构
  * 线程本地存储
  * 写时复制
  * 乐观锁
* 减少锁的持有时间
  * 锁的粒度要小
  * 读写锁
* 指标
  * 吞吐量 ---单位时间能处理的请求数量
  * 延迟 ---从发出请求到收到响应的时间
  * 并发量 ---同时处理的请求数量
## 08 管程
管程---管理共享变量以及对共享变量的操作过程，让它们支持并发，管理类的成员变量和成员方法，让这个类是线程安全的。
### MESA模型
* 互斥问题
  * 同一时间只允许一个线程访问资源
  * 将共享变量及其对应操作封装起来
* 同步问题
  * 线程之间的通信和合作
  * 条件变量和等待队列
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2020/08/29/16-02-53-21b72f68f8ac222e870493a9526d7d7b-20200829160253-f6fde2.png)
* 框的入口有一个等待队列，只允许一个线程进入入口，其他进入到入口等待队列中
* 当线程的发现条件不满足时，通过`wait`进入相应条件变量的等待队列中
* 当条件满足时，`notify notifyAll`通知线程，线程从条件变量的等待队列中出去，再次进入到入口等待队列
    ```java 
     public class BlockedQueue<T>{
          final Lock lock = new ReentrantLock();
          final Condition notFull = lock.newCondition(); //不满
          final Condition notEmpty = lock.newCondition;//不空
          void enq(T x){
              lock.lock();
              try{
                  while(队列已满)
                  {
                           //等待队列不满
                    notFull.await();
                  }
                    //入队操作
                    //通知可以出对
                    notEmpty.signal();
              }finally{
                lock.unlock();
              }
          }
          void deq(){
              lock.lock();
              try{
                        while(队列已空)
                        {
                            notEmpty.await();
                        }
                        //出队操作
                        notFull.signal()；
              }finally{
                  lock.unlock();
              }
          }
     }
    ```
    * await 等同于wait 
    * signal 等同于notify
### wait 
```java
while(条件不满足){
    wait();
}
```  
* MESA管程模型中，T2通知T1后，T2继续执行，T1不立即执行，**从条件变量的等待队列进入到入口等待队列中 ?????????**。好处，notify不用放到代码最后，副作用，当T1再次执行时，可能曾经满足的条件现在不满足，所以需要以循环的方式检测条件变量。
* 一个线程执行了wait方法以后，它不会再继续执行了，直到被notify唤醒，如果采用if，那么当唤醒后，就会直接执行while后的代码，而此时条件或许已经不满足了
### notify VS notifyAll
尽量使用notifyAll,除非
* 所有等待线程拥有相同的等待条件
* 所有等待线程被唤醒后，执行相同的操作
* 只需要唤醒一个线程
* 重点---while里面的等待条件是相同的
* 对于队列不满这个条件变量，其阻塞队列里的所有线程都是在等待 队列不满这个条件。都是执行下面3行代码
    ```java
    while(队列已满){
        notFull.await();
    }
    ```
### 总结
java中的管程只有一个条件变量,`synchronized`修饰的代码块在编译期间会自动生相关加锁和解锁的代码    
## java线程的生命周期
### 通用线程生命周期
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2020/08/29/17-13-15-4b04a01a989f1b7d114c7fb47e4ff91b-20200829171315-6cb4ac.png)
* 初始状态
  * 语言层面 线程被创建，不允许分配CPU执行
  * 操作系统层面，未创建线程
* 可运行状态
  * 语言层面 可以分配CPU执行
  * 操作系统 已创建线程
  * 运行状态
    * 被分配到CPU的线程
  * 休眠
    * 运行中的线程调用阻塞的API（阻塞方式读写硬盘文件）或等待某个事件（条件变量），释放CPU。当等待的事件完成了，进入可执行状态
    * 线程执行完毕
### java线程生命周期
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2020/08/29/17-20-11-302e964599179fbb679ea1543d75a4ee-20200829172011-69451f.png)
* 六大状态
  * new
  * Runnable
  * Blocked
  * Waiting
  * Timed_waiting
  * Terminated
* 可运行--->休眠
  * runnable->Blocked
    * 线程等待synchronized的隐式锁，当获得锁时，blocked--->runnable
    * java在调用阻塞api时，操作系统状态上会阻塞，java线程状态不发生变化，还是runnable状态。jvm不关系操作系统调度相关状态，等待cpu使用（操作系统可执行状态）与等待i/o（操作系统休眠状态）没区别，归入runnable状态
  * runnable--->Waiting
    * 获得synchronized隐式锁的线程调用无参的`object.wait（）`
    * 无参数的`thread.join()`，执行完毕Waiting--->runnable
    * 调用`LockSupport.park()`runnable--->waiting,`LockSupport.unpark(Thread thread)` waiting--->runnable
  * runnable--->Timed_waiting
    * `Thread.sleep(long millis)`
    * `object.wait(long timeout)`,获得管程隐式锁的线程
    * `Thread.join(long millis)`
    * `LockSupport.parkNanos(object blocker,long deadline)`
    * `LockSupport.parkUntil(long deadline)`
* 初始状态--->可运行/运行
  * 继承Thread对象，重写run方法
     ```java
     class mythread extends Thread{
         public void run{

         }
     }
     ```
   * 实现Runnable 接口，重写run方法，并将该类作为创建thread对象的参数
    ```java
    class Runner implements Runnable{
        @Override
        public void run(){

        }
    }
    Thread thread = new Thread(new Runner());
    ```
    之后调用`thread.start()`进入runnable状态
* 可运行/运行--->终止 runnable--->terminated
  * stop() @Deprated 已抛弃，立即杀死线程，可能来不及释放锁
  * interrupt 方法
    * 异常方式
      * A处于waiting 或者Timed_waiting 时，其他线程调用A的interrupt，A会回到runnable状态
        * wait join sleep 都会throws interruptedException异常,触发条件就是其他线程调用了该线程的interrupt 方法
      * A处于runnable状态时，
        * 并且阻塞在`java.nio.channels.interruptibleChannel`时，其他线程调用A.interrupt，线程A触发java.nio.channels,ClosedByInterruptException
        * 阻塞在`java.nio.channels.Selector`上，其他线程调用A.interrupt，线程A的`java.nio.channels.Selector`会立即返回？？？？
    * 主动方式
      * 调用`A.isInterrupted()`方法，检测自己是不是被中断了
### 总结
可通过jstack命令或者JAVA VisualVM工具导出线程栈信息
## 10 线程数量
### 目的提升I/O利用率和CPU利用率
* CPU密集计算型
  * 线程数量=CPU核数+1（+1为线程偶尔内存页失效或阻塞时，可以替补）
* I/O密集
  * 最佳线程=CPU核数×（1+（IO耗时/CPU耗时））
### 总结
压测时，关注CPU、IO利用率和性能指标（响应时间、吞吐量）之间的关系
## 11  局部变量的线程安全性
### 方法的调用过程
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2020/09/05/11-31-27-5f89771fdca420aab336bb8f1984902c-20200905113127-5a9ef9.png)
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2020/09/05/11-34-47-dc8e0c92a7bed2fc8e21cc335a955e78-20200905113447-bd41a9.png)
* CPU通过堆栈寄存器（调用栈）获得调用方法的参数和返回地址
* 每个方法在调用栈里都有自己的独立空间，为栈帧，在栈帧里有对应方法需要的参数和返回地址。当调用方法时，创建栈帧并压入调用栈；方法返回时。对应栈帧会自动弹出。栈帧和方法同生共死。
### 局部变量
局部变量放在调用栈里，如果一个变量想跨越方法边界，就必须创建在堆里。

![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2020/09/05/11-38-30-5ccd5e3491cb02d5dab3557a79ce7b3b-20200905113830-f40c42.png)
### 线程的调用栈
每个线程有自己独立的调用栈，局部变量保存在线程自己的调用栈的对应方法的栈帧里，不会共享，没有伤害。
### 线程封闭
方法内部的局部变量，不会和其他线程共享，没有并发问题。是一种解决并发问题的思路   
数据库连接池通过线程封闭技术，保证一个Connection一旦被一个线程获取后，在这个线程关闭Connection之前的这段时间，不会再分配给其他线程。  
## 面向对象思想写并发
### 封装共享变量
* 将共享变量作为对象属性封装在内部，对所有公共方法制定并发访问策略。
* 对于不会发生变化的共享变量final修饰
```java
public class Counter{
    private long value;
    synchronized long get(){
        return value;
    }
    synchronized long add(){
        return ++value;
    }
}
```
#### 识别共享变量之间的约束条件
共享变量之间的约束条件，基本表现在if语句
```java
public class SafeWM{
    private final AtomicLong upper = new AtomicLong(0);// 上限
    private final AtomicLong lower = new AtomicLong(0);//下限

    void setUpper(long v){//设置上限的时候 判断下
        if(v < lower.get()){
            throw new IllegalArgumentException();
        }
        upper.set(v);
    }
    
    void setLower(long v){//设置下限的时候 判断下
        if(v > upper.get()){
            throw new IllegalArgumentException();
        }
        lower.set(v);
    }

}
```
#### 制定并发访问策略
* 避免共享 利用线程本地存储以及为每个任务分配独立的线程
* 不变模式  Actor模式，CSP模式，函数式编程
* 管程及其他同步API 
  * 优先使用成熟的工具类
  * 低级同步原语
  * 避免过早优化
## 14 Lock&Condition 上
Java通过Lock和Condition接口来实现管程，Lock用于解决互斥问题，Condition用于解决同步问题。
死锁问题破坏不可抢占条件，synchronized无能为力。   
不可抢占，其他线程不能强行抢占T1线程的占有资源      --------占用资源的线程进一步申请资源时，申请不到就释放获得的资源
synchronized申请资源不得时，会进入阻塞状态，没有机会会被唤醒
* 能够响应中断  进入阻塞状态后，能够响应中断信号,能被中断信号唤醒，有机会释放锁
* 支持超时 在一段时间内没有获得锁，则返回错误,而不是阻塞
* 非阻塞的获取锁，获不得锁则直接返回            
对应API
* `void lockInterruptibly() throws InterruptedException`支持中断的lock
* `boolean tryLock(long time,TimeUnit unit) throws InterruptedException;`支持超时的lock
* `boolean tryLock()` 支持非阻塞获得锁lock
### Lock 可见性保证
* lock 常用模板`try finally`
```java
class X{
    private final Lock rt1 = new ReentrantLock();
    int value;
    public void addOne(){
        rtl.lock();
        try{
            value += 1;
        }finally{
            rtl.unlock();
        }
    }
}
```
* ReentrantLock内部有一个volatile的成员变量state，获得锁时，读写state值，解锁时，也会读写state值。
```java
class SimpleLock{
    volatile int state;
    lock(){
        //
        state = 1;
    }
    unlock(){
        //
        state = 0;
    }
}
```
X addone的可见性分析
* 顺序性原则 value+=1 Happens-Before unlock()
* volatile原则 unlock() Happens-Before 另一个线程lock()
* 传递性原则 value+=1 Happens-Before 另一个线程的lock()
### 可重入锁
一个锁可以锁多遍，线程可以重复获得同一把锁。   
可重入函数，多个线程可以同时调用的函数,每个线程都能获得正确的结果，同时在一个线程内支持线程切换，无论切换多少次都是正确的。线程安全。
```java
class X{
    private final Lock rt1 = new ReentrantLock();
    int value;
    public int get(){
        rtl.lock();
        try{
            return value;
        }finally{
            rtl.unlock();
        }
    }
    public void addOne(){
        rtl.lock();
        try{
            value = 1+ get(); //可以多次获得锁，否则阻塞
        }finally{
            rtl.unlock();
        }
    }
}
```
#### 公平锁、非公平锁
```
public ReentrantLock() {
    sync= new NonfairSync();
}
public ReentrantLock(boolean fair){
    sync = fair ? new FairSync() : new NonfairSync();
}
```
* 公平锁，唤醒的策略就是唤醒等待时间最长的线程
* 非公平锁，不能保证所有线程都能被唤醒
#### 锁的最佳实践
* 永远只在更新对象的成员变量时加锁
* 永远只在访问可变的成员变量时加锁
* 永远不在调用其他对象方法时加锁
### 总结
lock 有别于synchronized隐式锁
* 能响应中断
* 支持超时
* 非阻塞的获取锁
* synchronized死锁 破坏占有且互相等待条件，一个第三方的资源管理class（条件变量优化）
* lock死锁 破坏不可抢占条件，
```java
class Account{
    private int balance;
    private final Lock lock = new ReentrantLock();
    void transfer(Account tar, int amt){
        while(true){
            if(this.lock.tryLock()){
                try{
                        if(tar.this.lock.tryLock()){
                            try{
                                this.balance -= amt;
                                target.balance += amt;
                            }finally{
                                tar.lock.unlock();
                            }//finally
                        }//tar.lock
                }//try
            }// if this.lock 
        }//while
    }//transfer
}
```
* while true无法跳转
* 活锁问题
```java
class Account{
    private int balance;
    private final Lock lock = new ReentrantLock();
    void transfer(Account tar, int amt){
        while(true){
            if(this.lock.tryLock()){
                try{
                        if(tar.this.lock.tryLock()){
                            try{
                                this.balance -= amt;
                                target.balance += amt;
                                break;// new add 跳出循环
                            }finally{
                                tar.lock.unlock();
                            }//finally
                        }//tar.lock
                }//try
            }// if this.lock 
            Thread.sleep(随机时间);//避免活锁
        }//while
    }//transfer
}
```
## 15 Lock&Conditiont 2
Condition实现了管程的条件变量,管程只有一个条件变量而Lock&Condition可以有多个
### 两个条件变量实现阻塞队列
```java
     public class BlockedQueue<T>{
          final Lock lock = new ReentrantLock();
          final Condition notFull = lock.newCondition(); //不满
          final Condition notEmpty = lock.newCondition;//不空
          void enq(T x){
              lock.lock();
              try{
                  while(队列已满)
                  {
                           //等待队列不满
                    notFull.await();
                  }
                    //入队操作
                    //通知可以出对
                    notEmpty.signal();
              }finally{
                lock.unlock();
              }
          }
          void deq(){
              lock.lock();
              try{
                        while(队列已空)
                        {
                            notEmpty.await();
                        }
                        //出队操作
                        notFull.signal()；
              }finally{
                  lock.unlock();
              }
          }
     }
```
Lock&Condition实现的管程里只能使用`await() signal() signalAll()`    
synchronized管程只能使用`wait notify notifyAll`   
### 同步异步
调用方是否需要等待结果--同步，还是立即返回---异步     
Java默认同步，可通过下列方式实现异步
* 调用方创建一个子线程，在子线程中执行方法调用，称异步调用
* 方法实现时，创建一个新线程执行主要逻辑，主线程直接return，这种方法----异步方法
#### Dubbo源码
TCP协议层，发送完RPC请求后，线程不会等待RPC的响应结果，而平时大多RPC为同步。存在异步转同步过程。  
当RPC返回结果之前，阻塞调用线程，让调用线程等待，当RPC结果返回后在唤醒调用线程，让调用线程再次执行。
```java
public class DubboInvoker{
    Result doInvoke(Invocation inv){
        return currentClient
                      .request(inv,timeout)
                      .get();
    }
}
```
```java
private final Lock lock = new ReentrantLock();
private final Condition done = lock.newCondition();

//调用方调用get阻塞
Object get(int timeout){
    long start = System.nanoTime();
    lock.lock();
    try{
        while(!isDone()){
            done.await(timeout);
            long cur = System.nanoTime();
            if(isDone() || cur-start  > timeout){
                break;
            }
        }
    }finally{
        lock.unlock();
    }
    if(!isDone())
        return new TimeoutException();
    return returnFromResponse(;)
}
boolean isDone(){
    return response != null;
}
//RPC结果返回时调用
private void doReceived(Response res){
    lock.lock();
    try{
        response = res; //先设置response 再通知
        if(done != null){
            do.signal(); //尽量用signalAll
        }
    }finally{
        lock.unlock();
    }
}
```
## 16 semaphore 实现一个限流器
### 信号量模型   
一个计数器、一个等待队列、三个原子操作的方法。
* init 设置计数器的初始值
* down()计数器值减1，如果此时计时器值小于0，当前线程T1阻塞，否则当前线程T1可以继续执行----P操作---对应acquire()
* up()计数器值加1，如果此时计数器值<=0，则唤醒等待队列中的一个线程T2，并将T2从等待队列中移除-----V操作---对应release()
```java
class Semaphore{
    int count;
    Queue queue;
    Semaphore(int c){
        this.count = c;
    }
    void down(){
        this.count --;
        if(this.count <0){
            //将当前线程插入到等待队列
            //阻塞当前线程
        }
    }
    void up(){
        this.count++;
        if(this.count <=0){
            //移除等待队列中的某个线程T
            //唤醒线程T
        }
    }
}
```
### 信号量的使用
进入临界去前down,出来临界区up()
```java
static int count;
static final Semaphore s = new Semaphore(1);
static void addOne(){
    s.acquire();
    try{
        count += 1;
    }finally{
        s.release();
    }
}
```
两个线程T1，T2同时并发addOne()由于acquire的原子性，T1将s减1为0，之后T2将s减1为-1；对于T1，信号量里面的计数器值为0，大于等于0，T1会继续执行，T2,信号量里面的计数器值为-1，阻塞当前线程并将T2加入到阻塞队列。T1 release之后，s=0，小于等于0，等待队列中的T2会被唤醒。T2在T1执行完临界区的代码后才进入临界区，从而保证了互斥。
### 限流器
Seｍaphore允许多个线程访问同一个临界区   
限流器--不允许多于N个的线程同时进入临界区   
对象池---一次性创建N个对象，之后所有的线程重复利用这N个对象，对象在被释放前，不允许被其他线程使用
```java
class ObjPool<T, R>{
     List<T> pool;
    Semaphore sem;
    ObjPool(int size, T t){
        pool = new Vector<T>(){};
        for( int i = 0 ; i < size; i++){
            pool.add(t);
        }
        sem = new Semaphore(size);
    }
    //利用线程池中对象，调用func
    R exec(Function<T, R> func){
        T t = null;
        sem.acquire();
        try{
            t = pool.remove(0);
            return func.apply(t);
        }finally{
            pool.add(t);
            sem.release();
        }
    }
}
ObjPool<Long, String> pool = new ObjPool<Long, String>(10,2);
pool.exec(t -> {
    System.out.println(t);
    return t.toString();
})
```
如果把vector换成Arraylist（非线程安全），那么临界区内的多个remove和多个add操作不能保证线程安全。
## 17 readwritelock 实现完备的缓存
缓存提升性能的条件---缓存的数据一定读多写少。    
### 读写锁
* 允许多个线程同时读共享变量
* 只允许一个线程写共享变量
* 如果一个线程正在写，那么禁止读线程读共享变量。
### 缓存实现
```java
class Cache<K, V>{
    final Map<K, V> m = new HashMap<>();
    final ReadWriteLock rwl = new ReentrantLock();
    final Lock r = rwl.readLock();
    final Lock w = rwl.writeLock();
    V get(K key){
        r.lock();
        try{
            return m.get(key);
        }finally{
            r.unlock();
        }
    }
    V put(String key, Data v){
        w.lock();
        try{
            return m.put(key, v);
        }finally{
            w.unlock();
        }
    }
}
```
缓存首先要解决的是缓存数据的初始化问题，可以使用一次性加载或者按需加载方式。   
* 如果数据量不大，那么可以采用一次性加载方式
* 否则，当应用查询缓存，并且数据不在缓存时，才触发加载源头相关数据进缓存的操作。
### 实现缓存的按需加载
```java
class Cache<K, V>{
    final Map<K, V> m = new HashMap<>();
    final ReadWriteLock rwl = new ReentrantLock();
    final Lock r = rwl.readLock();
    final Lock w = rwl.writeLock();

    V get(K key){
        V v = null;
        r.lock();
        try{
            v = m.get(key);
        }finally{
            r.unlock();
        }
        if( v != null){
            return v;
        }
        w.lock();////@5
        try{
            //再次验证， 其他线程可能已经查询过数据库
            v = m.get(key);
            if(v == null){
                //查询数据库
                //v=???;
                m.put(key, v);
            }
        }finally{
            w.unlock();
        }
        return v;
    }
}
```
在未查询到v时，在查询数据库之前需要二次验证。
假设3个线程同时执行，开始缓存为空，那么T1 T2 T3同时调用get()方法，同时执行到@5处，只有T1获得锁，T2 T3进入到等待队列中。T1查询过后，写入缓存。T2 T3再次获得锁后需要再次验证一下～
### 读写锁的升级与降级
```java
r.lock()
try{
    v = m.get(key);
    if(v == null){
        w.lock();
        try{

        }finally{
            w.unlock();
        }
    }
}finally{
    r.unlock();
}
```
这种在读锁的lock和unlock期间，又存在写锁的lock和unlock的情况，为读锁升级为写锁。ReadWriteLock不支持这种升级，支持锁的降级。
```java
r.lock();
if(!cacheValid){
    r.unlock();
    w.lock();
    try{
        if(!cacheValid){   
        }
        r.lock();//释放写锁前，降级为读锁
    }finally{
        w.unlock();
    }
}
try{
    use(data);//此处仍持有读锁
}finally{
    r.unlock();
}
```
### 总结
* 只有写锁支持条件变量
* 读写锁也支持`tryLock() lockInterruptibly()`等方法
* 缓存同步问题：保证缓存数据和源头数据的一致性
  * 超时机制，超过一定时间，访问缓存中的数据触发重新从源头加载进缓存。
## 18 StampedLock 乐观、悲观锁
在读多写少的环境中，比读写锁更快。相较于读写锁的读锁和写锁，StampedLock有写锁、悲观读、乐观读三种模式。 写锁和悲观读类似写锁和读锁，但是StampedLock中写锁和悲观读锁加锁成功后返回stamp对象，解锁时需要传入该参数。
```java
final StampedLock s1 = new StampedLock();
long stamp = s1.readLock();
try{

}finally{
    s1.unlockRead(stamp);
}
try{

}finally{
    s1.unlockWrite(stamp);
}
``` 
同时乐观读方式允许多个线程读的同时，仅一个线程获取写锁。如果在乐观读操作期间，存在写操作，把乐观读升级为悲观读，之后再次读取共享变量。
```java
class Point{
    private int x, y;
    final StampedLock s1 = new StampedLock();
    int distanceFromOrigin() {
        long stamp = s1.tryOptimisticRead();
        int curx =x, cury = y;
        if(!s1.validate(stamp)){//判断是否存在写操作
            stamp = s1.readLock();//升级为悲观读
            try{
                curx =x ;
                cury = y;
            }finally{
                s1.unlockRead(stamp);//释放悲观读锁
            }
        }
        return Math.sqrt(curx * curx + cury*cury);
    }
}
```
### 注意事项
* StampedLock 为readwritelock的子集
* 不支持重入
* 悲观读锁，写锁不支持条件变量
* 如果线程阻塞在StampedLock的readLock()或者writeLock()上时，如果调用阻塞线程的interrupt方法，导致CPU飙升,如果需要支持中断，使用可中断的readLockInterruptibly和writeLockInterruptibly
### 总结 模板
```java
final StampedLock s1 = new StampedLock();
long stamp = s1.tryOptimisticRead();

if(!s1.validate(stamp)){
    stamp = s1.readLock();//升级为悲观锁
    try{
            //再次获取变量
    }finally{
        s1.unlockRead(stamp);
    }
}
```
```java
long stamp = s1.writeLock();
try{

}finally{
    s1.unlockWrite(stamp);
}
```
## 19 CountDownLatch 和CyclicBarrier 线程步调一致
### 背景
```java
while(存在未对账订单){
    pos = getPOrder();
    dos = getDOrders();
    diff = check(pos, dos);
    save(diff);
}
```
### 多线程
```java
while(存在未对账订单){
    Thread t1 = new Thread(()->{
        pos = getPOrder();
    });
    t1.start();
    Thread t2 = new Thread(()->{
        dos = getDOrder();
    });
}
t2.start();
t1.join();
t2.join();
//........
```
### 线程池 配合CountDownLatch
省去创建线程所需的时间，CountDownLatch解决多个线程间的等待
```java
Executor executor = Executors.newFixedThreadPool(2);
while(存在未对账订单){
    CountDownLatch latch = new CountDownLatch(2);// 表示需要等待两个线程
    executor.execute(()->{
        pos = getPOrder();
        latch.countDown();
    });
    executor.execute(()->{
        dos = getDOrder();
        latch.countDown();
    });
    latch.await();//等待两个查询结束
    //...................
}
```
### 生产者消费者
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2020/09/19/15-07-13-9c2869be72f32eeb17cb08bc87b7ae55-20200919150713-80f598.png)   
一个线程T1执行订单的查询，一个T2执行派送单的查询，当线程T1T2各自生产完成一条数据时，通知线程T3执行对账操作   
* T1 T2 步调一致
* 结束后能够通知到T3
* 计数器初始化2，T1、T2结束后减1，计数器大于0则线程T1、T2等待。如果计数器为0，则通知线程T3，并唤醒等待的线程T1、T2。同时将计数器重置2.
```java
Vector<P> pos;
Vector<D> dos;
Executor executor = Executors.newFixedThreadPool(1);
final CyclicBarrier barrier = new CyclicBarrier(2, ()->{
    executor.execute(()->check());
});
void check(){
    P p = pos.remove(0);
    D d = dos.remove(0);
    //...........
}
void checkAll(){
    Thread t1 = new Thread(()->{
        while(存在未对账订单){
            pos.add(getPOrders());
            //等待
            barrier.await();
        }
    });
    t1.start();

    Thread t2 = new Thread(()->{
        while(存在未对账订单){
            dos.add(getDOrders());
            //等待
            barrier.await();
        }
    });
    t2.start();
}
```
### 总结
* CountDownLatch 解决一个线程等待多个线程场景，且计数器不能循环利用，一旦计数器为0，再有线程调用`await`，直接通过
* CyclicBarrier 一组线程之间的等待，计数器可循环利用。
* CyclicBarrier 是在同步调用回调函数后才唤醒等待的线程，如果在回调函数直接执行check()，在执行check时，不能同时执行getPOrder和getDOrder，性能未提升。
* CyclicBarrier 执行回调函数的是将CyclicBarrier减到0的那个线程。
* 看到回调函数时，想想执行回调函数的是哪个？

## 并发容器
通过包装借助synchronized将非线程安全容器变为线程安全，同样`Vector Stack Hashtable`，虽然不是基于包装类，但同样基于synchronized，同样遍历时需要加锁。
遍历时存在竟态条件
* it.hasNext
* it.Next

```java
       List list = Collections.synchronizedList(new ArrayList());
        Set set = Collections.synchronizedSet(new HashSet());
        Map map = Collections.synchronizedMap(new HashMap());
// 非线程安全
        Iterator it = list.iterator();
        while (it.hasNext()) {
            if (it.next() instanceof String) {
                fun((String) it.next());
            }
        }
//        线程安全
        synchronized (list){
            Iterator its = list.iterator();
            while (its.hasNext()) {
                if (its.next() instanceof String) {
                    fun((String) its.next());
                }
            }
        }
```
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2021/02/23/19-18-04-c7ff57c4cb027c3e7269ae90cdae3de4-20210223191804-75f97d.png)
### List
CopyOnWriteArrayList 适用于写非常少，且容忍读和写短暂不一致的情况，效果读操作完全无锁，迭代器只读，不支持增删改
* 内部维护一个数组，变量array指向它，所有读操作基于此array
* 如果遍历时，出现写操作，将array指向的数组复制一份，然后在复制的数组上执行增加元素的操作，执行完后在将变量指向新的数组
* 读写并行，读基于旧的array，写基于新的数组
### Map
ConcurrentHashMap key无序，ConcurrentSkipListMap   
|diff|ConcurrentHashMap|ConcurrentSkipListMap|
|:-:|:-:|:-:|
|key|key无序|key有序|
|O(n)||O(logn)和并发数无关|
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2021/02/23/19-28-11-8a18b9737ee722513cb5a2efc3f3b1ad-20210223192811-6eabfe.png)
### Set 
`CopyOnWriteArraySet`&`ConcurrentSkipListSet`
### Queue 
* Blocking 阻塞，出队列，空时阻塞，入队列，满时阻塞
* Queue 单端
* Deque 双端
#### Queue & Blocking
* ArrayBlockingQueue 数组队列
* LinkedBlockingQueue  链表队列
* SynchronousQueue 无队列
* LinkedTransferQueue 链表且无队列
* PriorityBlockingQueue 按照优先级出队 
* DelayQueue 支持延迟出队
#### Deque &Blocking
* LinkedBlockingDeque
#### Queue & NonBlocking
* ConcurrentLinkedQueue
#### Deque & NonBlocking
* ConcurrentLinkedDeque
  
#### Conclusion
* 注意是否有边界，仅`ArrayBlockingQueue` `LinkedBlockingQueue`支持有界
* 单个操作安全，组合操作不一定安全
## 原子类
同一个变量的原子更新问题，多个变量还是互斥锁
```java
 static AtomicLong count = new AtomicLong(0);

    static void add10k() {
        int idx = 0;
        while (idx++ < 1000) {
            count.getAndIncrement();
        }
    }
```
```java
  //模拟CAS
    static class SimulateCAS {
        int count;

        synchronized int cas(int except, int newValue) {
            int curValue = count;
            if (curValue == except) {
                count = newValue;
            }
            return curValue;
        }

        void addOne() {
            int newValue;
            do {
                newValue = count + 1;
            } while (count != cas(count, newValue));
        }
    }
```
```java
//确定共享变量的内存地址 
    public final long getAndIncrement() {
        return unsafe.getAndAddLong(this, valueOffset, 1L);
   }
   //
     public final long getAndAddLong(Object var1, long var2, long var4) {
        long var6;
        do {
            //从内存中获取共享变量
            var6 = this.getLongVolatile(var1, var2);
            // 循环判断变量值 是否预期
        } while(!this.compareAndSwapLong(var1, var2, var6, var6 + var4));

        return var6;
    }
```
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2021/02/23/20-31-23-c7f354fb7ce9aa1fe07d0d283496df53-20210223203123-5ef820.png)
### 原子化的基本类型数据
```java
getAndIncrement() i++
getAndDecrement() i--
incrementAndGet() ++i
decrementAndGet() --i

getAndAdd(delta) +=delta 返回之前的值
addAndAdd(delta) +=delta 返回之后的值

compareAndSet(expect,update)

getAndUpdate(func) 新值通过func计算
updateAndGet(func)
getAndAcculate(x,func)
accumulateAndGet(x,func)
```
### 原子化的对象引用类型
实现对象应用的原子化更新
* `AtomicReference`
* `AtomicStampedReference` 通过int stamp解决ABA
* `AtomicMarkableReference` 通过boolean expectedMark 解决ABA
注意ABA问题
### 原子化数组
* `AtomicIntegerArray`
* `AtomicLongArray`
* `AtomicReferenceArray`

```java
  AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(5);
        atomicIntegerArray.addAndGet(0,2);
        System.out.println(atomicIntegerArray.get(0));
```
### 原子化对象属性更新
利用发射原子化更新对象的属性，但对象属性必须volatile
* `AtomicIntegerFieldUpdater`
* `AtomicLongFieldUpdater`
* `AtomicReferenceFieldUpdater`

```java
    static class Student{
        volatile int age = 0;
    }
    public static void main(String[] args) {
        AtomicIntegerFieldUpdater<Student> atomicIntegerFieldUpdater = AtomicIntegerFieldUpdater.newUpdater(Student.class,"age");
        Student student = new Student();
        student.age = 10;
        int ret = atomicIntegerFieldUpdater.get(student);
        System.out.println(ret);
    }
```
### 原子化累加器
仅仅执行累加操作，比原子化的基本数据类型 更快，不支持compareAndSet
* `DoubleAccumulator`
* `DoubleAdder`
* `LongAccumulator`
* `LongAdder`

## 线程池
