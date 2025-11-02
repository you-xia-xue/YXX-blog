这里存放着我早期散碎的笔记稍后整理

`Java`给我的印象就是类实例的集合体，不同的类实例通过调用彼此运行。

`Java`有4种版本：

	1. `Java SE`(`Sabdard Edition`)标准版
	2. `Java EE`(`Enterprise Edition`)企业版
	3. `Java ME`(`Micro Edition`)移动版
	4. `Java Card` 针对智能卡（如SIM卡）的版本


一个`Java`程序应当是这样的(使用驼峰命名类)
``` Java
package com.youxiaxue.io;
public class Main{
	public static void main(){
	}
}
```
package是一个域名，类似于`C/Cpp`的命名空间，作用是把类整合在一个命名空间内
在`C/Cpp`中我们通过导入库来实现调用库的方法，在`Java`我们调用类，有标准类也有内置类，也可以调用外部类,常用的方法类。IDE也可以自动导入
```java
import java.lang;//调用外部类
```
# 编译和运行
我们可以在命令行调用`java`编译器，来编译`.java`文件（需要在对应文件夹）
``` shell
javac Main
```
编译器会把它编译成JVM字节码`.class`
我们可以把这个文件跨平台传输，`Java`虚拟机会把字节码解释成本机的机器码
以下方法会调用java虚拟(`Java virtual Mechine`)，并运行
必须要包含域名和源文件名
``` Shell
java com.youxiaxue.io.Main
```
# 注释
注释有三种方法
``` java
// 这是一个单行注释
/* 这是一个多行注释
*/
/** 这是 一个文档注释（说明文档用途）
*/
```
前两者和`C/Cpp`一致，但文档注释可以插入标签
通常作为描述文件用[Java 文档注释 | 菜鸟教程](https://www.runoob.com/java/java-documentation.html)
## 类型
类型分为基本类型、引用类型、无类型
## 基本类型
`byte`占1字节
`int`占4字节在java可以这样`123_456_678`
`long`占8字节
`float`占4字节
`double`占8字节
`char`占2字节，存储字符
`boolean`占1字节
``` java
byte age = 30;
long views = 3_123_456_789L;
float a = 1.0f;
char word = 'A';
boolean bool = true;
```
可以对基本类型后面加入`.`来快速生成代码
`void`无类型
## 引用类型
``` java
int a=1;
int b=a;//a,b是两块内存，只是b用了a的值
Point point1 = new Point(1,2)
Point point2 = point1;//point1和point2指向同一块内存
```
我们使用变量名引用使用`new`生成的内存，很像`Cpp`的指针。如果没有初始化，默认值是`null`
## 语句
语句分为条件语句`if`，循环语句`for、while、do while、 for each`，选择语句`swich case`
## 运算符
运算符有算数运算符、布尔运算符、位运算符、逻辑运算符、三元运算符、特殊运算符
## Java的代码规范
