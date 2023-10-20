# const
## const与指针/引用
1. const在前：指针/引用所指物不可写(C#中:Interface)
2. const在后：指针(对引用不影响)本身不可写(C#中:readonly reference)

## const与函数
### const+类指针/引用前置修饰(括号前修饰返回值)
例：<font color=red>const</font> Item& GetItem();
**作用**：使返回的指针/引用**所指**物不可写(仍可以被类型转换**绕过**)
**应用**：当类需提供public接口向外部返回一个成员引用/指针(供外部读取**成员的成员**信息)时，将其修饰为所指物const,可以<font color=red>**防止**</font>外界通过该引用/指针修改成员而间接破坏原类对象内容

### const后置修饰函数(括号后修饰this形参)
例：int GetInt() <font color=red>const</font>;
**作用**：允许const对象或指针/引用所指的const对象<font color=red>**也能**</font>调用该函数
注意：const对象不能调用没有const后置修饰函数，普通对象可以调用所有public函数
**应用**：修饰**读取**public接口，使其在任何外部调用时都可读
**本质**：const修饰了隐藏的this实参
```
int GetInt(const MyClass& this/*隐藏的this引用*/){return this.m_int;}
```

### const中置修饰函数(括号中修饰形参)
例：void ReadItem(<font color=red>const</font> Item& item);
**作用**：
1. 防止函数中通过传入的引用修改数据
2. 即使是const实参，也可以处理

**应用**：需要传入某个引用读取其内部数据时，可以防止在函数中修改内部数据，并且也可以读取const实参数据
