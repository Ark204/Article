# RTTI与_cast转换
## RTTI
### 工作原理
1. **dynamic_cast**：仅继承中**安全**地上下转换
2. **typeid()**：返回参数**类型**，类似C#中typeof，常用==判断是否**完全**同类
3. **type_info**：相应类的信息，类似C#中Type

## auto
**功能**：**编译期**自动推导类型
**推导值**
```C++
const int a=10;
auto b=a;//b为non-const int,此处相当于拷贝
```
**推导指针/引用**：与模板推导一致，会保留前置const,省略后置const
**作用**：
1. 泛型编程：不知道，不需知道类型
	```C++
	template <typename T>
	void Func(void){
	   auto ret=T::Get();
	   cout<<ret<<endl;
	}
	```
2. 简化代码(声明迭代器)
3. 搭配decltype推导**返回类型**(C++14可以不加decltype)
	+ 注意：声明返回类型时，会舍弃ref与const
4. 声明**值**模板
	```C++
	template<auto data>//取代template<int data>、template<double data>
	bool Bigger(int i);
	```

**限制**
1. **必须**初始化
2. 不能声明**函数参数**(可以声明Lambda函数参数)
3. 不能声明类**非static**成员
4. 不能声明数组
5. 不能用于模板参数<code>set\<auto\> s=set\<int\>;</code>
6. vector\<bool\>::iterator(实际上是代理类)
7. 返回/参数类型为**大括号初始物**无法推导
	+ <code>auto func(){return {1,2,3}; }</code>无法通过编译
		```C++
		auto l=[](auto init){};
		l({1,2,3});//{1,2,3}为init_list 编译失败
		```

## decltype
+ 处理变量：**全保留**原类型(与<code>auto</code>不同,不省略后置const)
+ 处理函数：获取函数返回类型，不调用
+ 处理表达式：
	1. 若表达式的值类型为纯右值，则推导出T
	2. 若表达式的值类型为左值：若表达式只是变量名，则推导出T；其他情况推导出T&
	3. 若表达式的值类型为将亡值，则推导出T&&

**简化代码：**
```C++
std::unordered_map<std::string,float> m;
decltype(m)::value_type elem;//decltype(m)==std::unordered_map<std::string,float>这一大串
//同时在decltype(m)后使用‘::’可以访问类内定义的类型
```
**声明Lambda类型**：需要将Lambda作为模板参数时，用decltype声明其类型
```C++
auto op=[](const int& i1,const int& i2){return i1<i2};
set<int,decltype(op)> s(10,op);//Lambda无构造，必须传Lambda
```
**推导返回类型**：与auto搭配可以推导返回值的类型
**防止返回类型丢失**
```C++
decltype(auto) Get(vector<int>& v,int i){
   return v[i];//v[i]返回类型为int&,若只使用auto,返回类型为int
}
```
## type Trait(类型萃取)
在**编译**期根据一个/多个template实参，产出一个type或**value**，是泛型编码的**基石**
**处理共通类型**：common_type<T>
```C++
template <typename T1，typename T2>
??? min(const T1& t1,const T2& t2);//返回类型无法确定
//解决方案：
typename common_type<T1,T2>::type min(const T1& t1,const T2& t2);
//typename 用来指定其后跟着的是类型而不是静态变量(MClass::member可能为变量)
```
**改动类型**：remove_reference<T>
```C++
template <typename T>
T&& forward(remove_reference<T>& left){
   return static_cast<T&&>(left);
//remove_reference<T>还原T最原始的类型，再加上&
}
```
+ **注意**：与形参使用T&相比，该方式**禁用**了模板自动推导=>必须**显式**指定T

**蛀蚀**：dacay<T>
1. 将"by value"传入的类型转为相应类型(array/funtion->pointer)
2. 把lvalue转为rvalue---包括**移除**const与volatie
```C++
template <typename T1,typename T2>
pair<decay<T1>,decay<T2>> make_pair(T1 t1,T2 t2);
auto p=make_pair("123","45");//实参类型为const char[3]、const char[2]
cout<<typeid(p).name<<endl;//返回的类型为pair<const char*,const char*>
```
**迭代器类型萃取**：iterator_traits
```C++
template<typename Iter>
void search(Iter it){
    typename iterator_traits<Iter>::iterator_category cateEntity;
	//通过iterator_traits<Iter>萃取出Iter的迭代器类型
}
```
+ 为什么不直接通过<code>Iter</code>获取?
+ 答：如果Iter是**指针**，则无法获取，iterator_traits作为中间层，通过**偏特化**的方式提供指针的迭代器类型

## _cast类型转换
**统一形式**：接收对象=XX_cast<类名>(要转换对象);
### dynamic_cast
仅继承中**安全**地上下转换，尝试转换，若失败，对于指针返回**空**，对于引用抛出**bad_cast**异常
**适用范围**：有继承关系**指针**或**引用**

### const_cast
可以改变const或volatile特征，使返回const引用的保护手法被**绕过**，但是没办法修改**真正**const的值
**适用范围**：**指针**或**引用**以及<font color=red>指向对象成员</font>的指针
**注意**：用于指针时**前**置const与**后**置const可以自由转换：const int*->int* const

### static_cast
可相互**隐式**转换类型以及继承中：可以上下转换(仍**不安全**)，也可以转换int与enum
**本质**：**重新解释**该内存数据类型，并将内存内容转化为相应数据类型
**适用范围**：所有

### reinterpret_cast
用于天生危险转换，不允许删除const
**本质**：在**不改变内存内容**的情况下，**重新解释**该内存数据类型
**使用范围**：所有
