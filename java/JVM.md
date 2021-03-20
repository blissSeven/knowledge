# JVM
## JAVA
### Java ：
面向对象、静态类型、编译执行，有VM/GC和运行时(上下文环境-JVM)、跨平台的高级语言
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2021/03/20/16-19-03-608e9b530598085975f5735524a8cfb0-20210320161903-b004ec.png)
### 字节码
由单字节byte的指令组成，理论最多256个字节码opcode,实际仅200左右
* 栈操作、包括与局部变量交互的指令
* 流程控制
* 对象操作
* 算术运算以及类型转换
```java
package JVM;

public class HelloByteCode {
    public static void main(String[] args) {
        HelloByteCode obj = new HelloByteCode();
    }
}

```
`javac JVM/HelloByteCode.java`   
`javap -c ./HelloByteCode.class`
```java
Compiled from "HelloByteCode.java"
public class JVM.HelloByteCode {
  public JVM.HelloByteCode();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: new           #2                  // class JVM/HelloByteCode
       3: dup
       4: invokespecial #3                  // Method "<init>":()V
       7: astore_1
       8: return
}
```
计算在栈上，变量在变量表里，将变量表里的变量load到栈里去计算后在传到变量表里    
`javap -c -verbose ./HelloByteCode.class`  
```java
Classfile /home/bliss/miwork/project/learn-java/Think-in-java/src/main/java/JVM/HelloByteCode.class
  Last modified 2021-3-20; size 292 bytes
  MD5 checksum 5130a3cf5db01e579a2af1970dbe5e78
  Compiled from "HelloByteCode.java"
public class JVM.HelloByteCode
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #4.#13         // java/lang/Object."<init>":()V
   #2 = Class              #14            // JVM/HelloByteCode
   #3 = Methodref          #2.#13         // JVM/HelloByteCode."<init>":()V
   #4 = Class              #15            // java/lang/Object
   #5 = Utf8               <init>
   #6 = Utf8               ()V
   #7 = Utf8               Code
   #8 = Utf8               LineNumberTable
   #9 = Utf8               main
  #10 = Utf8               ([Ljava/lang/String;)V
  #11 = Utf8               SourceFile
  #12 = Utf8               HelloByteCode.java
  #13 = NameAndType        #5:#6          // "<init>":()V
  #14 = Utf8               JVM/HelloByteCode
  #15 = Utf8               java/lang/Object
{
  public JVM.HelloByteCode();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1 //栈需要2 哥深度，局部变量表槽位2哥
         0: new           #2                  // class JVM/HelloByteCode  //第0 位
         3: dup //第3位
         4: invokespecial #3                  // Method "<init>":()V
         7: astore_1
         8: return
      LineNumberTable:
        line 5: 0
        line 6: 8
}
SourceFile: "HelloByteCode.java"

```
JVM基于栈的计算机，每个线程有自己的线程栈，用于存储栈帧，每一次方法调用，JVM都会自动创建一个栈帧。栈帧有操作数栈，局部变量数组以及一个Class引用组成。Class引用指向当前方法在运行时常亮池中对应的Class。
```java
public class MovingAverage {
    private int count = 0;
    private double sum = 0.0D;

    public void submit(double value) {
        this.count++;
        this.sum += value;
    }

    public double getAvg() {
        if (0 == this.count) {
            return sum;
        }
        return this.sum / this.count;
    }
}

public class LocalVariableTest {
    public static void main(String[] args) {
        MovingAverage ma = new MovingAverage();
        int num1 = 1;
        int num2 = 2;
        ma.submit(num1);
        ma.submit(num2);
        double avg = ma.getAvg();
    }
}
```
```java
Classfile /home/bliss/miwork/project/learn-java/Think-in-java/src/main/java/JVM/LocalVariableTest.class
  Last modified 2021-3-20; size 416 bytes
  MD5 checksum 5993218b6de2c160a5f7e55b80cc221c
  Compiled from "LocalVariableTest.java"
public class JVM.LocalVariableTest
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #7.#16         // java/lang/Object."<init>":()V
   #2 = Class              #17            // JVM/MovingAverage
   #3 = Methodref          #2.#16         // JVM/MovingAverage."<init>":()V
   #4 = Methodref          #2.#18         // JVM/MovingAverage.submit:(D)V
   #5 = Methodref          #2.#19         // JVM/MovingAverage.getAvg:()D
   #6 = Class              #20            // JVM/LocalVariableTest
   #7 = Class              #21            // java/lang/Object
   #8 = Utf8               <init>
   #9 = Utf8               ()V
  #10 = Utf8               Code
  #11 = Utf8               LineNumberTable
  #12 = Utf8               main
  #13 = Utf8               ([Ljava/lang/String;)V
  #14 = Utf8               SourceFile
  #15 = Utf8               LocalVariableTest.java
  #16 = NameAndType        #8:#9          // "<init>":()V
  #17 = Utf8               JVM/MovingAverage
  #18 = NameAndType        #22:#23        // submit:(D)V
  #19 = NameAndType        #24:#25        // getAvg:()D
  #20 = Utf8               JVM/LocalVariableTest
  #21 = Utf8               java/lang/Object
  #22 = Utf8               submit
  #23 = Utf8               (D)V
  #24 = Utf8               getAvg
  #25 = Utf8               ()D
{
  public JVM.LocalVariableTest();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=6, args_size=1
         0: new           #2                  // class JVM/MovingAverage
         3: dup
         4: invokespecial #3                  // Method JVM/MovingAverage."<init>":()V
         7: astore_1
         8: iconst_1
         9: istore_2
        10: iconst_2
        11: istore_3
        12: aload_1
        13: iload_2
        14: i2d
        15: invokevirtual #4                  // Method JVM/MovingAverage.submit:(D)V
        18: aload_1
        19: iload_3
        20: i2d
        21: invokevirtual #4                  // Method JVM/MovingAverage.submit:(D)V
        24: aload_1
        25: invokevirtual #5                  // Method JVM/MovingAverage.getAvg:()D
        28: dstore        4
        30: return
      LineNumberTable:
        line 5: 0
        line 6: 8
        line 7: 10
        line 8: 12
        line 9: 18
        line 10: 24
        line 11: 30
}
SourceFile: "LocalVariableTest.java"

```
astore_1 1 代表变量池的slot为1    
iconst_1 1代表常量1
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2021/03/20/17-10-04-7e688227c306ddee5980fc2c6543c59d-20210320171003-50142d.png)
jvm 可以运算的类型有int、long、float、double提供了向其他类型的转换    
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2021/03/20/17-19-59-5c8988cf45a922cdd660addf20fc9ecc-20210320171959-5ce161.png)   
JVM 循环控制   
if_icompge、iinc 、 goto
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2021/03/20/17-24-07-5703aa8d7493b2d451a0b6a763839428-20210320172407-354331.png)
方法调用指令   
* invokestatic 调用静态方法，最快
* invokespecial 构造函数，也可以调用同类的private方法以及可见的超类方法
* invokevirtual 如果是具体类型的目标对象，调用公共、受保护、package级别的私有方法
* invokeinterface 通过接口引用来调用方法
* invokedynamic JDK7 新增，对动态类型语言、lambda 支持

