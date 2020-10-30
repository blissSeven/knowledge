# cmd
- [cmd](#cmd)
  - [crontab](#crontab)
    - [语法](#语法)
    - [时间格式](#时间格式)
    - [example](#example)
  - [shell](#shell)
    - [shell 变量](#shell-变量)
    - [传递参数](#传递参数)
    - [shell 运算符](#shell-运算符)
      - [算术表达式](#算术表达式)
      - [关系表达式](#关系表达式)
      - [布尔运算](#布尔运算)
      - [逻辑运算](#逻辑运算)
      - [字符串运算符](#字符串运算符)
      - [文件测试运算符](#文件测试运算符)
    - [echo](#echo)
    - [printf](#printf)
    - [test](#test)
      - [数值测试](#数值测试)
      - [字符串测试](#字符串测试)
    - [流程控制](#流程控制)
      - [if else](#if-else)
      - [for 循环](#for-循环)
      - [while 循环](#while-循环)
      - [until循环](#until循环)
      - [case 语句](#case-语句)
    - [shell 函数](#shell-函数)
      - [函数定义](#函数定义)
      - [函数参数](#函数参数)
    - [IO重定向](#io重定向)
      - [重定向](#重定向)
      - [Here Document](#here-document)
      - [/dev/null](#devnull)
    - [文件包含](#文件包含)
## crontab
### 语法
    ```bash
    crontab  [-u user] file //或者
    crontab [-u user] {-l | -r| -e}
    ```
-e 执行文字编辑器来设定时程表
-r 删除目前的时程表
-l 列出目前时程表
### 时间格式
f1 f2 f3 f4 f5 program
* f1 表示分钟,f2表示小时，f3表示一个月的第几天，f4月份，f5一个星期的第几天，program表示程序
* f1 为*时表示每分钟都要执行
* f1为a-b时表示从第a-b分钟这段时间内执行
* f1为*/n表示每n分钟时间间隔执行一次
* f1为a,b,c,d表示第a,b,c分钟执行
```java
*    *   *   *   *                
-    -    -    -    -               
|    |    |    |    |         
|    |    |    |    +----- 星期中星期几 (0 - 7) (星期天 为0)         
|    |    |    +---------- 月份 (1 - 12)         
|    |    +--------------- 一个月中的第几天 (1 - 31)           
|    +-------------------- 小时 (0 - 23)        
+------------------------- 分钟 (0 - 59)    
```
### example
* `* * * * */bin/ls` 每分钟执行一次   
* `0 6-12/3 * 12 * /usr/bin/backup` 12月份的6-12点，每间隔3小时0分，执行一次
* 程序在你所指定的时间执行后，系统会发一封邮件给当前的用户，显示该程序执行的内容，若是你不希望收到这样的邮件，请在每一行空一格之后加上 > /dev/null 2>&1 即可    
   `20 03 * * * . /etc/profile;/bin/sh /var/www/runoob/test.sh > /dev/null 2>&1 `
## shell
### shell 变量
* 使用变量
  *    ```bash
         yourname= "xxx"
         echo  $yourname 
         echo ${yourname} 
        ```
* 只读变量
  * `readonly yourname` 
  * `yourname="xcx"` 只读变量尝试修改时报错
* 删除变量
  * `unset yourname`
  * 不能删除只读变量
  * 被删除后不能再次使用
  * `echo yourname`输出为空
* shell字符串
  * 单引号
    * 单引号中任何字符原样输出，中的变量无效
    * 不能出现单独一个单引号，（对单引号转义也不行），可成对出现，作为字符拼接
  * 双引号
    * 可有变量，可转义
  * 拼接
    *   ```java
            your_name="runoob"
            # 使用双引号拼接
            greeting="hello, "$your_name" !"
            greeting_1="hello, ${your_name} !"
            echo $greeting  $greeting_1
            # 使用单引号拼接
            greeting_2='hello, '$your_name' !'
            greeting_3='hello, ${your_name} !'
            echo $greeting_2  $greeting_3

            ------------------------------------------------
            hello, runoob ! hello, runoob !
            hello, runoob ! hello, ${your_name} !
        ```
  * 字符串长度
    * `string="abcd", echo ${#string}`
  * 提取子字符串
    * index从0开始
    * `string="abcd", echo ${string: 1:3}`  bcd
    * 查找子字符串
      * 查找字符 i 或 o 的位置(哪个字母先出现就计算哪个)：
      * `string="runoob is a great site"； echo `expr index "$string" io`  # 输出 4`
* shell 数组
* 定义数组
    * `array_name=(value0 value1 value2 value3)`
    * `array_name[0]=value0`
    * array_name[n]=valuen
  * 读取数组
    * `valuen=${array_name[n]}`
  * 获得数组长度
    * 数组元素个数
      * `length=${#array_name[@]}`    
      * `length=${#array_name[*]}`
    * 单个元素长度
      * `length=${#array_name[n]}`
* 注释
  * `#`
  * ```bash
     :<<EOF 
     ....
     EOF
     #EOF可以为其他
     ```
### 传递参数
* $0 执行的文件名
* $1 第一个参数
* `./test.sh 1  2  3`
* $# 参数个数
* $* 以一个单字符串显示所有参数
  * `echo "传递的参数作为一个字符串显示：$*";`
  * 传递的参数作为一个字符串显示：1 2 3
  * 类比传入了一个参数
* `$$` 当前进程ID 
* $! 后台运行的最后一个进程的ID 
* $@ 与$*相同，但是使用时加引号，并在引号中返回每个参数。
  * 传递的参数作为一个字符串显示："1" "2" "3"
  * 类比传入了3个参数
* $- 显示Shell使用的当前选项
* $? 显示最后命令的退出状态
### shell 运算符
#### 算术表达式
表达式和运算符之间要有空格，反引号
* `expr $a + $b`
* `expr $a - $b`
* `expr $a \* $b`
* `expr $a / $b`
* `expr $a % $b`
* `[$a == $b]`,相同true，
* `[$a != $b]`，不同为true　　
#### 关系表达式
关系运算符只支持数字，不支持字符串，除非字符串的值是数字
* [ $a -eq $b ] 相等返回 true
* [ $a -ne $b ] 不等true
* -gt 左边的数是否大于右边的
* -lt 左边的数是否小于右边的
* -ge 边的数是否大于等于右边的
* -le 左边的数是否小于等于右边的
#### 布尔运算
* ! 非
* -o 或
* -a 与
#### 逻辑运算
* &&
* ||
#### 字符串运算符
* = 相等
* !=
* -z  长度是否为0 `[-z $a]`
* -n 长度是否不为0 `[-n $a]`
* $ 字符串是否为空 `[$a]`
#### 文件测试运算符
| 操作符|说明 | 举例| 
| :----: | :----: | :-------------------------------------------------------------------: | :-----------------------: |    
|-b  file |	检测文件是否是块设备文件，如果是，则返回 true。|	[ -b $file ] 返回 false。|    
|-c file |	检测文件是否是字符设备文件，如果是，则返回 true。|	[ -c $file ] 返回 false。 |
| -d file|	检测文件是否是目录，如果是，则返回 true。	|[ -d $file ] 返回 false。   |
|-f file|	检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。|	[ -f $file ] 返回 true。       
|-g file |	检测文件是否设置了 SGID 位，如果是，则返回 true。|	[ -g $file ] 返回 false。  |
|-k file |	检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。| 	[ -k $file ] 返回 false。|   
|-p file |	检测文件是否是有名管道，如果是，则返回 true。|	[ -p $file ] 返回 false。   |
|-u file |	检测文件是否设置了 SUID 位，如果是，则返回 true。|	[ -u $file ] 返回 false。   |
|-r file |	检测文件是否可读，如果是，则返回 true。|	[ -r $file ] 返回 true。   |
|-w file | 检测文件是否可写，如果是，则返回 true。|	[ -w $file ] 返回 true。   |
|-x file | 检测文件是否可执行，如果是，则返回 true。|	[ -x $file ] 返回 true。   |
|-s file |	检测文件是否为空（文件大小是否大于0），不为空返回 true。|	[ -s $file ] 返回 true。|    
|-e file |	检测文件（包括目录）是否存在，如果是，则返回 true。|	[ -e $file ] 返回 true。   |
* `-S`判断某文件是否是socket
* -L 检测文件是否存在 并且是一个符号链接
### echo
* 显示转义
  * `echo "\" it is a test \''''`   -------- "It is a test"
* 显示变量
   ```bash
    #!/bin/sh
    read name 
    #read 命令从标准输入中读取一行,并把输入行的每个字段的值指定给 shell 变量
    echo "$name It is a test"
   ```
* 显示换行
   ```bash
   echo -e "OK! \n" # -e 开启转义
    echo "It is a test"
   ```
* 显示不换行
  * echo 自动添加换行符，我们可以手动添加 \n。
    ```bash
    #!/bin/sh
    echo -e "OK! \c" # -e 开启转义 \c 不换行
    echo "It is a test"
    ```
* 显示命令执行结果--反引号
  * ``` echo  `date` ```
### printf
* %-10s 指一个宽度为10个字符（-表示左对齐，没有则表示右对齐），任何字符都会被显示在10个字符宽的字符内，如果不足则自动以空格填充，超过也会将内容全部显示出来。
  ```bash
    printf "%-10s %-8s %-4s\n" 姓名 性别 体重kg  
    printf "%-10s %-8s %-4.2f\n" 郭靖 男 66.1234 
    printf "%-10s %-8s %-4.2f\n" 杨过 男 48.6543 
    printf "%-10s %-8s %-4.2f\n" 郭芙 女 47.9876 
  ```
### test
检查数值、字符、文件某个条件是否成立
#### 数值测试
|--- |----|
| :-: | :-: |
|-eq|等于为真|
|-ne|不等为真|
|-gt|大于为真|
|-gt|大于等于为真|
|-lt | 小于为真|
|-le|小于等于为真|
```bash
if test $[num1] -eq $[num2]
then
fi

a=5
b=6
result=$[a+b]  
#等号左右不能空格！！！！
```
#### 字符串测试
|--|--|
| :-: | :-: |
| = | 等于为真 |
| != | 不等为真 |
| -z 字符串 | 长度为0为真|
| -n 字符串| 长度不为0为真|

### 流程控制
#### if else
```bash
if [ $a == $b]
then  
    echo "a=b"
elif [$a -gt $b]
then
    echo "a>b"
elif [$a -lt $b]
then
    echo "a<b"
else
    echo "no solution"
fi #倒置的if :smile:
```
```bash
num1=$[2*3]
num2=$[1+5]
if test $[num1] -eq $[num2]
then
    echo '两个数字相等!'
else
    echo '两个数字不相等!'
fi
```
#### for 循环
```bash
for var in item1 item2 item3
do
     command1
     command2
done

for var in item1 item2; do command1; command2; done;
```
#### while 循环
```bash
int=1
while (($int<=5))
do
    echo $int
    let "int++" #用于执行一个多个表达式
done 
```
```bash
while true #死循环
do
done
```
```bash
while : #死循环
do
done
```
#### until循环
```bash
until condition
do 
    command
done 
```
#### case 语句
```bash
case $anum in 
1) echo 'you choose 1'
;; # 类似 break
2) echo 'you choose 2'
;;# 每一模式最后必须以右括号结束，模式支持正则表达式
3|4) echo 'you choose 3 or 4'
;;
*) echo 'catch any pattern'# *捕获任意模式
esac
```
### shell 函数
#### 函数定义
* 加中括号的可加可不加
* function 关键字 ---（）函数括号 ---返回return
```bash
[function] funname [()]{
  [return int;]
}
```
#### 函数参数
```bash
funwithparam(){
  echo 'first param $1'
  echo 'second param $2'
 echo 'tenth param ${10}'
  echo 'param num $#'
  echo '作为一个字符串输出所有参数 $*'
}
```
|----|      ----     |
| :-: |      :-:       |
|$#  |参数个数|
|$*  |以一个单字符串|
|$$ | 当前进程ID|
|$!  |后台运行的最后一个进程的ID|
|$@|与$*相同，但是使用时加引号，并在引号中返回每个参数。|
|$-|显示shell使用的当前选项，与set相同|
|$?|显示最后命令的退出状态，0表示没有错误|
### IO重定向
|----------|-----------------------------------------|
|----------|-----------------------------------------|
|cmd > file|输出重定向到file|
|cmd < file|输入重定向到file|
|cmd >> file|输出以追加方式到file|
|n > file|将文件描述符为n的文件重定向到file|
|n >> file|追加|
|n >& m|将输出文件m n 合并|
|n <& m|将输入文件m n 合并|
|<< tag|将开始标记tag和结束标记tag之间的内容作为输入|
* 文件描述符0 输入 1输出 2 错误
#### 重定向
一般情况下，每一个linux命令都会打开三个文件
* stdin 描述符为0，默认从stdin读取数据
* stdout 描述符为1，默认向stdout输出数据
* stderr 描述符为2，向stderr写入错误信息
`cmd > file`将stdout重定向到file          
`cmd < file`将stdin重定向到file        
`cmd 2 > file`将stderr重定向到file          
`cmd > file 2>&1`将stdout和stderr合并后重定向到file    
`cmd < file1 > file2`将stdin 重定向file1，stdout重定向file2
#### Here Document
* 将两个delimiter之间的内容作为输入传递给cmd
* 结尾delimiter顶格写
* 开始的delimiter前后空格忽略
```bash
cmd << delimiter
    document
delimiter
```
```bash
wc -l << EOF
    欢迎来到
    菜鸟教程
    www.runoob.com
EOF
```
#### /dev/null
* 执行某个命令，不希望在控制台显示输出结果，可将输出重定向到/dev/null，起到禁止输出效果
* `cmd > /dev/null`
* /dev/null 特殊文件，写入的内容会抛弃，读取内容为空
* `cmd > /dev/null 2>&1` 屏蔽stdout stderr
* 2和>之间没有空格，2>才表示错误输出 
### 文件包含
* `. filename`注意.和filename之间的空格
* `source filename`
```bash
#!/bin/bash
. ./test1.sh 
source ./test1.sh
``` 
获取所有文件夹名称
```shell
for dir in $(ls /usr/)
do
  [ -d $dir ] && echo $dir
done
```
文件后缀、目录提取
* `${var##*/}` 去掉变量var从左边算起的最后一个'/'字符，及其左边内容，返回右边内容
* `${var##*.}`
* `${var#*.}` 去掉变量var从左边算起的第一个.字符及其左边内容，返回右边
* `${var%%.*}`去掉变量var从右边算起的最后一个'.'字符及其右边内容，返回左边







