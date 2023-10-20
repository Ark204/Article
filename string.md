**代码**
```C++
class string
{
public:
	string() { init_null_impl(); }//默认构造
	string(const char* newData) {
		init_impl(newData);//计算长度并复制
	}
	string(const string& newStr) {//复制构造 深复制
		init_impl(newStr.m_data,newStr.m_size);//调用指定长度重载版本，避免遍历
	}
	string(string&& newStr)noexcept :m_data(newStr.m_data),m_size(newStr.m_size)  {//移动构造
		newStr.init_null_impl();//主动为对方赋空
	}
	string& operator=(const string& cpy) {//赋值运算
		if (&cpy == this)return *this;//自我赋值
		delete[] m_data;//释放原来
		init_impl(cpy.m_data,cpy.m_size);//调用指定长度重载版本，避免遍历
		return *this;
	}
	string& operator=(string&& cpy) noexcept{//移动赋值
		if (&cpy == this)return *this;//自我赋值
		swap(cpy);//交换双方所有权---代替--->主动释放自身原数据、主动为对方赋空
		return *this;
	}
	~string() { delete[] m_data; }
	void swap(string& other) {//交换各自所有权，避免拷贝
		std::swap(other.m_data, m_data);
		std::swap(other.m_size, m_size);
	}
private:
	void init_null_impl() {//用于默认初始化||赋空
		m_size = 0;
		m_data = new char[1]{'\0'};
	}
	void init_impl(const char* newData) {//遍历计算长度并拷贝
		if (newData == nullptr) {//null
			init_null_impl();
			return;
		}
		//深拷贝
		m_size = std::strlen(newData);//遍历字符串
		m_data = new char[m_size + 1];//多留一位保留'\0'
		strcpy_s(m_data, m_size + 1, newData);//strcpy(tar,len,source)效率高于遍历复制
	}
	void init_impl(const char* newData,std::size_t size) {//指定长度
		if (newData == nullptr) {//null
			init_null_impl();
			return;
		}
		//深拷贝
		m_size = size;
		m_data = new char[m_size + 1];//多留一位保留'\0'
		strcpy_s(m_data, m_size + 1, newData);//strcpy(tar,len,source)效率高于遍历复制
	}
private:
	char* m_data;
	std::size_t m_size;
};
```
# 要点
<code>const char* cc="qwe";</code>
字符串常量存在于常量区，该指针指向对应常量区地址
+ **注意**：string中实际数据都在**堆**上，内部指针指向该堆数据


<code>m_data = new char[m_size + 1];</code>
+1额外分配一个单位存放'\0'


<code>std::strlen()</code>
**本质**：**遍历**字符串计数直到'\0'，返回长度不包括'\0'


## 底层拷贝函数(Cpy)
<code>strcpy(char* target,const char* source)</code>
**本质**：**逐字符**拷贝，遇到'\0'停止

<code>memcpy(void* target,const void* source,size_t len)</code>
**本质**：用于复制**任意类型内存块**，不仅限于字符串。可复制任何二进制数据，包括字符串
**效率**：通过**指定的len**，将source所指+len的内存块整个拷贝到target所指内存块，**效率高于strcpy**

<code>strcpy_s(char* target,size_t len,const char* source)</code>
**本质**：类似memcpy方式拷贝字符串，**效率也相似**

<code>init_impl(const char* newData,std::size_t size)</code>
**优化点**：使用指定的len为字符串分配内存，避免调用<code>strlen()</code>遍历字符串

<code>swap(string& other)</code>
**优化点**：内部交换各自**值**，**避免深复制**

```C++
string& operator=(string&& cpy) noexcept{//移动赋值
	if (&cpy == this)return *this;//自我赋值
	swap(cpy);//交换双方所有权---代替--->主动释放自身原数据、主动为对方赋空
	return *this;
}
```
**优化点**：交换双方所有权---代替--->主动释放自身原数据、主动为对方赋空
+ 自身原数据：转交给对方后由对方析构函数释放(对方析构更有意义)
+ 不为对方赋空：减少赋空操作的内存分配
