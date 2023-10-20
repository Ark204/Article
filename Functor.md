# 可调用实体
1. (普通/成员)函数指针
2. 普通函数
3. 可转化为函数指针的类对象
4. 重载了operator()的类对象(函数对象)
# Function Object(函数对象)
**本质**：类重载了operator()
**优点**：
1. 有**成员**->有**状态**
2. 特有类型
3. 执行速度比funtion pointer快

**传递方式**：默认by-value
+ 显式指定为ref类型
	```C++
	class MyFun{...};//函数对象
	MyFun func;//函数对象实例
	auto it=find_if<decltype(c):iterator,&MyFun>//显式指定参数
	(c.cbegin(),c.cend(),func);//此时func按引用传递
	```
+ for_each()返回函数对象
	```C++
	class MyFun{...};//函数对象
	MyFun back=for_each(c.begin(),c.end(),MyFun());//MyFun构造函数
	back.value()//从返回的函数对象获取数据
	```

# binder(Function Adapter)
**作用**：将函数的参数**绑定->减少**或**占位->保留**，并返回新的函数
```C++
std::bind(std::plus<int>(),//构造函数对象(要绑定的函数)
std::placeholders::_1,//占位，表第一个参数保留未知
10);//绑定第二个参数为10
```
**返回类型**：Binder模板类->根据调用bind()**实参**有相应返回类型
**绑定成员函数**
```C++
class Person{
  public: void print();//本质为print(Person& p);
};
int main{
   auto fun=bind(&Person::print,_1);//保留_1占位，Person所有实例通用版
   Person p,p2;
   fun(p);//fun(Person& p); =>p.print();
   fun(p2);//fun(Person& p); =>p2.print();
   auto fun_p=bind(&Person::print,&p);//绑定第1参数，仅实例p版本
   func();//func(); =>p.print();
}
```
1. 由于成员函数**不允许隐式转换**为func ptr，所以需显式(\&)取地址
2. _1保留占位版本只是将print(Person& p)原型显现出来
3. &p绑定版本则将print(Person& p)的第一参数绑定为p
**注意**：bind()绑定参数方式是**by-value**，若要绑定ref，可用<code>ref()</code>或<code>cref()</code>

# Lambda
**优点**：快速
<code>[&]</code>：by-ref传入
<code>[=]</code>：by-value传入
<code>\[sum\]()matable{...};</code>：by-value传入初始化，**所有**Lambda内部共享(类似类static成员)
**返回类型**：每个Lambda有自己的类型
## Lambda vs 函数对象
**处理状态需求**
1. 1Lambda+捕获
	```C++
	int t=10;
	auto bigger=[&t](int i){//使用&捕获(by-ref)
   	return i>t;
	};
	find_if(c.cbegin(),c.cend(),bigger);//找>10的elem
	t=20;//改变t值
	find_if(c.cbegin(),c.cend(),bigger);//找>20的elem
	```
	+ <font color=red>注意</font>：**必须**使用**&捕获**
		Lambda在定义同时，仅初始化**一次**需捕获参数，在后续调用同一个Lambda时，使用的都是**定义时**初始化的捕获参数
	+ 局限：依赖局部数据
2. 构造两个Lambda
	```C++
	find_if(c.cbegin(),c.cend(),[](int i){return i>10;});//找>10的elem
	find_if(c.cbegin(),c.cend(),[](int i){return i>20});//找>20的elem
	```
**优劣**
1. 需要类型时，函数对象更优
2. 函数对象适用**全局性**，Lambda适用**局部性**
3. 面对**状态**，函数对象使用**成员**，Lambda使用\[&=\]

## Lambda vs binder
**处理bind需求**：
```C++
auto func=[](int i){//绑定plus(a,b)第二实参为10
   plus<int>(i,10);
}；
```
**优劣**
1. Lambda更简便

# function<>函数包装器
**作用**：通用**可调用实体**包装器
**特点**：只要可调用实体**参数列表与返回类型均相同**，类型**一致**
+ 解决模板低效性
	```C++
	template <typename T,typename F>
	T use_f(T t,F func){return func();}
	//当F实参为Lambda、普通函数、bind()函数对象、函数对象等，会生成多个模板实例
	```
	1. 在传入模板前，用function**包装**可调用实体再传入，只生成一个模板实例
	2. 将模板函数改为<code>T use_f(T t,function<T(T)> func)</code>

## function vs 函数指针
1. function代码简洁
2. function可以使基类**正确调用虚函数**
3. function可接收的范围**更广**
