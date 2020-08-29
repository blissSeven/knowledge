# Java Overview
- [Java Overview](#java-overview)
  - [语法](#语法)
    - [变量](#变量)
    - [修饰符](#修饰符)
    - [运算符](#运算符)
  - [常用类](#常用类)
    - [Number&Math类](#numbermath类)
    - [String](#string)
    - [StringBuffer/Builder](#stringbufferbuilder)
    - [StringJoiner](#stringjoiner)
    - [包装类型](#包装类型)
    - [java bean](#java-bean)
    - [枚举 #archor](#枚举-archor)
    - [BigInteger](#biginteger)
    - [BigDecimal](#bigdecimal)
    - [数组](#数组)
    - [正则表达式](#正则表达式)
  - [面向对象](#面向对象)
    - [方法](#方法)
    - [继承](#继承)
    - [多态](#多态)
    - [接口](#接口)
    - [包](#包)
    - [文件](#文件)
  - [异常](#异常)
    - [反射](#反射)
      - [Class 类](#class-类)
      - [访问字段](#访问字段)
      - [调用方法](#调用方法)
      - [获取继承关系](#获取继承关系)
      - [动态代理](#动态代理)
    - [重写（Override） VS 重载（Overload）](#重写override-vs-重载overload)
## 语法  
### 变量
*  $a 合法标识符
* java 数据类型  
   * 内置  
      * byte 默认0 	
      * int 默认0   
      * long 默认0L/0l
      * float 0.0f
      * double 0.0d
      * boolean false
      * char 16bit Unicode字符，最小0 \u0000，最大65535 \uffff
      * Byte.SIZE Byte.MIN_VALUE Byte.MAX_VALUE
   * 引用类型
* java 变量类型
  * 局部变量没有默认值，声明后，必须初始化后才能用
  * 成员/实例变量
     * public 能被所有class访问
     * private 只能被该类访问
     * protectd 本类访问+子类访问+同一个package中类访问
     * default 同一个package中类访问，即使继承了父类，不在一个package也不能访问
  * 类变量
     * 类变量多声明为常量,public/private,final,static类变量
### 修饰符
   * public类、方法、构造方法、接口能被任何其他类访问，若相互访问的public类不在同一包中，需导入相应的包
   * private方法、变量、构造方法只能被所属类访问，且类和接口不能private(内部类除外)
   * protected可修饰成员、构造方法、方法成员，不能修饰类(内部类除外),接口及接口的成员变量/成员方法不能protected
      * 子类基类同一包中
      * 子类基类不在同一包，子类可以访问从基类继承类的protected方法，不能访问基类实例的protected方法
   * 继承规则
      * 父类声明public的方法，子类必须public
      * 父类protected，子类protected/public，不能private
      * 父类private,不能被继承
   * 非访问修饰符
      * static 修饰类方法/类变量
      * final修饰类、方法、变量，final类不能被继承，final方法不能被继承类重定义，final变量为常量
        * final变量，必须显示指定初始值 ，或者在定义时初始化，或者在类的构造函数中初始化
        * final 和static一起使用创建类常量
      * abstract 创建抽象类，抽象方法，
        * 一个类不能同时abstract && final
    	 * 包含一个抽象方法（方法声明为abstract），则该类为抽象类
    	 * 抽象方法不能final/static
    	 * 继承抽象类的子类必须实现父类所有抽象方法，除非子类也是抽象类
    	 * VS interface
      	 * interface 没有构造函数  abstract 类有构造函数
      	 * interface内部任何方法不允许实现，abstract类允许有一般非abstract的方法，有具体实现
      	 * interface内部没有super，this变量，abstract类有
      	 * interface成员变量一定常数，abstract类成员变量为一般变量
      	 * interface所有成员public，abstract类可以任何等级
      	 * interfact成员变量默认public (允许继承)static（和实例无关，类本身） final（接口定义的常亮不能被修改）
      * synchronized 
        * 该方法同一时间只能被一个线程访问
      * transient
        * 包含在定义变量的语句中，预处理类和变量的数据类型。不会被持久化
      * volatile
        * 修饰变量在每次被线程访问时，都强制从共享内存中重新读取变量值。当成员变量发生变化时，强制线程将变化值写回内存，两个不同线程总是看到某个成员变量的同一个值
### 运算符
  * instanceof
    * 检查对象是否是一个特定类型（类类型/接口类型）
    * boolean result = name instanceof String;
* switch-case
  * switch必须为byte、short、int、char、String
  * case必须为字符常量、字面量
## 常用类
###  Number&Math类
![](https://raw.githubusercontent.com/BlissSeven/image/master/spring/2020/08/14/10-57-08-010cc66ea9ab25f54c25e27b51541cb7-OOP_WrapperClass-5d96cc.png)
  * xxxValue 
    * byteValue 以byte形式返回指定数值 
  * compareTo
  * equals
  * valueof 给定参数的原生Number对象值，参数可以原生数据类型或String,为静态方法
    * static Integer.valueof(int i)
    * static Integer.valueOf(String s)
    * static Integer.valueOf(String s,int radix) radix指定使用的进制数
* Character
  * char的封装类型
    * isLetter() 是否是字母
    * isDigit()是否数字
    * isWhitespace()是否空白字符
    * isUpperCase()是否大写字母
    * toString()返回字符字符串形式
###  String
* 常用函数
  * char charAt(int index)
  * int compareTo(Obejct o)
  * int compareToIgnoreCase(String str)
  * String concat(String str)
  * boolean contentEquals(StringBuffer sb) 判断二者内容相同
  * static String copyValueOf(char[] data) 将char数组转为String
  * staic String copyValueOf(char[] data,int offset,int count)
  * boolean endsWith(String suffix)
  * boolean equals(Obejct s) 只能比较两个String内容是否相同
  * int hashCode()
  * String intern() 返回字符串对象的规范化表示
    * 检查字符串池中是否有该字符串，如果存在则返回池中的该字符串的引用。否则将该字符串添加进池并返回引用
  * new String(byte[]) //将byte数组转为String
  * String replace(CharSequence target, CharSequence replacement) //替换字符串并返回新的字符串
  * String[] split(String regex)//根据正则切分
    ```java
    str="A,B,C,";
        String[] ss=str.split("\\,");
        System.out.println(Arrays.toString(ss));
    ```
  * trim()去除字符串首尾空白
  * strip() 去除字符串首尾空白+类似中文空格字符\u3000也会去除(trim 不会！！)
  * boolean isEmpty() 
  * boolean isBlank()
  *  static String  join(CharSequence delimiter, CharSequence... elements) //根据间隔符将序列Join
  *  static String valueOf(Object obj)//将Object 转为String
  *  Integer.parseInt（）Boolean.parseBoolean("true") //将String转为其他
  *  char[] toCharArray()    new String(char[])//转为char数组
  *  编码转换
     *  byte[] b2="Hello".getBytes("UTF-8")
     *  String s1=new String(b2,"GBK")//将byte[] 转为GBK的String
     *  Java的String和char在内存中以Unicode编码
### StringBuffer/Builder
  * 对字符串多次修改，且不产生新的对象
  * String Buffer 线程安全，慢
  * StringBuilder 线程不安全，快
  * 对于普通的String +操作，java在编译时自动把多个连续的+操作编码为StringConcatFactory操作，运行期，StringConcatFactory 自动把字符串连接操作优化为数组复制或者StringBuilder操作
  * public StringBuffer append(String s)
  * public StringBuffer reverse()
  * public delete(int start,int end)
  * public insert(int offset,int i)在offset处，将int字符串插进去
  * replace(int start,int end,String str)
  * indexOf(String str) 返回第一次出现的子字符串的索引
  * indexOf(String str,int fromIndex)
  * int lastIndexOf()
  * CharSequence subSequence(int start,int end)
### StringJoiner
* 分隔符拼接数组
    ```java
        StringJoiner sj=new StringJoiner(",");
        sj.add("name");
        sj.add("hello");
        sj.add("world");
        System.out.println(sj.toString());//name,hello,world

         String[] names = {"Bob", "Alice", "Grace"};
        var sj = new StringJoiner(", ", "Hello ", "!"); //指定拼接头  、拼接尾
        for (String name : names) {
            sj.add(name);
        }
        System.out.println(sj.toString());//Hello Bob, Alice, Grace!
    ```
### 包装类型
* 将一个基本类型视为引用(引用类型)
* 引用可以null，基本类型不可以null
* auto boxing / unboxing自动装箱/拆箱
  * ``` int i=100; Integer n=Integer.valueof(i);``` 
  * ```Integer n=100;``` 自动调用Integer.valueOf   ----     int->Integer成为自动装箱
  * ``` int x=n;``` 自动调用Integer.intValue() ----  Integer->int
  * 装箱拆箱影响效率，当Integer为null时，拆箱引发NullPointerException
* 所有包装类型为不变类
  * 定义class时final
  * 每个字段final修饰，保证创建实例后无法修改字段
  * 为了保证不变类的比较，还需要正确覆写equals()和hashCode()方法
    ```java
    public final class Integer{
      private final int value;
    }
    ```
* 包装类型比大小 equals()
  * 为了节省内存，Integer.valueOf()对于较小的数，始终返回相同的实例
* 创建Integer 
  * ```Integer n=new Integer(100)```  总是创建新的Integer实例
  * ```Integer n=Integer.valueOf(100)``` 尽可能返回缓存的实例，优选！！！
    * 能创建新对象的静态方法为静态工厂方法，优选！！！
* 进制转换
  * ```int x1=Integer.parseInt("1000")```
  * ```int x2=Integer.parseInt("100",16)```//16进制
  * ```Integer.toString(100)```
  * ```Integer.toString(100, 36)```//36进制
  * ```Integer.toOctalString(100)```//8进制
  * ```Integer.toBinaryString(100)```//2进制字符串
  * ```Integer.MAX_VALUE;```
  * ```Long.SIZE```
  * ```Long.BYTES```
  * ```Byte.toUnsignedInt(x)```  把一个负的byte按无符号整型转换为int
  * 把一个short按unsigned转换为int，把一个int按unsigned转换为long
### java bean
* 如果class定义满足,则称为JavaBean
  * private实例字段
  * public 读写字段
    * 读写方法满足
      * ```public Type getXyz()```
      * ```public void setXyz(Type value)```
###  枚举 #archor
enum是一个class，每个枚举的值都是class实例
  * 优势
    * 带有类型信息
    * 不会引用非枚举的值
    * 不同枚举类型不能比较/赋值  安全
  * 同其他class 区别
    * 枚举类可以有自己构造函数，默认private
    * 只能定义出enum的实例，而无法通过new操作符创建enum的实例
    * enum继承自java.lang.Enum，且无法被继承
    * 可以将enum类型用于switch语句
    * 枚举类的字段也可以是非final类型，即可以在运行期修改，但是不推荐这样做！
  * 静态方法values() 返回当前类中所有值
  * 成员方法ordinal() 找到枚举常量的索引
  * 成员方法valuesOf("SUN") 返回指定字符串的枚举常量
   ```java
   public class EnumTest {
    enum Color{
        RED,BLUE,GREEN;
       private Color(){
            System.out.println("constructor called for "+this.toString());
        }
        public void colorinfo(){
            System.out.println("Universe color");
        }
    }

    public static void main(String[] args) {
        Color cr=Color.BLUE;
        for(Color c:Color.values()){
            System.out.println(c);
        }
        System.out.println(cr.ordinal());//=1 返回索引
        Color red=Color.valueOf("RED");
        System.out.println(red);//=RED返回指定字符串值的枚举常量
    }
   }
   ```
   ```java
   public class Main {
    public static void main(String[] args) {
        Weekday day = Weekday.SUN;
        if (day.dayValue == 6 || day.dayValue == 0) {
            System.out.println("Work at home!");
        } else {
            System.out.println("Work at office!");
        }
    }
}

enum Weekday {
    MON(1), TUE(2), WED(3), THU(4), FRI(5), SAT(6), SUN(0); //给每个枚举常量添加字 段 类似构造函数
    public final int dayValue;

    private Weekday(int dayValue) {
        this.dayValue = dayValue;
    }
}
   ```
* 对枚举常量调用toString()会返回和name()一样的字符串。但是，toString()可以被覆写，而name()则不行。
* 判断枚举常量的名字，要始终使用name()方法，绝不能调用toString(),因为可能被复写  :laughing:
* 覆写toString()的目的是在输出时更有可读性
   ```java
   enum Weekday {
      MON(1, "星期一"), TUE(2, "星期二"), WED(3, "星期三"), THU(4, "星期四"), FRI(5, "星期五"), SAT(6, "星期六"), SUN(0, "星期日");

      public final int dayValue;
      private final String chinese;
      private Weekday(int dayValue, String chinese) {
          this.dayValue = dayValue;
          this.chinese = chinese;
      }
      @Override
      public String toString() {
          return this.chinese;
      }
  }
   ```
### BigInteger
在Java中，由CPU原生提供的整型最大范围是64位long型整数。
java.math.BigInteger就是用来表示任意大小的整数。BigInteger内部用一个int[]数组来模拟一个非常大的整数，不变类，继承自Number         
* ```BigInteger bi = new BigInteger("1234567890");```
* 对BigInteger做运算的时候，只能使用实例方法, (没有相应的操作符重载 :laughing:)
   ```java
   BigInteger bi=new BigInteger("1234567890");
        BigInteger bi2=new BigInteger("1234567890");
        System.out.println(bi.add(bi2));
   ```
* 转换为有限类型
  * ```i.longValue()  byteValue()  intValue()  doubleValue()```
  * ```longValueExact()``` 超范围时，抛出异常ArithmeticException
### BigDecimal
表示任意大小且精度准确的浮点数
* scale() 表示小数位数
   ```java
   BigDecimal d1 = new BigDecimal("123.45");
   System.out.println(d1.scale()); // 2,两位小数
   ```
* stripTrailingZeros  将一个BigDecimal格式化为一个相等的，但去掉了末尾0的BigDecimal
* BigDecimal的scale()返回负数，例如，-2，表示这个数是个整数，并且末尾有2个0
* 对一个BigDecimal设置它的scale，如果精度比原始值低，那么按照指定的方法进行四舍五入或者直接截断
    ```
        BigDecimal d1 = new BigDecimal("123.456789");
        BigDecimal d2 = d1.setScale(4, RoundingMode.HALF_UP); // 四舍五入，123.4568
        BigDecimal d3 = d1.setScale(4, RoundingMode.DOWN); // 直接截断，123.4567
    ```
  * 对BigDecimal做加、减、乘时，精度不会丢失，但是做除法时，存在无法除尽的情况，这时，就必须指定精度以及如何进行截断：
     ```java
     BigDecimal d1 = new BigDecimal("123.456");
    BigDecimal d2 = new BigDecimal("23.456789");
    BigDecimal d3 = d1.divide(d2, 10, RoundingMode.HALF_UP); // 保留10位小数并四舍五入
    BigDecimal d4 = d1.divide(d2); // 报错：ArithmeticException，因为除不尽
     ```
* 对BigDecimal做除法的同时求余数：
    ```java
        BigDecimal n = new BigDecimal("12.345");
        BigDecimal m = new BigDecimal("0.12");
        BigDecimal[] dr = n.divideAndRemainder(m);
        System.out.println(dr[0]); // 102
        System.out.println(dr[1]); // 0.105
    ```
* 总是使用compareTo()比较两个BigDecimal的值，不要使用equals()！
* 使用equals()方法不但要求两个BigDecimal的值相等，还要求它们的scale()相等
### 数组
  * public static void fill(int[] a,int val) val初始化所有a
  * public static boolean equals(long[]a,long[] b) 相同顺序，相同数据，则相同
* Date
  * boolean before(Date date) 调用对象在date对象之后
  * long getTime() 返回毫秒数
  * String toString()
  * ```java
      //统计运行时间
      long start=Sytem.currentTimeMills()
      long end=Sytem.currentTimeMills();
    ```
   * Calender 用于设置/获取日期的特定部分
     * date=0 表示上一个月的最后一天，-1 表示上一个月的倒数第二天，一次类推月份
     * ```java
           Calendar c=Calender.getInstance()
           c1.set(2009, 6 - 1, 12);
           Calendar.DAY_OF_WEEK
           Calendar.HOUR_OF_DAY //24小时 小时
           Calendar.HOUR
           c1.add(Calendar.DATE, 10);
            c.set(2017,1,1);  2017 1 1
            c.set(2017,1,0);  2017 0 31
            c.set(2017,0,0); 2016 11 31
            c1.set(2017, 2, -10); 2017 1 18
        ```
### 正则表达式
  * Pattern 正则表达式的编译表示,接受正则表达式作为参数
    * Pattern m=Pattern.compile(regrex)
  * Matcher对输入字符串进行解释和匹配操作
    * 调用pattern对象的matcher方法获得Matcher对象
  * PatternSyntaxException 非强制异常类
  * 捕获组
    * 将多个字符当一个单独单元处理，通过对括号内的字符分组来创建
    * 通过从左至右计算开括号 进行编号
    * 通过matcher的groupCount查看分组个数。
      * group(0)代表整个表达式，所以不在count内
   ```java
         // 按指定模式在字符串查找
      String line = "This order was placed for QT3000! OK?";
      String pattern = "(\\D*)(\\d+)(.*)";

      // 创建 Pattern 对象
      Pattern r = Pattern.compile(pattern);

      // 现在创建 matcher 对象
      Matcher m = r.matcher(line);
      if (m.find( )) {
         System.out.println("Found value: " + m.group(0) );
         System.out.println("Found value: " + m.group(1) );
         System.out.println("Found value: " + m.group(2) );
         System.out.println("Found value: " + m.group(3) ); 
      } else {
         System.out.println("NO MATCH");
      }
       $ 匹配结束
       * 零次或多次匹配前面表达式
       + 1次或多次匹配前面表达式
       {n} n>=0 匹配n次
       {n,} n>=0至少匹配n次
       {n,m}   0<=n<=m 至少n次，至多m次
       ? 当此字符紧随其他限定符(*、+、？、{n}、{n,m}、{n,}) 表示匹配模式非贪心
       x|y 匹配x或y
       [xyz] 匹配包含的任一字符
       [a-z]
       [^a-z] 反向范围匹配
       \d 数字匹配
       \D 非数字匹配
       \n 换行符
       \s 任意空白
       \S 任意非空白
       \w 任意字类字符，包括_ 等同[A-Za-z0-9]
       \W 任意非单词字符
       \b匹配一个字边界，即字与空格键的位置 er\b 表示以er为结尾或者其后有空格的单词
       \B
   ```
  * Matcher索引方法
    * public int start() 返回之前匹配的初始索引
    * public int start(int group) 给定组捕获的子序列的初始索引
    * public int end() 匹配字符后的偏移量
    * public int end(int group) 
    * public boolean lookingAt() 将从区域开头的输入序列与该模式匹配，不要求全部匹配
    * boolean find() 查找该模式匹配的输入序列的下一个序列
    * boolean matches() 整个区域与模式是否匹配,要求全部匹配
* Stream File IO
    ![](https://raw.githubusercontent.com/BlissSeven/image/master/spring/2020/08/14/15-35-28-34060eeffcbacabb0423f52305cf458c-20200814153528-0b2e5b.png)
   * 控制台
     * 输入
       * 控制台输入由System.in完成，为获得一个绑定到控制台的字符流，将System.in包装在BufferedReader对象中
         ```java
         int read( ) throws IOException //从BufferReader对象读取一个字符，流结束返回-1
         String readLine() throws IOException //读取字符串

               BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
         char cha;
         do{
               cha=(char)br.read();
               System.out.println(cha);
         }while(cha != 'q');

         String str;
         do{
               str = br.readLine();
               System.out.println(str);
         }while(!str.equals("end"));
         String strend = "end";
         System.out.println(strend =="end");//true
         ``` 
      * 输出
       输出由print println完成，由PrintStream定义，System.out是该类的一个引用。PrintStream继承OutputStream类，且实现了write方法 
        ```java
        void write(int byteval) //将byteval的低八位字节写到流中
         int b='A';
        System.out.write(b);
        ```
## 面向对象
### 方法
  * 可变参数
    * 一个方法只能指定一个可变参数，且必须是方法的最后一个参数
      ```java
         public static void print(double... num){
        if(num.length==0) {
            System.out.println("no argument");
        }
        for(double n:num){
            System.out.println(n);
           }
        }
      ```
   * finalize 方法   
     在垃圾收集器之前调用，可用来清除回收对象
     ```java
     //常用格式
     protected void finalize(){
     }
     ```
### 继承
* extends 可以继承一个类，implements 可以多个接口
* 通过super接口调用父类方法/变量，尤其子类构造函数需要调用父类构造函数的时候
  * 任何类的构造方法，编译器默认会在子类的构造函数调用父类的默认构造函数，调用失败则异常
* 向上转型
  * 子类向上转型为父类
  * `isAssignableFrom` 判断上转型是否成立
     ```java
     Number.class.isAssignableFrom(Integer.class)
     ```
    * `instanceof` 判断一个实例是否是某个类型
       ```java
       Object n = Integer.valueOf(123);
       boolean isDouble = n instanceof Double;
       ```
* 向下转型
  * 只有当父类原来是一个子类的引用时，转型成功（父类由子类向上转型而成）
  * 通过 instanceof 判断
     ```java
     Person p = new Student();
    if (p instanceof Student) {
        // 只有判断成功才会向下转型:
        Student s = (Student) p; // 一定会成功
    }
     ```
* 继承VS组合
  * 继承 is a
  * 组合 has a
  * 当需要做向上转型时，考虑继承   
### 多态
* 所有类均继承自Obejct，重写override Object方法
  * String toString()
  *  boolean equals(Object o)
  * int hashCode()
* final
  * 父类方法final,父类不允许子类对其方法override
  * 变量final，意味着初始化后不可修改
    * 定义时初始化
    * 在类的构造函数中初始化 !!!
### 接口
  * 接口定义变量默认 pulibc static final
  * 接口定义方法默认 public abstract
  * 使用时，实例化对象永远为某个具体的子类，通过接口去引用它，因为接口比抽象类更抽象
  * default方法
    * 给子类增加方法，且子类不用实现
     ```java
        interface Person {
        String getName();
        default void run() {
            System.out.println(getName() + " run");
        }
    }

    class Student implements Person {
        private String name;

        public Student(String name) {
            this.name = name;
        }

        public String getName() {
            return this.name;
        }
    }
     ``` 
### 包
* 包不具有继承性质
* import 
   ```java
   import mr.jun.* //不包括子包的class
   import static java.lang.System.*     //导入静态字段和方法
   ```
* 默认导入
  * 默认import当前package的其他class
  * 默认import java.lang.*
### 文件
   * 输入
     * FileInputStream
       ```java
       void close() throws IOException{} 关闭文件输入流以及有关的系统资源
       void finalize() throws IOException{} 清除与该文件的连接，确保在不再引用文件输入流时调用其 close 方法
       int read() 
       int read(byte[] r)//读取r的size大小的字节
       int available() 返回下一次对此输入流调用的方法可以不收阻塞的从此输入流读取的字节数
       ```
     ```java
       InputStream is=new FileInputStream("/home/bliss/sn.txt");
      //        File f=new File("/home/bliss/sn.txt");
      //        InputStream is2=new FileInputStream(f);
        byte[] bytes=new byte[512];
         is.read(bytes);
         int oneint = 0;
         oneint=is.read();
        System.out.println(new String(bytes));
     ```
   * 输出
     * FileOutputStream
        ```java
        void close() throws IOException{}
        void finalize()throws IOException {}
        void write(int w)throws IOException{}
        void write(byte[] w)
        ``` 
       ```java
       OutputStream os = new FileOutputStream("/home/bliss/sn.txt")
       File f = new File("/home/bliss/sn.txt")
       OutputStream f = new FileOutputStream(f)
       ``` 
  * 目录
    * File FileReader FileWriter
      * 创建目录--File对象方法
        * boolean mkdir() 用于创建文件夹。失败 File对象指定的文件夹存在或整个路径不存在
        * boolean mkdirs()创建文件夹及其父文件夹
      * 读取目录--File
        * 目录即File对象，包含其他文件和文件夹
        *  boolean isDirectory() 
        *   String[] list() 包含的文件以及文件夹
      * 删除目录/文件--File
        * boolean delete()   
* Scanner 类
  * 用于获取用户输入
  * next  一定读取有效字符后才结束，可忽略有效字符前的空白，将有效字符后的空格作为结束，不能获得空白
  * nextLine 以回车为结束符，可空白
    ```java
     Scanner scanner=new Scanner(System.in);
        if(scanner.hasNextLine()){
            String str=scanner.nextLine();
            System.out.println(str);
        }
        if(scanner.hasNext()){
            String str=scanner.next();
            System.out.println(str);
        }
        scanner.close();
    ``` 
## 异常     
  ![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2020/08/14/17-34-35-a6c7405ad2c619a15bc380a6d9a4f855-20200814173435-16e88c.png)
  * 必须捕获的异常Exception 及其子类，但不包括RuntimeException及其子类，---检查性异常 不处理编译不通过
  * 运行时异常 
  * 错误
  * 所有未捕获的异常，最终也必须在main()方法中捕获，不会出现漏写try的情况。这是由编译器保证的。main()方法也是最后捕获Exception的机会
  * 异常也可以不捕获，再把异常抛出 throws，也可以通过编译器检查
  * 捕获异常后至少 调用printStackTrace()方法打印异常栈
   ```java
   //catch 中异常依次往下越来越抽象
   try{
   }catch{}
   catch{}
   finally{
      //一定会执行
   }
      try {
          process1();
          process2();
          process3();
      } catch (IOException | NumberFormatException e) { // 多个同级异常时 IOException或NumberFormatException
          System.out.println("Bad input");
      } catch (Exception e) {
          System.out.println("Unknown error");
      }
   void withdraw(double amount) throws RemoteException,InsufficientFundsException{
   }
   //自定义异常
   class RemoteException extends Exception{
   }
   ```
   * 断言    
     调试方式，失败返回AssertionError异常，断言不能用于可恢复的程序错误，只应该用于开发和测试阶段
     对于可恢复的程序错误，不应该使用断言，应该抛出异常并在上层捕获
     * ```assert x >= 0 : "x must >= 0";```//添加一个可选的断言消息
     * JVM默认关闭断言指令，即遇到assert语句就自动忽略了，不执行
     *  java -ea Main.java 启用assert
* 日志
  * java.util.logging JDK的Logging定义了7个日志级别
  * SEVERE | WARNING | INFO | CONFIG | FINE | FINER | FINEST
  * Logging系统在JVM启动时读取配置文件并完成初始化，一旦开始运行main()方法，就无法修改配置
  * 配置不太方便，需要在JVM启动时传递参数-Djava.util.logging.config.file=<config-file-name>
    ```java
     import java.util.logging.Level;
    import java.util.logging.Logger;
     Logger logger = Logger.getGlobal();
        logger.info("start process...");
        logger.warning("memory is running out...");
        logger.fine("ignored.");
        logger.severe("process will be terminated...");
    ```
   * SLF4J+Logback
### 反射
反射--程序在运行期间可以拿到一个对象的所有信息，解决在运行期间，对某个实例一无所知的情况下，如何调用其方法
#### Class 类
* class 本质是数据类型，没有继承关系的数据类型无法相互赋值
* class 类时JVM在执行过程中动态加载的，第一次读取到一种class类型时，将其加载进内存。
* 每加载一种class类，JVM创建一个Class 类型的实例，并将两者关联起来（Class 一个名字为Class的class类 :laughing:  :confused:）
    ```java
    public final class Class{
      private Class{}//只有JVM可以构造
    }
    ```
  * 新建String类为例 ，JVM加载String时，读取String.class到内存，为String创建一个Class实例并关联
  * ```Class cls=new Class(String)```  
  * 如图JVM每个Class实例都指向一个数据类型（class或interface） 
    ![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2020/08/15/17-32-01-76f4f467c556e76bfb9c7c9954f4872a-20200815173201-a333f9.png)
* 一个Class包含class的所有信息    ，**通过Class实例获得类String信息的方法为反射**
   ![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2020/08/15/17-37-04-3c233784562e9f8a79ae111655d35afd-20200815173704-4d333d.png)
* 获取一个class类绑定的Class
  * `Class cls=String.class;` //通过一个class的静态变量
  * `String s="hello";Class cls=s.getClass()` class实例的getClass接口
  * `Class cls=Class.forName("java.lang.String")` 根据一个class的完整类名

* Class 实例比较
  *  Class 实例在JVM中是唯一的
      ```java
      Class cls1=String.class;
      Class cls2=s.getClass();
      boolean same=cls1==cls2;//true
      ```
     ```java
         Integer n = new Integer(123);
        boolean b1 = n instanceof Integer; // true，因为n是Integer类型
        boolean b2 = n instanceof Number; // true，因为n是Number类型的子类
        boolean b3 = n.getClass() == Integer.class; // true，因为n.getClass()返回Integer.class
        boolean b4 = n.getClass() == Number.class; // false，因为Integer.class!=Number.class
     ```
  * 获取class 信息
    ```java
               System.out.println("Class name: " + cls.getName());
              System.out.println("Simple name: " + cls.getSimpleName());
              if (cls.getPackage() != null) {
                  System.out.println("Package name: " + cls.getPackage().getName());
              }
              System.out.println("is interface: " + cls.isInterface());
              System.out.println("is enum: " + cls.isEnum());
              System.out.println("is array: " + cls.isArray());
              System.out.println("is primitive: " + cls.isPrimitive());
              /**
                Class name: java.lang.String  --------String
                Simple name: String
                Package name: java.lang
                is interface: false
                is enum: false
                is array: false
                is primitive: false

              Class name: [Ljava.lang.String;    ---------String[]
              Simple name: String[]
              is interface: false
              is enum: false
              is array: true
              is primitive: false
              **/
    ```
* 动态加载   
  JVM在执行Java程序的时候，并不是一次性把所有用到的class全部加载到内存，而是第一次需要用到class时才加载，可以在运行期根据条件来控制加载class
   ```java
   // Commons Logging优先使用Log4j:
      LogFactory factory = null;
      if (isClassPresent("org.apache.logging.log4j.Logger")) {
          factory = createLog4j();
      } else {
          factory = createJdkLog();
      }

      boolean isClassPresent(String name) {
          try {
              Class.forName(name);
              return true;
          } catch (Exception e) {
              return false;
          }
      }
   ```
#### 访问字段
更多的给工具或者底层框架使用，目的是在不知道目标实例任何信息的情况下，获取特定字段的值   
* 通过Class实例获取字段信息
  * `Field getField(name)` 获取public，包括从父类继承的字段
  * `Field getDeclaredField(name)` 获取本类所有字段，包括private，不能获取继承的字段（获取的private字段，人不能访问该字段的值，除非 `setAccessible(true)`）
  * `Field[] getFields()` 获取所有public字段(包括父类)
  * `Field[] getDeclaredFields()`
  * 一个Field对象包含一个字段的所有信息
    * `getName` 返回字段名称
    * `getType` 字段类型 String.class
    * `getModifiers()` 字段修饰符 private/public/final
     ```java
      public final class String{
        private final byte[] value;
      }

     Field f = String.class.getDeclaredField("value");
      f.getName(); // "value"
      f.getType(); // class [B 表示byte[]类型
      int m = f.getModifiers();
      Modifier.isFinal(m); // true
      Modifier.isPublic(m); // false
      Modifier.isProtected(m); // false
      Modifier.isPrivate(m); // true
      Modifier.isStatic(m); // false
     ```
   * 获取字段值
     * `Field.get(Object)` 获取指定实例的指定字段的值
       ```java
       Object p = new Person("Xiao Ming")
       Class c = p.getClass();
       Field f = c.getDecalaredField("name");
       f,setAccessible(true); //不管这个字段是否是public，一律通过访问
       Object value = f.get(p);//获取指定实例的指定字段的值
       ```
     * setAccessible(true)可能会失败,如果JVM运行期存在SecurityManager，那么它会根据规则进行检查，有可能阻止setAccessible(true)
   * 设置字段值
     * 修改非public字段，需要首先调用setAccessible(true)
     * `Field.set(Object,Object)` //指定的实例/待修改的值
       ```java
            public class Main {
        public static void main(String[] args) throws Exception {
            Person p = new Person("Xiao Ming");
            System.out.println(p.getName()); // "Xiao Ming"
            Class c = p.getClass();
            Field f = c.getDeclaredField("name");
            f.setAccessible(true);
            f.set(p, "Xiao Hong");
            System.out.println(p.getName()); // "Xiao Hong"
          }
          }

          class Person {
              private String name;
              public Person(String name) {
                  this.name = name;
              }

              public String getName() {
                  return this.name;
              }
            }
       ```
#### 调用方法
* Method
  * 获取method
  * `Method getMethod(name,class)`       获取public 方法 包括从父类继承的
  * `Method getDeclaredMethod(name,class)`获取当前类的某个method 不包括父类
  * `Method[] getMethods()`
  * `Method[] getDeclaredMethods()`
* Method 属性
  * `getName()` 返回方法名称
  * `getReturnType()` 返回类型，一个Class类型表示 String.class
  * `getParameterTypes()` Class数组表示的参数类型 {String.class,int.class}
  * getModifiers() int 表示的修饰符
* 调用method
  * 通用方法
    * Method.invoke(Object,parameter)// 对object对象的method方法调用parameter参数
      ```java
        String s="hello";
        Method m=String.class.getMethod("substring",int.class)//substring  int 参数
        String r = (String) m.invoke(s,6);
      ```
  * 静态方法
    ```java
     // 获取Integer.parseInt(String)方法，参数为String:
    Method m = Integer.class.getMethod("parseInt", String.class);
    // 调用该静态方法并获取结果:
    Integer n = (Integer) m.invoke(null, "12345");
    ```
* 非public 方法
    ```java
       Person p = new Person();
        Method m = p.getClass().getDeclaredMethod("setName", String.class);
        m.setAccessible(true);
        m.invoke(p, "Bob");

              class Person {
          String name;
          private void setName(String name) {
              this.name = name;
          }
      }
    ```
* 多态
  * invoke时，会传入实际调用的对象，所以反射支持多态
* 构造方法
  * Constructor总是当前类定义的构造方法，和父类无关，因此不存在多态的问题
  *   调用非public的Constructor时，必须首先通过setAccessible(true)设置允许访问。setAccessible(true)可能会失败。
  * `Class.newInstance()` 只能调用该类public无参数构造方法。有参/非public no
  * `getConstructor(class...)`  获取某个public Constructor
  * `getDeclaredConstructor(class)` 获取某个constructor 
  * `getConstructors(class...)`  
  *  `getDeclaredConstructor(class)`
    ```java
    Person p = new Person();
    Person p1 = Person.class.newInstance();

     // 获取构造方法Integer(int):
        Constructor cons1 = Integer.class.getConstructor(int.class);
        // 调用构造方法:
        Integer n1 = (Integer) cons1.newInstance(123);
        System.out.println(n1);

        // 获取构造方法Integer(String)
        Constructor cons2 = Integer.class.getConstructor(String.class);
        Integer n2 = (Integer) cons2.newInstance("456");
        System.out.println(n2);
    ```
#### 获取继承关系
*  获取父类Class
   *  class.getSuperclass(); 一直到object父类null
    ```java
    Class i = Integer.class;
    Class n = i.class.getSuperclass();
    ```  
  * 获取interface
    * class.getInterfaces 返回当前类直接实现的接口类型，不包括父类实现的接口
    * 此外，对所有interface的Class调用getSuperclass()返回的是null，获取接口的父接口要用getInterfaces 
    * 没有implements接口时返回空数组
      ```java
       Class s = Integer.class;
        Class[] is = s.getInterfaces();
        for (Class i : is) {
            System.out.println(i);
        }
        //interface java.lang.Comparable
         // interface java.lang.constant.Constable
         // interface java.lang.constant.ConstantDesc
      ```
#### 动态代理
* 一般接口均 向上转型并指向某个实例，动态代理可以在运行期间动态创建interface实例
  * 定义InvocationHandler实例，负责接口的方法调用
  * 通过Proxy.newProxyInstance()创建interface实例 
    * 参数 ClassLoader 接口类
    * 需要实现的接口数组，至少需要传入一个接口进去
    * 处理接口方法调用的InvocationHandler
  * 将返回的Object强制转型为接口
     ```java
         interface Hello{
          void morning(String name);
      }
        public class ReflectionTest {
        public static void main(String[] args) {
            InvocationHandler handler = new InvocationHandler() {
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        System.out.println(method);
                        if (method.getName().equals("morning")) {
                            System.out.println("good morning" + args[0]);
                        }
                        return null;
                    }
                };
                Hello hello = (Hello) Proxy.newProxyInstance(
                        Hello.class.getClassLoader(),
                        new Class[]{ Hello.class },
                        handler);
                hello.morning("Bob");
            }
        }
     ```
### 重写（Override） VS 重载（Overload）
  * Override
    * 重写方法不能抛出新的异常或者比被重写方法方法更加宽泛的异常
    * 声明为static的方法不能重写，但是可以再次声明
    * final方法不能重写
  * Overload
     * 被重载的方法可以声明新的异常或更广的检查异常
     * 被重载的方法可以改变访问修饰符
     * 被重载的方法可以改变返回类型
