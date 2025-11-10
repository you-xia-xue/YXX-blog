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