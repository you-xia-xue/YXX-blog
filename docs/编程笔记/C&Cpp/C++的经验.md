[Cpp参考网站](https://cppreference.cn/)
# 重载运算符（operator）
C++提供了使用`operator`关键字来定义关键字的功能
双目运算符的第一个参数在左，第二个参数在右
``` Cpp
class test{
public:
	float y,x;
	test(float X,float Y):x(X),y(Y){}
	test ADD(test& other){
		return test(x+other.x,y+other.y);
	}
//以下方法逻辑:使用+时调用operator+函数
	
	test operator+(test& other){
	return ADD(other);
	}
}
```
`operator+`本身可以作为方法去在其他地方被调用，但是不建议这样做
`operator+`的本质是把特殊字符作为函数使用用
```Cpp
test a(4f,5f);
test b(6f,7f);
teat c = a+b;
//上下等价
test C = a.operator+(b)
```
如果在非成员函数进行的运算符重载需要显式指出参数

我们通过`std::cout<<`的方法可以推断出`<<`本身也是被重载了，在标准库里的`<<`默认只能输出库里被定义的类型。
如果想要在使用`std::cout`打印我们定义的类型，需要写一个对应的重载函数以实现
``` Cpp
std::ostream& operator<<(std::ostream& stream,const test& other){
	stream<<other.x<<","<<other.y;
	return stream;
}
```
# 类型转换
C风格的强制转换类型:`int C;(float) C`，易导致未定义行为

Cpp风格的提供了不同情况的强类型转换的检测（额外的性能开销），更容易的定位
`dynamic_cast<目标类型>(被转换变量)`动态检测类型是否能成功强制转换，不能转换则返回nullptr。在使用中可以用于检测基类指针能否成功转换不同的派生类
`reinterpret_cast`把类型解释成另一个类型（等同于类型双关）
`static_cast`编译期检测是否能强制转换，更多的检查（不合法的转换会导致编译失败），派生类可被转换成基类（不能是虚基类）
`const_cast`只读变量类型转换成普通变量
## 运行时类型信息 Run-Time Type Information(RTTI)
通过[[#虚函数（virtual funcion）]]表，在运行时获取对象的实际类型，可以绕过静态类型获取真实类型，并调用。
通过`dynamic_cast`指针，实现基类安全检测转换派生类，不能跨多个继承类转换
通过`typeinfo`获取真实类型
如果不需要这样的性能开销,可以这样关闭
![[Pasted image 20250831122446.png]]
## 隐式转换 
在函数调用和赋值时遇到参数类型不对，但是定义了如何转换其他类型的值。编译器会进行隐式转换成对应的类型然后在进行赋值
比如char* 数组可以转化成std::string类型
在类中的构造函数使用了单个参数。
当使用参数的类型赋值类实例时，会将参数转换成临时实例赋值给类
如果函数使用了类作为参数，那么函数也会通过构造参数来构建一个临时实例
使用`explicit`显示声明不可进行意外的隐式转换
单个语句不支持多个隐式转换

# 智能指针
**需要\<memory>库**
智能指针是原始指针的包装，提供了自动创建堆对象分配内存(new)，根据使用的智能指针自动销毁(delete)的方法
`unique_ptr`作用域指针
	出作用域则连同堆对象被删除，一个对象不可被多个该类指针指向。
	对应使用`make_unique`方法
`shared_ptr`引用指针
	以使用多个该类指针指向堆对象（有一个隐藏的引用计数开销），如果没有指针指向该对象则该对象被删除。
	可以通过`=`赋值来复制该指针
	对应使用`make_shared`方法
`week_ptr` 虚指针
	可以用来指向`shared_ptr`指针，但是不会增加对应对象的计数，可以用来检测该对象是否存在
``` Cpp

#include <iosteam>
#include <memory>
//假设有一个Test的类
{
//这里在栈上建立了一个智能指针（可以使用auto），同时使用另一个make_unique方法来比较安全的创建堆对象（调用构造函数）
std::unique_ptr<Test> test = std::make_unique<Test>();
//出了作用域，智能指针销毁，同时销毁堆对象（调用析构函数）
//也可以使用new,通过构造函数初始化对象
std::unique_ptr<Test>(new Test())
}

```

## 栈对象和堆对象

### 栈内存
关于栈的内容(比较复杂)
只需要知道程序是在有限连续空间的栈中进行操作的
进入一个新的作用域就压栈，退出一个作用域就弹栈
栈从高地址位向下生长
而且足够的小的栈内存可以直接载入CPU缓存栈(Cache)里更高速的运算
### 堆内存

因为分配给程序栈内存是有限的，而堆内存的分配是从闲置内存（可以用过操作系统动态分配）分配的 
可以为大量类似的对象申请堆内存，以避免栈内存耗尽
但是如果内存耗尽，导致的结构是灾难性的（整个操作系统）
而且向操作系统请求分配时消耗性能较多，如果预先分配堆内存可以有效减小这一步性能开销，但是查找空闲内存的性能开销仍在
``` Cpp
class tes{}
//下面的类s指针指向了一个堆内存的实例
test* one = new test();//此处会调用构造函数，new会返回一个指针
//用new创建的实例需要使用delete释放内存(不然会内存泄漏)
delete one;
//如果使用
```


|      优点       |             缺点             |
| :-----------: | :------------------------: |
| 不会随着作用域的结束被释放 |      需要有指针指向（否则无法被调用）      |
|    不会占用栈内存    | 需要手动(delete)释放内存(否则容易内存泄漏) |
位域允许指定成员占用的类型（作为存储单元）具体比特位数（unsigned int类型通常位4字节，32比特位，下面的a,b,c分别占用从低位到高位共6比特位）
``` C
struct Flags { 
unsigned int a : 1; // 占用1位 
unsigned int b : 2; // 占用2位 
unsigned int c : 3; // 占用3位 };
```
若是总位数超过了存储单元
则声明第二个存储单元，前面未使用的存储单元为无效值
``` C
struct Bits2 { 
unsigned int x : 30;// 30位 
unsigned int y : 4; // 4位（30+4=34 > 32，需新单元） 
}; 
// 大小：8字节（2个unsigned int单元，每个4字节对齐）
```
如果未声明成员名，则作为填充/占位
如果是声明0宽度的位于，会强制
``` C
struct{
unsigned char : 4;//占用4比特的空间
unsigned char : 0;//后续位域使用新单元
//此时是1字节大小
unsigned char : 4;
//此时总共是2字节大小
}
```

# 内存对齐
![[Pasted image 20250825101551.png]]
# 引用 （References）
函数传递参数只会传递其副本而非本体，这导致上层函数的局部变量是无法被修改的。
在C语言学过通过传递指针并解引用实现跨函数地修改局部变量
**引用是上述问题更简单更简洁的实现方式**
它可声明一个变量的引用，对引用的操作会直接作用在变量上。因为和变量是绑定的，所以必须初始化一个变量且不能被修改。
在底层上他们都是传递a的地址，只是在编译器层面将引用和变量进行操作绑定
![[Pasted image 20250824093423.png]]
# 类（class）
类包含不同的数据类型，函数，通过类可以声明并创建实例，可以对实例进行类的相关操作
1. public（公开）修饰类成员可以让类内部的变量/函数可以被外部直接调用
2. 使用private（私有）修饰类成员让其只能被内部调用且不可被继承
3. 使用protected（保护）修饰类成员，会使得该类成员私有且可以被继承调用
4. 含有纯虚函数的类为抽象类(不能建立实例)
	不含纯虚函数的类为具体类(不能建立实例)
5. 在类内声明*friend*类（友元），可以使得该友元类使用该类的所有成员。如果用声明一个友元函数，则函数可以调用该类的所有成员（无需通过公共接口）

|     结构体      |       类       |
| :----------: | :-----------: |
|  数据类型组成的结构   | 功能和不同类型组成的结构  |
| 默认公有（public） | 默认私有（private） |
结构体/类定义的变量称为成员变量(member)
同理也有成员函数
可以用不同的命名方式作为提示（比如m_)
可以使用：：(作用域运算符)指定类
可以对成员函数用const修饰以实现调用只读类成员
如果成员用mutable修饰，则在const 成员函数内也可以对该成员进行修改
``` Cpp
class A{
private:
int m_x,m_y;
public:
	void GetX() const{
	//m_x只读
		std::cout<< m_x;
	}
	//第一个const描述一个不可被修改的int变量，第二个描述不可修改的int指针，第三个描述调用的类成员不可被修改
	const int* const SetX() const{}
}
```

我们可以在构造函数这样定义实现成员的初始化(避免在构造函数内进行调用赋值导致的性能开销)
``` Cpp
class test{
private:
	int someone,asd;
public:
//冒号位置在函数声明后方
	test(int X)
	:someone(X),asd(8);//成员带括号，括号内赋值
	{
	}	
}
```

实例内每一个非静态成员函数调用时，本质上向函数传递了实例本身，所以可以在函数内调用实例的成员变量
![[Pasted image 20250824154001.png]]
## this
使用this关键字可以获取指向当前类的指针（不可以被修改的指针常量）
以确保在相同变量名的情况下能正确调用当前实例的成员
``` Cpp
class test{
public:
	int x, y;
	test(int x,int y){
		this->x = x;
		this->y = y
		test* const e = this;
		test& f = *this;
	}
	test()const{
		const test* const e = this;
		const test& f = this
	}
}
```
## 单例(singleton) 设计模式
[教学视频](https://www.bilibili.com/video/BV1Dk4y1j7oj/?p=83&share_source=copy_web&vd_source=db8aceb651e9de7c3673c5eee54d917f)
一个类在全局中只有一个实例，则为单例。
使用`::`（作用域运算符）调用类的静态方法来调用实例并对其操作
例
``` Cpp
class singleton{
public:
singleton(const singleton&)=delete;//删除复制构造函数
 //类的静态方法，返回实例的引用.可以直接通过方法调用其他方法
 static singleton& Get(){
 //因为static修饰的变量在静态内存中，所以不用创建变量通过该方法在静态内存中调用即可
	 static singleton Instance;
	 return Instance;
 }
private:
singleton(){}=delete;//可以删掉构造函数避免被创建其他实例
}
```
`singleton::get().function`这样调用实例的方法
`auto& a = singlenton::get()`创建单例的引用
使用[[#命名空间(namespace)]]可以达到类似的效果，只是不能创建引用。 

## 构造函数
构造函数是创建实例时对实例进行的操作
如果使用下面的代码将无法创建实例
``` C++
//删除默认的函数
log()=delete;
```
在类中用类名定义一个构造函数使创建实例时，会使用该函数初始化实例或者其他操作
![[Pasted image 20250825092131.png]]
这里e使用了类的构建函数，并初始化了成员变量
``` Cpp
Entity e(10.f,5.f)
```
### 拷贝构造函数
Cpp默认提供了一个拷贝构造函数以实现实例之间的赋值
但是会出现复制指针导致的多指针指向同一块内存的情况出现（浅拷贝）。
如果要避免该问题需要深拷贝：分配并复制内存，指针指向新的内存以避免重复释放
``` Cpp
class String{
int* x,y;
public:
//默认构造函数原理
	String(){};
//默认拷贝构造函数原理
	/*String(const String &other):x(other.x),y(other.y){}*/
//深拷贝
	String(const String &other):y(other.y){
	x = new int;
	memcpy(x,other.x,sizeof(int))
	}
}
```
尽量使用`const`+引用调用函数以避免重复创建实例
## 析构函数
当一个实例被销毁时（无论是栈实例，还是堆实例），会使用这个函数。在实例的生命周期结束时，实例都会调用析构函数再销毁
``` Cpp
class Notclass{
~test(){}}
```
也可以通过实例调用析构函数（不会销毁实例）
### 虚析构函数
多态使用基类指针销毁时只会调用基类的析构函数，使用`virtual`使得析构函数称为虚函数，和普通虚函数相同，不同的是：
1. 基类必须提供实现
2. 派生类的析构函数自动变成虚函数（可以省略`virtual`）
## 继承
[学习视频](https://www.bilibili.com/video/BV1Dk4y1j7oj/?p=28&share_source=copy_web&vd_source=db8aceb651e9de7c3673c5eee54d917f&t=410)
一个类可以继承其他类（基类），使得基类（派生类）可以得到派生类的公开(public)和保护(protected)的功能（默认继承为类的私有成员）
结构体类似原理（不过默认继承为公开成员）
派生类总是基类的超集，派生类可以在基类的基础上实现不同的功能
``` Cpp
class father{
public:
int a,b
}
//下面继承father类的成员为公开成员
class child : pubilc father{}
//child的实例可以调用a,b
class child1 : pubilc father,pubilc child2{}
//多继承
```
**创建派生类会调用基类的构造函数，析构同理**
如果派生类定义了和基类同名的成员，那么会隐藏基类的同名成员，派生类调用需指出：
\[基类名]：\[成员]
如果外部函数使用了基类指针作为参数，那么也可以取派生类的地址作为参数
这样调用时，如果派生类和基类有同名函数，只能使用基类定义的同名函数（可能不符合预期），如果想使用派生类的同名函数实现需要使用虚函数。

### 虚函数（virtual funcion）
派生类继承基类的函数时，只能调用基类方法的实现，虚函数可以让派生类覆写基类的函数

虚函数的核心是实现对派生类中继承的基类成员的覆写。
基类负责提供通用的功能（描述基本功能），派生类可以选择覆写(可以用override特指)这一接口使该方法与自身绑定，绑定后编译会在调用（可以用基类指针/引用调用）该类方法会自动匹配该类
覆写要求:
1. 返回类型一致（如果返回派生类则可以不同）
2. 参数一致
3. 函数名一致
下面是定义在类的动态实现
``` Cpp
class X {
public:
	virtual void say(){//虚函数
	 std::cout<<"test"<<std::endl;
	}
}
class X1 : public X{
	void say() override {//覆写函数
	std::cout<<"test"<<std::endl;
	}
}
```
也可以在类外定义不同的重载函数作为静态实现

虚函数会造成性能开销：
	 需要内存建立V表以映射不同的类型到对应的函数内
	 调用时需要遍历V表以确定使用的函数
	 需要指针指向V表
### 接口（interface）（纯虚函数）
这样定义一个虚函数:
``` Cpp
virtual void test()=0;
```
这表明该成员方法没有实现，该类的派生类必须要实现该函数（覆写）才能构建实例
**使用纯虚函数的类不能建立一个实例**
从需求上看纯虚函数给派生类提供了一个需要被实现的方法，指导派生类需要实现什么
## 封装
通过控制类的访问权限，实现隐藏必要的操作，同时开放对外的接口。实现保护数据同时隐藏类内部的操作。
1. 数据安全
2. 降低耦合
3. 简化使用
4. 代码可控

## 多态
通过指向派生类的基类指针，调用基类的虚函数接口，使用派生类的函数实现
**一个接口多个实现**
不过本质是根据实例类型取调用对应类的方法

可以写一个仅使用基类指针作为参数的通用函数来处理所有的派生对象
优点
1. 接口与实现分离（基类接口，派生类实现）
2. 扩展性强（派生类只覆写，不改调用）
3. 代码更简洁且通用
## 静态（static）
**如果只论在结构体和类外的静态变量或函数**，和C的功能是一样的（调整生命周期，内部链接属性）
1. 在编译时就初始化并创建在程序中（只初始化一次） 
2. 功能上类似于文件范围内的“private”
3. 可以解决不同翻译单元名字冲突的问题
4. 作用域则为对应的函数内（在函数内才能被调用）
可以使"s_"作为提示

**使用static在结构体和类中修饰变量/函数会使其丧失类的特性（变量本身先于实例产生）**

---
**结构体和类使用static修饰变量**
编译时就会为结构体/类建立实例通用的静态变量
结构体/类创建实例的共用同一个静态变量。
虽然可以通过实例操纵变量，但本质上是对“类”的修改
所以可以使用：
\[结构名/类名]::\[变量名]对类的静态变量进行修改

**结构体和类使用static修饰函数**
编译时就会为结构体/类建立一个通用的静态方法
静态函数同理\[结构名/类名]::\[方法名]进行调用
但是静态函数不能调用非静态变量：因为静态函数先于实例存在。
若调用了一个非静态变量
从逻辑上讲：编译器不知道要调用哪一个实例的变量。
从本质上，函数调用时没有传入实例，但是调用了一个未知的实例的成员变量
所以出现了语法错误
# 声明外部变量/函数(extern)
总体功能和C一样：调用/使用其他文件内全局变量
但可以使用extern "C"{\[内容]}向编译器说明内容按照C标准进行编译/链接

# 枚举enum（enumeratetion）
 [说明](https://www.bilibili.com/video/BV1Dk4y1j7oj/?p=25&share_source=copy_web&vd_source=db8aceb651e9de7c3673c5eee54d917f&t=169)
枚举这样定义：
enum （可选名）（：可选整数类型名）{(内容)}
enum后面接名字则构建了一个枚举类型，使用枚举类型可以声明枚举变量（限制变量的值）
可选名后面 接整数类型名可以指定元素的大小范围（用于控制内存）
枚举类型可以将整数用不同的形式进行指代
默认第一个元素为0，可以对元素显式定义值，后者以此为基础递增。
![[Pasted image 20250824180800.png]]
目的是为了方便代码管理和可读性
但是容易出现名称冲突，隐式类型转换
使用enum class （名字）声明一个枚举类可以避免上述问题

# 重载函数
在相同的作用域使用相同的返回类型并拥有相同的函数名，只有且必须使用的参数不同的函数（使用时需避免类型转换引起的冲突）
![[Pasted image 20250825095149.png]]

可以用相同的名字实现相似的操作，更易读
# 数组
C++的数组和C的不同之处在于可以使用一个”编译期常量“作为数组大小使用
编译期常量指在编译时就确定的一个定值
而const是在运行时确定的（不一定是编译期常量）
``` cpp
//x必须被静态常量/常量表达式初始化
constexpr int x=5;//x可视作常量
//y被组合关键字修饰，static表示编译时就存在，const表示不可修改，同前者一样
static const int y=5;
```
C++的标准库有多种建立数组的方法（不同的性能开销  ）
``` Cpp
#include <iostream>
//引入定义模板
#include <array>
//定义一个固定的数组
std::array<int,5> another;
```

```Cpp

```
提供了更安全，更方便进行操作的模式
## 字符串
同数组类型类似
Cpp标准库为字符串提供了模板类（通用操作）:
std::basic_string 可以指定存储的字符类型
还有针对char特化操作的std::string类
``` Cpp
#include <iostream>
//载入string类
#include <string>
std::string a;
//可以对a进行特化的操作
a.find("anything")
//string提供了重载+=的方法，所以可以直接通过+=拼接不同的字符串
a += "asd"
```
C风格的多行文本是这样的
``` C
//编译器会把他们拼接在一起
const char* text="a\n"
"b\n"
"c\n"
```
可以通过一下方式来方便的写文本（同等作用）
``` Cpp

//R是操作符，作用是忽略转义字符，直接以括号内的编辑方式输出
const char* text=R"(a
b
c)"
```
### 字符类型

|    类型    |     大小      | 常量显式标记    |
| :------: | :---------: | --------- |
|   char   |     1字节     | u8(UTF-8) |
| wchar_t  | 2字节（根据环境变化） | L(宽字符)    |
| char16_t |     2字节     | u(UTF-16) |
| char32_t |     4字节     | U(UTF-32) |
### 小字符串（编译器行为）
一些编译器规定了使用小字符串时不使用new分配，而是分配栈内存，比如MSVC当字符数小于16/8时(取决于字符类型)，创建一个栈内存并存储字符串，如果大于16字符则调用new函数
## string_view(c++17)
本质是指向字符串某处的const char指针，加上大小以读取特定数量
``` Cpp
//这条代码会分配堆内存以存储字符串，(name.substr(0,3));第一个表示复制字符串的范围
std::string firstName(name.substr(0,3));
//这条代码的会创建一个栈内存的静态数组，存储
std::string_view firstName(name.c_str()+4,3);
```
优点就是不用考虑在截断位置补'\0' 

# 动态数组（vector）
 Cpp标准库提供了安全方便的一个数组的类实现以实现动态调整数组的大小
 ```Cpp
 std::vector</*(类型)*/> /*(名字)*/
 ```
以上代码会创建一个vector 对象
 会创建一个连续内存以存储对应类型的数据(指针或数据)
 可变的原理是数组空闲内存耗尽而改变大小时创建更大的数组并复制数据（一定的性能开销）
 而且复制时会触发实例的拷贝构造函数
 权衡数据类型来存放指针或数据
 
 对vector对象有以下方法
 ``` Cpp
 class test{
 int a,b,c
 //假设有个构造函数
 }
 std::vector<test> test1;
 //下面调用方法推入第一个对象（使用"{}"初始化a,b,c）
 test1.push_back({1,2,3})
 //也可以直接传递实例
 test1.push_back(test(2,3,4))
 test1.size();//返回数组的元素数量
 //以下是基于range的for循环数组
 //根据test元素数量来循环
 for (test& t:test1){}
 //该函数可以通过迭代器(iterator)+偏移量删除对应元素
 //begin()方法返回指向数组的首元素的迭代器
 test1.erase(test1.begin()+0);
 //该函数清空数组
 test1.clear();
 ```

使用对`vector`对象使用`.reserve('数字')`方法会一次性创建对应数字的内存。除非内存用尽，推入对象时不会再创建一个新的数组，而是优先使用空闲内存

使用`.emplace_back()`可以传递对应类构造函数的参数，然后再数组内构建实例
## 迭代器（**Iterator**）
迭代器是一种数据结构，提供了逻辑接口去访问容器的工具。
上述提到的`.begin()`可以获取`vector`容器的第一个元素的迭代器
不同容器的迭代器的方法是不同的(重载运算符的方法也不同)
我们可以通过使用`*迭代器`调用迭代器指向对应的元素
也可以直接使用迭代器本身作为变量进行比较
比如`while`比较同一类型迭代器实现循环调用
可以使用**rangefor**的方法（作上面`while`的语法糖)
``` Cpp
//假设我们定义了一个vector容器对象 num;
for(auto num:nums)//nums有多少个对象就遍历多少次

```
# 库
一个完整的库包含一个头文件夹和库文件夹
库文件夹内部是以及编译好的二进制文件(根据输出决定选用的文件)，
而头文件夹可以让我们使用预构建好的二进制文件中定义的函数
库分为静态链接和动态链接：
* 动态链接的库在运行时链接(允许更多的优化 )
* 静态链接的库会在编译时就链接进文件内
## 静态链接
链接器会把整个库链接到可执行文件内，提供了更多的程序优化可能
不过如果库文件十分冗杂沉重，调用的函数不多，使用静态链接的效果就不好，本体文件会过于笨重
## 动态链接
如果程序运行时依赖动态库，那么如果没有该动态库，程序将启动失败
如果程序运行不依赖动态库，可以任意按需调用
好处是如果不经常调用，可以随时卸载以腾出空间
# 模板（templates）
模板内可以制定一套规则，使得编译器可以根据传入模板的类型生成对应类型的函数实例（这样使用可以理解为泛型）
`template<typename T>` 
	`template`关键字建立一个模板,`typename`关键字定义一个类型名，根据传入模板的类型名替换`typename`定义的类型名
也可以传入对象:
`templara<int N>`
	 这样建立的模板和宏类似,编译器会将N替换成传入的int类型变量，然后为该变量构建一个函数
本质上编译器通过模板函数调用，自动创建相应的函数实现或类。
逻辑上使用一个通用的函数/类定义，指出可变部分以使得不同的类型可以使用同一个函数实现。
# 宏（Micro）
在预处理时的纯文本转换
`#define 被替换 替换体 `会把在文件里所有的被替换体替换为替换体
## 宏函数
可以使用`#define 被替换（宏变量） 替换体 `
假设替换体有宏变量，那么在替换时，会括号里的值替换进入替换体，替换对应的宏变量。
规则和C一致
# auto
`auto`关键字会让编译器自动推理声明变量的类型
这对于需要通过隐式转换赋值的变量是不好的，同时使用`auto`的变量如果被赋予的类型出现了改动，会影响所有使用到该变量的操作（但是变量本身可以通过编译）
如果使用一个冗杂且仅在类方法中定义的类型，可以使用`auto` 进行自动匹配以对该类进行更简单的操作
# lambda
  构造一个匿名的函数，构造规则如下
  ``` Cpp
  //举例
  [a](int value){
	  value =a;
  }
  ```
`[]`代表着捕获规则/变量（必须）
	`[=]`代表内部会创建调用到的外部变量（不可修改）
	`a`表示捕获变量并在内部创建同名变量
	`[&]`表示在内部引用外部变量
使用`mutable`修饰方法使得内部捕获的变量可以被修改。

如果lambda是一个封闭的函数体（不用到外部变量）可以赋值到函数指针然后作为回调函数的参数使用
如果需要使用外部的变量，可以通过`std::function<void(int)> fun`进行调用
	这里`function`是通用函数包装器，用来存储和调用“函数”
	`void`是函数的返回值
	`int`代表使用的参数
	`fun`类似函数指针的指针名，通过赋值获取函数地址（或者作为函数引用）
因为函数指针不能使用外部的变量，而`funsion`可以一同包装并调用

**通常作为算法函数需要调用到的方法参数使用**
# 命名空间(namespace)
C语言没有命名空间，这意味着如果两个库文件包含了同一个函数（包括参数名字），会触发重定义报错,通常的处理方法是`库名_方法名`，避免重复，尽管如此还是有冲突的可能。
使用`namespace 名字`可以解决该问题，他要求调用命名空间内方法时需要使用`::`显式指定调用的命名空间名字
``` Cpp
//例
namespace apple{
	void print(std::string string){
		std::cout<<string<<std::endl;
	}
}
int main(){
	apple::print();//
}
```
可以使用`using namespace 命名空间名`以直接引入命名空间的成员
可以使用`using namespace::func`引用指定成员
可以使用`namespace 别名=命名空间名`给命名空间取别名
**类可以看作一个特殊的命名空间**
## using
可以当作`typedef`使用,也可以定义模板别名,例:
`using Int = int`;
`template<typename T> using IntMap = std::map<int, T>`;
可以解除基类的成员隐藏/扩大基类成员访问权限，例(假设有一个Base基类):
`using Base::func;using::value`

# 让C++项目更快

多线程：创建多个任务，使用(多)核心（交替/同时）处理/多个任务
并行：同时执行多个任务
异步：发起和结束分离，主要函数执行后续操作,通过执行回收操作处理其他任务的结果
多线程强调”任务拆分“；并行强调“同时”，异步强调“不阻塞操作”
## 互斥锁（mutex）
如果需要避免多个线程同时访问同一共享资源使用互斥锁`mutex`实现逐个访问，使用方法：
`std::mutex mtx` 创建一个互斥锁类，有以下方法
	`.lock()`加锁，但是销毁类时不解锁会导致UB
	`.unlck()`解锁，没有锁解锁会导致UB
	`.try_lock()`尝试加锁，会返回`bool`值
逐个访问机制原理是获取`mutex`类时检查锁，（自旋）检查失败则调用内核使得线程进入睡眠（堵塞），释放CPU性能，释放锁时调用内核唤醒锁
使用RAII（资源获取即初始化）风格的锁管理类可以构建时锁定`mutex`类，销毁实例时**析构解锁锁**
有以下方法（前者比后者性能稍快）:
- **`std::lock_guard`**：构造时锁定mutex类型，析构时 **必然释放锁**（无论是否正常退出），且无法手动解锁.例`std::lock_guard<std::mutex> lock(mtx)`构造lock实例(不可移动)，锁定mt 
- **`std::unique_lock`**：更灵活，可手动解/加锁，如果析构时仍持有锁，**会自动释放**，可以使用`std::defer_lock`延迟锁定（不立刻锁定），例：`std::unique_lock<std::mutex> lock(mtx,std::defer_lock）`
### 自旋锁（Spin Lock）
反复检查锁的状态，直到解锁。
不涉及内核操作，只是单纯的循环
## 线程（thread）
**需要\<thread>库**
一个线程只能做一件事，多线程一次性做多件事。
例`std::thread worker(Dowork,num)`
	`thread`是一个类型，用来定义/执行一个线程
	`worker`创建的线程名
	`Dowork`表示线程调用的函数指针(必须)
	`num`表示传入`Dowork`的参数，如果传递引用使用`std::ref(num)

线程可以使用主线程的方法/全局变量，可以使用传入的引用和指针改变主线程
必须使用`std::thread::join()`暂停主线程以等待子线程完成**否则类型实例销毁时调用析构函数会导致程序崩溃**
使用`std::thread::detach()`可以使线程脱离主线程独自运行，结束时释放资源，且有高风险:
1. 子线程调用结束的主线程资源时会失败
2. 可能出现无法结束的后台线程
## 异步结果管理（future）
**需要\<future>库**
`std::future`有三种状态：无效（已被使用/移动），未就绪（异步未完成），就绪(操作已完成)
`std::shared_future`和`future`的区别在于可以被反复`.get`获取任务结果
声明未初始化的`std::future`实例需要`std::future<类型>`显式指定类型以创建实例，比如声明数组
该类的实例在获取异步发起中（如async）被创建并作为方法的返回值返回
有以下方法:
1. `std::future::get()`阻塞主线程直到获取异步操作的结果，返回任务的返回值（或者任务的抛出异常），方法调用后无效
2. `.wait()`阻塞主线程直到获取异步操作完成
3. `.wait_for(duration)`：超时等待duration时间（[[#chrono]]内提到的时间类型）,返回`std::future_status` 枚举
	- `std::future_status::ready`：任务已完成（结果就绪）；
	- `std::future_status::timeout`：超时（任务仍在运行）；
	- `std::future_status::deferred`：任务是延迟执行的`std::launch::deferred` 策略，未开始执行）。
4. `.wait_until(time_point)`：超时等待（带时间点）
	- **功能**：与 `wait_for` 类似，但等待截止时间是一个具体的时间点（`time_point`，如 `std::chrono::system_clock::now() + 2s`）。
	- **返回值**：同 `wait_for`，为 `std::future_status` 枚举。
5. `.valid()`
	- **功能**：判断当前 `std::future` 对象是否与一个未完成的异步操作相关联（是否可以调用 `get()` 或 `wait()`）。
    - **返回值**：`bool`，`true` 表示有效（可操作），`false` 表示无效（如已调用 `get()`、已移动、或未关联任何任务）。
### async
`std::async`是一个异步任务工具，
使用方法：`std::future abc= std::async(std::launch::async,wait_and_print，num,stream)`
	`std::launch::async`是在`std::launch`定义的枚举类(必须作为参数)，代表立即创建执行，`std::launch::deferred`则需要返回结果时执行
	`num,stream`是传入`wait_and_print`的参数
	`std::async`本身是一个方法，返回一个`std::future`类型实例（自动推导），被该方法初始化的`std::future`变量可以显式指定返回类型，默认自动推导，可以通过该实例获取线程结果
async是复制参数到函数内，如果参数需要被引用修改需要使用`std::ref()`传递参数的引用，`std::cref()`传递const的参数引用，或者直接传递地址实现内存修改
如果返回值没有初始化对应的类型变量，那么将触发临时类型析构函数，如果是`std::launch::deferred`会取消任务，如果是`std::launch::async`阻塞当前线程直到任务结束。对应类型实例销毁时，同理。

# chrono
**需要\<chorno>库**
使用`using std::literals::chrono_literals`可以使用chrono的字面量后缀
字面量使用特殊后缀会调用对应方法返回`chrono_literals`类型
如`std::this_tread::sleepfor(1s)`需要一个`chrono_duration`类型，如果使用了前者的`using`那么`1s`的`1`会被`s`调用并返回表示秒的`chrono_duration`类型
`chrono_duration.count`方法可得到
使用实现准确获取当前时间刻
`std::chrono::high_resolution_clock::now()`;
	`high_resolution_clock`是高精度时间类，`now()`返回当前时钟类型`time_point`可以转换成`chrono_duration`类型
# 算法
## 排序(sort)
`std::sort`提供迭代器使得可以排序容器（默认升序）
也可以提供函数进行特定的排序（类似qsort）

# 单一变量存储多个值

## 类型双关
使用不同类型指针绕过类型系统并访问不同类型的变量，本质是直接访问底层内存，同一内存不同类型访问
**可以通过这种方式访问相同类型连续存储的非数组形式的变量**
比如把一个类通过另一个类表示表示

## 联合体（union）
`union`使得同一块内存可以用不同类型表示（大小取决于最大的）
 通常通过结构体间接使用
 
``` Cpp
struct Union{
 union{
 int a;
 float b;
 }
}
Union.a=4;
Union.b;

```
以下方法实现了一个内存同时表示两种类型
![[Pasted image 20250830191049.png]]

## 容器存储多个值
常见是使用`struct`结构体，把多个值传入结构体，然后函数返回该结构体的
或者使用`pair` `tuple`方法创建一个容器返回实现:
1. `tuple`在`<>`写入任意数量的类型，可以此创建一个tuple实例
但是获取值需要`std::get<(offset)>(类名)` 不能直接看出值代表什么.举例
``` Cpp
//说明函数返回一个tuple类型的容器
std::tuple<std::string,int> CreatePerson(){
return {"abc",24};
//返回类型需要在“{}”内，以实现对容器/结构体的初始化
};
```
2. `pair`只能存两个不同类型的，但是只需要`.first和.second`调用值但是也不能直接看出调用了什么
3. 使用`tie(name,age)`可以把返回的容器传递给指定变量，但是需要提前声明变量
以上方法可以传递多个不同类型的值
又或者建立堆变量，用相应类型数组传递
或者使用Cpp的`array`和`vector`方法然后传递整个数组类

### 结构化绑定(C++17)
假设使用了上面的`tuple`
`auto[name,age]=CreatePerson();`
可以自动创建变量，并按照返回的结构初始化

## variant（C++17）
**需要\<variant>库**
和[[#联合体（union）]]类似，但是避免了`union`以不同形式访问同一内存的情况，为类型单独分配了内存，根据输入的类型使用对应内存
`std::variant<int,std::string> data`(必须有类型)
	如果输入`int`类型，`data.index`为0，同时修改相应的内存。同理如果为`std::string`，`data.index`为1。
	使用`std::holds_alternative<T> (data)`判断`data`是否为特定类型
使用`std::get<int>(data)`如果存储类型正确，则可获取到该值，但是类型不对会导致异常。

## any(C++17)
**需要\<any>库**
`std::any data` 可以使得data可以被赋予任何类型,有以下方法。

| `std::any_cast<T>(&any_obj)` | 提取存储的值：若类型匹配，返回 `T*`；否则返回 `nullptr`（无异常，适合安全检查）。               |
| ---------------------------- | -------------------------------------------------------------- |
| `has_value()`                | 返回 `bool`：`true` 表示存储了值，`false` 表示为空（未存储值）。                    |
| `type()`                     | 返回存储值的类型信息（`std::type_info` 对象），可用于类型判断。                       |
| `reset()`                    | 清空存储的值，使其变为 “空状态”。                                             |
| `std::any_cast<T>()`         | 提取存储的值：若类型匹配，返回 `T&` 或 `const T&`；否则抛出 `std::bad_any_cast` 异常。 |
对于小类型(int,double,string)，`any`对象会使用内部定义的`union`直接存储，对于大类型，动态使用new分配并保存指针。
**分配性能开销大，间接访问性能影响，运行时类型检查**

# 基准测试
**debug和release的代码输出不同，耗时不同**
使用计时类实现耗时
``` Cpp
#include <chrono>
//以下是一个计时类型，建立实例时计时，销毁实例时输出与前者的间隔
class Time{
public:
	Time(){
	m_StartTimepoint = std::chrono::high_resolution_clock::now();
	}
	~Time(){
		if(!m_stop)
			Stop();
	}
	void Stop(){
	auto endTimepoint = std::chrono::high_resolution_clock::now();
//转换m_StartTimepoint的时间单位为microseconds,.time_since_epoch获取纪元时间到当前时间的时间差,.count()获取时间间隔的数值部分
	auto start = std::chrono::timepoint_cast<std::chrono::microseconds> (m_StartTimepoint).time_since_epoch().count();
	auto end = std::chrono::timepoint_cast<std::chrono::microseconds> (m_StartTimepoint).time_since_epoch().count();
	auto duration = end - start;
	double ms = duration*0.001;
	std::cout<<duration << "us("<<ms<<"ms)/n"
	m_stop=true;
	}
}
private:
std::chrono::timepoint<std::chrono::high_resolution_clock> m_StartTimepoint;
bool m_stop;
```
## 可视化基准测试
[教学视频]( https://www.bilibili.com/video/BV1Dk4y1j7oj/?p=82&share_source=copy_web&vd_source=db8aceb651e9de7c3673c5eee54d917fj)
格式化并输出json文件并使用[edge](edge://tracing/)进行分析
![[Instrumentor.h]]
`InstrumentationTimer timer(名字)`创建带名字的计时器
使用宏来实现快速的调用和禁止
``` Cpp
#if PROFILING 1
//快速输入名字
#define PRONFILE_SCOPE InstrumentationTimer timer##__LINE__(name)
//直接获取当前函数
#define PROFILE_FUNCTION PRONFILE_SCOPE(__FUNCTION__);
//下面的宏获取函数签名
//#define PROFILE_FUNCTION PRONFILE_SCOPE(__FUNCSIG__);
#else
#define PRONFILE_SCOPE
#define PROFILE_FUNCTION 
#endif
```
可以实现自动输入所在犯法
# fstream
**需要使用\<fstream>库**
使用`ifstream`文件方法类构建一个文件流对象，对象的构造函数参数为需要打开的文件
例如:`std::ifstream stream(data.txt)`
	创建一个叫`stream`的在程序目录中`data.txt`文件流
可以通过指定参数选择读取方法：例如`std::ifstream stream(data.txt,ios::in)`以只读模式打开(默认)
	`ios::binary`二进制模式
	`ios:ate`在文件末尾打开
	`ios::app`追加模式，在文件末尾处写入

对`stream`使用对应方法以读取数据
	`std::getline(stream,line)`逐行读取stream赋值到``line，直到`EOF`
	`stream>>num`跳过空白字符(空格制表符，换行符)读取对应类型的数据并赋值
	`stream.get(c)`读取任意一个字符并赋值给c
	`stream.read(变量,typeinfo(类型))`读取相应字符数的内存，赋值到对应变量的内存
	

# 内存分配指示器
如例：
``` Cpp
struct AllocationMetrics{
	uint64_t TotalAllocated = 0;//分配的内存
	uint64_t TotalFreed = 0;//释放的内存
	//调用现分配内存的方法
	uint64_t CuttentUsage(){return TotalAllocated-TotalFreed}
}
static AllocationMetrics s_AllocationMetrics;//一个实例代表着分配
//使用静态方法，只在文件内调用，打印使用能存
static void PrintMemoryUsage(){
	std::cout<<"MemoryUsage:"<< s_AllocationMetrics.CuttentUsage()<< std::endl;
}
```
## 内存分配跟踪
通过重载new关键字，在方法内使用断点追踪外部代码使用到new的代码细节（因为通过new调用析构函数是编译器强制行为，重载只影响分配行为）
``` Cpp
void* operator new(size_t size)//可以获取到使用该字的字节数，并作为参数
{A
	s_AllocationMetrics.TotalAllocated += size;//加分配的内存
	return malloc(size);
}//返回void*指针，赋值时会隐式转换的
void* operator delete(void* memory,size_t size)//获取使用该字的内存指针
{
	s_AllocationMetrics.TotalFreed -= size;//加释放的内存
	free(memory);
}//相对应的释放内存，没有问题
```
用来减少对堆内存分配

# Optional(C++17)
**需要使用\<optional>库**
用来检测对应类型的值是否存在于实例的方法
`std::optional<int> value`表示value作为`optional`对象可能包含`int`值
有以下方法
	`.has_value()`返回bool值表示有无值
	`.value()`获取值
	`.value_or(default)`如果无法返回值，返回`default`值
	`.reset()`清空值
	可以使用`*`解引用获取值，但是无值会导致未定义行为
``
# 左值和右值
通常来讲，左值是在内存中有实际位置的值，右值是临时的，不占用内存空间的值（右值也因此不可被修改）

`const int&`:普通引用需要左值，但是使用`const`修饰的引用,可以引用右值Cpp会根据右值赋值一个临时左值能被其引用（方便传递），也符合不能修改右值的逻辑
`int&&`:右值引用，不能引用左值
	可以修改其值，读取其值的数据
	使用右值引用数组直接接管右值，减少拷贝（类拷贝）
	可以通过[[#重载函数]]针对右值进行优化操作
	引用被销毁时，其值也被丢弃（引用本质是左值）
## 移动语义
 函数之间的传递本质是复制，如果涉及需要创造堆内存的对象进行复制，那么调用函数意味着要分配堆内存
 而且通过方法（隐式转换）返回的临时对象也要进行复制/销毁
 以上的性能消耗可以通过移动语义解决
 简单的方法就是通过右值引用接管临时值，然后窃取右值的资源（不让右值释放需要的资源）
### std::move
`std::move(实例)`在编译时，根据传入的实例类型，返回相应实例的右值。我们可以通过`operator=`构造移动赋值运算符，让已有变量窃取右值资源，然后清空右值。
## C++五法则
1. 移动构造函数（使用右值构造）
2. 移动赋值运算符（使用右值赋值已有变量）
# 参数求值计算（C++17）
像`int i = a++ + a++`以前这种未定义行为,在C++17必须要逐个计算参数，即使未定义顺序
# noexcept（无异常修饰）
表示函数**绝对不会抛出任何异常**，所以编译器可以对此优化
如果该函数有异常风险会导致严重错误