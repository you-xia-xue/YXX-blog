# Java导入（整理中）
这里存放着我早期散碎的笔记稍后整理，因为这是我学习笔记，所以不是很完善  
[这里附上菜鸟教程，作为参考](https://www.runoob.com/java/java-tutorial.html)，
[廖雪峰的官方网站](https://liaoxuefeng.com/books/java)  
`Java`给我的印象就是类实例的集合体，不同的类实例通过调用彼此运行。  

## 挑选一个IDE
我个人选择的是	IntelliJ IDEA(我开发MCmod时也会用到)
## 第一个Java程序
一个`Java`程序应当是这样的(使用驼峰命名类)
``` Java
package com.youxiaxue.io;//使用一个域名作为包名
import java.lang.System;//导入java.lang包名中的System类
public class Main{
	public static void main(){
		java.lang.System.out.println("hello World!");
		System.out.println("hello World!");
	}
}
```
我们接下来每条简单解释一下：  
1. `package com.youxiaxue.io` `package`(包)通常是用来区分不同实现的同名类的。声明包名后，在其他文件调用时还需要输入完整包名才能调用本文件的类  
	类似于`C/Cpp`的命名空间，作用是把声明的类整合在一个包名下，如果类内部成员不做声明，默认只能在包内调用  
在`C/Cpp`中我们通过导入库来实现调用库的方法，类似的在`Java`我们调用类，有内置类（可以直接调用），也可以调用外部类（在文件外定义的）,常用的方法类,IDE会自动导入。
2. `import java.lang` `import`是用
## 编译和运行
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
## 注释
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
### 基本类型
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
### 引用类型
``` java
int a=1;
int b=a;//a,b是两块内存，只是b用了a的值
Point point1 = new Point(1,2)
Point point2 = point1;//point1和point2指向同一块内存
```
我们使用变量名引用使用`new`生成的内存，很像`Cpp`的指针。如果没有初始化，默认值是`null`
### 语句
语句分为条件语句`if`，循环语句`for、while、do while、 for each`，选择语句`swich case`
### 运算符
运算符有算数运算符、布尔运算符、位运算符、逻辑运算符、三元运算符、特殊运算符
### Java的代码规范