![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2021/03/20/17-38-11-f93e5fc0dd934158742d6ab330f97bc9-20210320173810-8fb811.png)  

## 类加载器
* 加载 找class文件
* 验证 验证格式、依赖
* 准备 静态字段、方法表
* 解析 符号解析为引用
* 初始化 构造器、静态变量赋值、静态代码块
* 使用
* 卸载
类加载时机   
* 虚拟机启动时，初始化用户制定的main class
* new 一个类
* 调用静态方法的指令，
* 子类初始话触发父类初始化
* 一个接口定义default方法，当直接实现或间接实现该接口的类的初始化时，触发该接口初始化
* 反射API对某个类进行反射调用，初始化该类
* 初次调用MethodHandle实例，初始化该MethodHandle指向的方法所在的类   
不会初始化(可能加载)
* 通过子类引用父类的静态字段，触发父类初始化，不会触发子类初始化
* 定义对象数组，不会触发该类初始化
* 常量在编译期间会存入调用类的常量池中，本质上没有直接引用定义常量的类，不会触发定义常量所在的类
* 通过类名获取Class对象，不会触发该对象
* 通过Class.forName加载类时，指定参数initialize=false
* 通过ClassLoder默认loadClass方法，只加载，不初始化

类加载器
* 启动类加载器BootStramClassLoader 加载核心类 \lib\rt.jar
* 扩展类加载器ExClassLoader jdk扩展目录
* 应用类加载器AppClassLoader 用户类  
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2021/03/20/17-58-09-c29f476ac8cbadea550d80eca18ba9c2-20210320175809-3cbfed.png)    
  特点
* 双亲委托--依次递归看父类加载器是否有，
* 负责依赖--加载该类同时，加载该类的依赖类
* 缓存加载--类只加载依次，之后缓存
自定义类加载器实现模块化   

```java
public class ClassLoaderTest {
    public static void main(String[] args) {
//        System.out.println(System.getProperty("sun.boot.class.path"));
        URL[] urls = sun.misc.Launcher.getBootstrapClassPath().getURLs(); //sun.misc.
        for (URL url : urls) {
            System.out.println(url.toExternalForm());
        }
        printClassLoader("扩展类加载器",ClassLoaderTest.class.getClassLoader().getParent());
        printClassLoader("应用类加载器",ClassLoaderTest.class.getClassLoader());

    }

    public static void printClassLoader(String name, ClassLoader CL) {
        if (CL != null) {
            System.out.println(name + " => " + CL.toString());
            printURLForClassLoader(CL);
        }else{
            System.out.println(name+" => null");
        }
    }

    public static void printURLForClassLoader(ClassLoader CL) {
        Object ucp = insightField(CL, "ucp");
        Object path = insightField(ucp, "path");
        ArrayList ps = (ArrayList) path;
        for (Object p : ps) {
            System.out.println("-->" + p.toString());
        }
    }

    private static Object insightField(Object obj, String fName) {
        try {
            Field f = null;
            if (obj instanceof URLClassLoader) {
                f = URLClassLoader.class.getDeclaredField(fName);
            } else {
                f = obj.getClass().getDeclaredField(fName);
            }
            f.setAccessible(true);
            return f.get(obj);
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
        return null;
    }

}
```
```java
file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/resources.jar
file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/rt.jar
file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/sunrsasign.jar
file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/jsse.jar
file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/jce.jar
file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/charsets.jar
file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/jfr.jar
file:/usr/lib/jdk/jdk1.8.0_261/jre/classes
扩展类加载器 => sun.misc.Launcher$ExtClassLoader@726f3b58
-->file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/ext/sunpkcs11.jar
-->file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/ext/localedata.jar
-->file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/ext/cldrdata.jar
-->file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/ext/nashorn.jar
-->file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/ext/sunjce_provider.jar
-->file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/ext/dnsns.jar
-->file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/ext/zipfs.jar
-->file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/ext/jaccess.jar
-->file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/ext/jfxrt.jar
-->file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/ext/sunec.jar
应用类加载器 => sun.misc.Launcher$AppClassLoader@18b4aac2
-->file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/charsets.jar
-->file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/deploy.jar
-->file:/usr/lib/jdk/jdk1.8.0_261/jre/lib/ext/cldrdata.jar
```

  