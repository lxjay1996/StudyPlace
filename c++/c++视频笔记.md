# C++

## $ 进程虚拟地址空间划分

- `.text`: 存放指令
- `.rodata`: read only data 只读数据段，存放字符串常量
- `.data`: 存放初始化后值不为零的变量
- `.bss`：存放未初始化或初始化后为0的变量
- `.heap`:
- `.stack`:
- 每一个进程的用户空间是私有的，但是内核空间是共享的
- 进程之间的通信方式之一：匿名管道通信（利用内核空间的共享区域）

![q](./images/进程的虚拟空间地址.png)

## $ 函数的栈调用

- 参数压栈、函数栈帧的开辟和回退，存在着函数调用的开销
- 先把函数的参数压栈
- 调用函数前先把当前地址入栈
- 遇到函数的左括号，把main函数的栈帧ebp指针入栈，再为被调用的函数开辟新的栈帧，执行函数
- 遇到函数结束的右括号，释放申请的栈帧空间，出栈ebp，返回main函数的栈帧

## $ 基础

### $$ 形参带默认值的函数

1. 给默认值的时候，从右向左给
2. 调用效率的问题
3. 定义处可以给形参默认值，声明处也可以给形参默认值
4. 不管定义处给，还是声明处给，每个参数的默认值只能出现一次

### $$ 内联函数

- 函数代码简单短小，经常调用，就应该设置为内联函数

1. 与普通函数的区别
    在编译过程中，就没有函数的调用开销了，在调用处直接把函数的代码块复制过去替换，不再生成相应的函数符号。
2. 但是不是所有的inline都会被编译器处理成内联函数-递归
3. inline只是建议编译器把该函数处理成内联函数
4. debug版本上，inline不起作用，inline只有在release版本下才能出现

### $$ 函数重载

- 一组函数，函数名相同，但是参数列表的个数或者类型不同，那么这一组函数就称为函数重载。
- 一组函数要称得上是重载的话，必须是处在同一作用域下
- const或volatile的时候，是怎么影响形参类型的：顶层const不会改变形参类型，底层const会影响。
- 一组函数，函数名和参数列表都相同，仅仅是返回值不同，不叫重载。
- 请你解释一下什么叫多态
  - 静态（编译时期）的多态：函数重载就是之一，还有模板。。。
  - 动态（运行时期）的多态：

1. 为什么C++支持重载，C不支持
    因为C++产生函数符号的时候，是由函数名+参数列表类型来组成的；而C产生函数符号只由函数名来产生。总之就是编译器产生函数符号的规则不同。
2. 函数重载需要注意些什么
3. C++和C之间如何互相调用
    - C调用C++:也是无法直接调用，要把被调用的部分在C++源文件中用`extern "C" {函数定义}`括起来，这样这部分就会用C编译器编译，C可以直接调用了
    - C++调用C：无法直接调用，要用`extern "C" {C 函数的声明}`告诉编译器按照C编译器进行编译该处代码

### $$ const 指针 引用 在函数中的引用

1. const怎么理解？C和C++中的const区别？
    const修饰的变量不能再作为左值进行赋值！！！初始化完成后，值不能被修改！！  
    const的编译方式不同，C中，const就是当作一个变量来编译生成指令的；C++中，所有出现const常量名字的地方，都被常量的初始化替换掉了，编译期就完成了替换。  
    C中的const 修饰的量，可以不用初始化，所以C中const修饰的不叫常量，叫做常变量，可以通过指针等方式来修改，不能当做常量来使用  
    C++中的const必须初始化，且初始值是一个立即数，叫作常量；如果初始值不是一个立即数，那么就是一个常变量，是一个变量。

2. const和一级、多级指针的结合

    - const修饰的量常出现的错误是：
      - 常量不能再作为左值  <--- 可以直接修改常量的值
      - **不能把常量的地址泄露给一个普通的指针或普通的引用变量** <--- 可以间接修改常量的值
    - const与指针的类型转换：
      - `int* <= const int*`   是错误的
      - `const int* <= int*`   是正确的
      - 多级指针必须两边都有const才正确
      - `int** <= const int**` 是错误的
      - `const int** <= int**` 是错误的
      - `int** <= int*const*`  是错误的，是一级指针的转换:`* <= const*`
      - `int*const* <= int**`  是正确的，是一级指针的转换：`const* <= *`
      - **const如果右边没有指针*的话，const是不参与类型的**

3. 引用

- 引用是一种更加安全的、弱化的指针，是变量的别名

- 引用和指针的区别：
  - 引用必须初始化，指针可以不初始化
  - 定义一个引用变量，和定义一个指针变量，在汇编层面是一样的， 引用和指针的汇编代码一样；通过引用修改引用内存的值，和通过指针解引用修改指针所指向的内存的值，其底层汇编指令也是一样的。
  - 引用只有一级引用，没有多级引用；指针可以有一级指针，也可以有多级指针
  - 引用从一而终，始终代表了初始化的对象；指针可以更换指向的对象
  
  ```cpp
    int arary[5] = {1, 2, 3, 4, 5};
    int *p = array;
    // int (*q)[5] = &array; // 数组指针
    int (&q)[5] = array; // 对数组的引用
    sizeof(array); // 20
    sizeof(p); // 4
    sizeof(q); // 20
  ```

- 左值引用和右值引用
  - 左值：有内存具名的变量，值可以修改
  - 右值：没有内存，没名字
  - 右值引用：专门用来引用右值类型，指令上，可以自动产生临时变量，然后对该临时量进行引用；一个右值引用变量本身是一个左值,已经有内存具名了；`int &&c = 2; c = 3; int d = c;`；不能对左值进行右值引用。

### $$ new和delete

- new和delete，称作运算符
- malloc和free，称作C的库函数

- new和malloc, delete和free 的区别
  - malloc是按照字节来开辟内存，需要指定开辟多大字节数的内存，函数返回类型是void*, 需要进行转换为其他类型；`int *p = (int*)malloc(sizeof(int) * 5);`
  - 操作符new，通过直接指定类型来开辟内存，不用进行类型转换；new不仅可以做内存开辟，还可以做内存初始化操作。`int *p1 = new int[5]();`
  - 对于开辟内存失败的判断不一样：malloc开辟内存失败，是通过返回值和nullptr作比较；而new开辟内存失败，是通过抛出bad_alloc类型的异常来判断
  - new做两件事，一是分配内存，二是调用类的构造函数；同样，delete会调用类的析构函数和释放内存。而malloc和free只是分配和释放内存。
  - new建立的是一个对象，而malloc分配的是一块内存
  - new/delete是保留字，不需要头文件支持；malloc/free需要头文件库函数支持
  - new和delete配套使用，new[]和delete[]配套使用

- new有多少种
  
    ```cpp
    int *p1 = new int(20);      // 可以抛出异常的new
    int *p2 = new (nothrow) int;    // 不抛出异常的new
    const int *p3 = new const int(5);   // 申请存放常量类型空间的new
    定位new
    int data = 0;
    int *p4 = new (&data) int(50);
    ```

- new 和 delete可以混用吗？为什么要区分单个元素和数组的内存分配和释放呢？
  - 对于普通的内置类型，没有构造函数，new最后只是调用melloc进行单纯的内存分配，不涉及对象的构造和析构，所以此时`new/delete[]`、`new[]/delete`可以混用,但是最好不要
  - 对于自定义的类类型，有析构函数，为了正确调用析构函数，那么开辟对象数组的时候，会多开辟4个字节，记录对象的个数

## $ OOP面向对象

### $$ OOP编程  this指针

- OOP思想
  C: 各种各样函数的定义，struct  
  C++: 类 =》实体的抽象类型；  
  实体（属性、行为） =》 ADT(adstract data type);  
  对象（代表着实体） 《= （实例化）类（属性=》成员变量  行为=》成员方法）  
  属性一般是私有的成员变量，给外部提供公有的成员方法，来访问私有属性  

类不占用空间，对象才占用内存（栈或堆上构造对象）,对象占用内存的大小只与成员变量有关  
类内部实现的方法，自动处理为inline内联函数；类外定义的方法就是普通的方法，要想成为内联，必须要加上inlne  
**类可以定义无数的对象，每一个对象都有自己的成员变量，但是它们共享一套成员方法；调用成员方法时，都会自动加上一个this指针参数，把对象的地址传给成员方法，告诉成员方法是哪个具体对象的调用;** this指针就是用来区分当前类实例化的不同对象  

- OOP语言的四大特征
  抽象   封装/隐藏   继承   多态

### $$ 构造函数  析构函数

- 函数的名字和类名一样，没有返回值
- 构造函数可以带有参数，析构函数没有参数，所有构造函数可以有多个，叫做构造函数重载，而析构函数没有参数，只能有一个
- 不用手动调用，在定义对象时（先开辟内存，再调用构造函数），就会自动调用构造函数进行构造初始化，同理，在出了作用域或delete对象时，会自动调用析构函数来销毁对象（先调用析构函数，再释放内存）
- 析构函数可以手动调用，但是调用以后，对象的资源已经被释放了，只是把对象内部指针指向的内存释放掉，指针本身还没有释放，再调用成员函数的话，可能会对该指针进行操作，会造成内存的非法访问

- .data段上的对象：程序启动时构造，结束时析构
- heap段上的对象：new时构造，delete时析构
- stack段上的对象:进入了函数局部作用域时，遇到定义的地方构造，出了函数进行析构

### $$ 对象的浅拷贝和深拷贝

- 对象的默认拷贝构造是做内存的数据拷贝，不一定有错。但是，如果对象占用了外部的资源（比如说**对象中有指针指向了对象之外的外部资源** ，浅拷贝只会拷贝对象内部的数据，而不会为新对象内的指针开辟新的内存，这样多个指针就会指向同一块内存，这样当释放指针内存的时候，就会造成野指针，指向已经被释放了的内存），那么浅拷贝就错了。
- 所以**当类中有分配外部资源的时候（比如有指针），一定要重写拷贝构造、拷贝赋值和析构三个函数** ，避免浅拷贝。重写拷贝构造函数，把分配的外部资源也拷贝一份过来，不会造成野指针，各个指针指向自己的内存；重写拷贝赋值函数，当原对象赋值给另一个已存在对象后，要先把被赋值对象的空间给释放掉（避免内存泄露），再像拷贝构造那样开辟新的空间进行赋值。拷贝赋值与拷贝构造相比多一步对被赋值对象内存的释放，因为赋值是两个已经存在的对象之间进行。
- 拷贝赋值时一定要避免自我赋值，你想啊，赋值前要先释放被赋值对象的空间，你都把自己给释放了，还怎么进行赋值，所以一定要在拷贝赋值函数中检测是否进行了自我赋值（传入的参数==this?）,防止数据被删除
- 小结拷贝赋值函数过程：
  - 防止自我赋值
  - 释放被赋值对象的外部资源
  - 最后开辟内存进行赋值（类似于拷贝构造函数）

### $$ 手动实现一个自己的 string 类

- **对象的拷贝尽量不用memcpy和realloc** , 会造成浅拷贝，因为两个函数只是做内存的直接拷贝，不会把对象所占用的外部资源也拷贝，所以当对象成员变量中有指针的时候，会造成多个对象指向同一个外部资源，形成野指针。
- 迭代器的功能：提供一种统一的方式，来透明的遍历容器

```cpp
class MyString {
public:
  // 自定义了普通构造函数，就不会产生默认构造函数
  MyString(const char *str = nullptr) {
    if(str != nullptr) {  // 判断是否传入了空指针
      m_data = new char[strlen(str)+1]; // 开辟内存，strlen计算长度不带"\0"
      strcpy(this->m_data, str);  // 字符串复制过来
    } else {  // 如果传入的字符串指针为空，开辟一个字节空间，存放'\0'
      m_data = new char[1];
      *m_data = '\0';
    }
  }
  // 拷贝构造函数
  MyString(const MyString &other) {
    // 这里就不用判断other.m_data指针是否为空了，因为上面的构造函数保证了m_data一定指向一块内存，可以放心的进行复制
    m_data = new char[strlen(other.m_data)+1];
    strcpy(m_data, other.m_data);
  }  
  // 析构函数
  ~MyString() {
    delete[] m_data;
    m_data = nullptr; // 防止野指针出现
  }  
  // 重载赋值函数
  // 返回MyString& 是为了支持连续的operator= 操作
  MyString& operator=(const MyString &other){
    if(this == &other) return *this; // 自我赋值检测
    delete[] m_data;  // 把当前被赋值对象的所指向的内存给释放掉，避免内存泄露
    // 类似于拷贝构造，开辟内存，进行复制
    m_data = new char[strlen(other.m_data)+1];
    strcpy(m_data, other.m_data);
    return *this;
  }
  // >/>=/<=/</== 关系运算符重载函数
  bool operator>(const MyString& rhs)const {
    return strcmp(m_data, rhs.m_data)>0;
  }
  bool operator>=(const MyString& rhs)const {
    return !(strcmp(m_data, rhs.m_data)<0);
  }
  bool operator<=(const MyString& rhs)const {
    return !(strcmp(m_data, rhs.m_data)>0);
  }
  bool operator<(const MyString& rhs)const {
    return (strcmp(m_data, rhs.m_data)<0);
  }
  bool operator==(const MyString& rhs)const {
    return (strcmp(m_data, rhs.m_data)==0);
  }
  // size()函数，返回大小，不包括'\0'
  int size()const { return strlen(m_data); }
  // []操作符重载，支持随机访问
  char& operator[](int index) { return m_data[index]; }
  const char& operator[](int index)const { return m_data[index]; }
  // 返回C风格字符串，返回的是一个指针，指向一个由空字符结尾的字符数组
  // 指针的类型是const char*，确保不会改变字符数组的内容
  const char* c_str()const { return m_data; }

  // 给字符串类型提供迭代器的实现,指向的是容器底部元素的位置
  class iterator {
  public:
    iterator(char* p = nullptr) : _p(p) {}
    // 运算符重载: !=  前置++  提领*
    // 一般来说迭代器实现前置++就行了，不会产生临时量，直接返回
    bool operator!=(const iterator& it) {return _p != it._p;}
    void operator++() {++_p;}
    char& operator*() {return *_p;}
  private:
    char* _p;
  }
  // begin返回容器底层首元素的迭代器,end返回容器最末元素后继位置的迭代器
  // 注意begin和end都是MyString的方法，而不是迭代器的方法
  iterator begin() {return iterator(m_data);}
  iterator end() {return iterator(m_data + size());}

private:
  char *m_data; //用于保存字符串
  friend ostream& operator<<(ostream& out, const MyString& rhs);
  friend MyString operator+(const MyString& lhs, const MyString& rhs);
};

// +运算符重载函数  要写在全局作用域
MyString operator+(const MyString& lhs, const MyString& rhs) {
  // 需要开辟足够的内存空间来存放拼接后的字符串
  MyString tmp; // 出了作用域，tmp会自动被析构，释放内存
  tmp.m_data = new char[strlen(lhs.m_data) + strlen(rhs.m_data) +1];
  strcpy(tmp.m_data, lhs.m_data);
  strcat(tmp.m_data, rhs.m_data);
  return tmp;
}
// <<输出流运算符重载
ostream& operator<<(ostream& out, const MyString& rhs) {
  out << rhs.m_data;
  return out;
}

int main() {
  // 调用普通构造函数
  MyString str1;
  MyString str2("hello");
  MyString str3 = "world";

  // 调用拷贝构造函数
  MyString str4 = str3;
  MyString str5(str3);

  // 调用重载赋值函数
  /*
  str1 = str2
  str1.operator=(str2) ----> MyString& str1
  str3 = str1
  */
  str3 = str1 = str2;

  return 0;
}
```

### $$ 构造函数初始化列表

- 构造函数初始化列表可以指定成员变量的初始化方式
- 成员变量的初始化和它们定义的顺序有关，和构造函数初始化列表中出现的先后顺序无关。

### $$ 类的成员方法/变量

- **核心就是this指针**

1. 普通成员方法 =》 编译器会添加一个`class*this`形参变量
   - 属于类的作用域
   - 调用依赖一个对象（常对象是无法调用的 实参：`const class*`, 而普通成员方法的形参：`class*`）
   - 可以任意访问对象的私有成员
2. 静态成员方法 =》 不会生成this指针形参
   - 普通成员方法有this指针，**静态成员方法没有this指针** ，所以不需要通过对象来访问，直接用类名作用域来访问`class::静态方法`，也因此静态成员方法只能访问静态成员
   - 类中的静态成员变量/方法，属于类，而不属于某一个对象，为所有对象所共有
   - 类中的静态成员变量，必须类中声明，类外定义
3. 常成员方法 =》 生成一个`const class* this`指针的形参
   - 只要是只读操作的成员方法，一律实现为const常成员方法
   - 属于类的作用域，调用依赖一个对象、普通对象或常对象都可以
   - 可以任意访问对象的私有成员，但是只能读， 不能写

### $$ 指向类成员（成员方法、成员变量）的指针

- 指向类成员变量的指针：要加类的作用域，通过实例化的对象来调用
- 指向类成员方法的指针：要加类的作用域，通过实例化的对象来调用
- 指向类静态成员变量/方法的指针：只要加类的作用域

```cpp
// 指向类成员变量的指针
int Test::*p = &Test::data;
t1.*p = 10;
t2->*p = 20;
// 指向类成员方法的指针
void (Test::*pfun)() = &Test::show;
(t1.*pfun)();
(t2->*pfun)();
// 指向类静态成员变量/方法的指针
int Test::*p = &Test::count;
cout << *p;
```

## $ 模板编程

### $$ 函数模板

- 模板的意义：对类型也可以进行参数化了
- 模板的实参推演：如果用户没有指定类型参数，编译器可以根据用户传入的实参类型，来推导出模板类型
- 模板的非类型参数：必须是整数类型（整数或地址/引用都可以）都是常量，只能使用，而不能修改
- **函数模板** 是不会进行编译的，因为编译器不知道模板参数的具体类型，只有在调用点指明了参数类型后（或者经过模板的实参推演），才能从原模板进行实例化，编译为具体的**模板函数**。
  - 函数模板：不编译，因为类型还不知道
  - 模板的实例化：函数调用点进行实例化
  - 模板函数：才是被编译器所编译的
- 模板特例化：特殊的实例化（不是编译器提供的,而是用户提供的）
  - 对于某些类型来说，依赖编译器默认实例化的模板函数，代码处理逻辑是错误的，所以用户要提供特定类型的模板函数。常见的就是字符串类型，需要特殊处理
- 函数模板、模板的特例化、非模板函数（普通函数）它们不是重载关系，因为模板特例化的函数名字与其他的不一样
- 模板代码是不能在一个文件中定义，在另外一个文件中使用的。模板代码在调用之前，一定要看到模板定义的地方，这样，模板才能进行正常的实例化，产生能够被编译器编译的代码。**所以，模板代码一般都是放在头文件中的，然后在源文件中进行#include包含**

### $$ 类模板

- 类模板 =》 实例化 =》 模板类
- 模板名称 + 类型参数列表 = 类名称
- 构造、析构函数的`<T>` 可以省略

手写一个简单的vector:

- 容器的构造应该把开辟内存和构造对象分开
- 容器的删除元素，应该只需要析构对象，而不要释放内存
- 所以容器的开发中，不能直接使用new和delete
- 容器的空间配置器allocator做四件事：内存开辟/内存释放   对象构造/对象析构

```cpp
template<typename T>
class vector {
public:
  vector(int size = 10) { // 普通构造函数
    _first = new T[10]; // new即开辟内存，又构造了对象，最好把开辟内存和构造对象分开
    _last = _first;
    _end = _first + size;
  }
  ~vector() {     // 析构函数
    delete[] _first;
    _first = _last = _end = nullptr;
  }
  // 拷贝构造函数
  vector(const vector<T>& rhs) {
    int size = rhs._end - rhs._first;
    int len = rhs._last - rhs._first;
    _first = new T[size];
    _last = _first + len;
    _end = _first + size;
    // 元素拷贝过来
    for(int i=0; i<len; i++) _first[i] = rhs._first[i];
  }
  // 赋值操作函数，可以连续赋值
  vector<T>& operator=(const vector<T>& rhs) {
    // 防止自我赋值
    if(this == rhs) return *this;
    // 释放被赋值对象申请的内存
    delete[] _first;
    // 拷贝构造
    int size = rhs._end - rhs._first;
    _first = new T[size];
    int len = rhs._last - rhs._first;
    for(int i=0; i<len; i++) _first[i] = rhs._first[i];
    _last = _first + len;
    _end = _first + size;

    return *this;
  }

  void push_back(const T& val) {
    if(full()) expand();
    *_last++ = val;
  }
  void pop_back() {
    if(empty()) return;
    _last--;
  }
  T back() const {
    return *(_last-1);
  }
  bool full() const { return _last == _end; }
  bool empty() const { return _last == _first; }
  int size() const { return _last - _first; }
private:
  T *_first;  // 指向数组的起始位置
  T *_last;   // 指向数组中有效元素的后继位置
  T *_end；   // 指向数组空间的后继位置
  // 容器的二倍扩容
  void expand() {
    int size = _end - _first;
    T* ptmp = new T[2 * size];
    for(int i=0; i<size; i++) ptmp[i] = _first[i];
    delete[] _first;
    _first = ptmp, _last = _first + size, _end = _first + 2*szie;
  }
};
```

```cpp
// 加入空间配置器
// 定义容器的空间配置器，和C++标准库的allocator实现一样
template<typename T>
class Allocator {
  // 负责内存开辟
  T* allocate(size_t size) return (T*)malloc(sizeoof(T) * size);
  // 负责内存释放
  void deallocate(void* p) free(p);
  // 负责对象构造
  void construct(T* p, const T& val) new (p) T(val);  // 定位new
  // 负责对象析构
  void destroy(T *p) p->T(); // ~T()代表T类型的析构函数
}

// 带有空间配置器的vector
template<typename T, typename Alloc = Allocator<int>>
class vector {
public:
  vector(int size = 10) {
    _first = _allocator.allocate(size); // 开辟内存
    _last = _first;
    _end = _first + size;
  }
  ~vector() {
    for(T* p=_first; p!=_last; p++) _allocator.destroy(p); // 析构容器有效元素
    _allocator.deallocate(_first);// 然后释放_first指针指向的堆内存
    _first = _last = _end = nullptr;
  }
  vector(const vector<T>& rhs) {
    int size = rhs._end - rhs._first;
    int len = rhs._last - rhs._first;
    _first = _allocator.allocate(size);
    _last = _first + len;
    _end = _first + size;

    for(int i=0; i<len; i++) _allocator.consttruct(_first+i, rhs._first[i]);
  }
  
  vector<T>& operator=(const vector<T>& rhs) {

    if(this == rhs) return *this;

    // delete[] _first;
    for(T* p=_first; p!=_last; p++) _allocator.destroy(p); // 析构容器有效元素
    _allocator.deallocate(_first);// 然后释放_first指针指向的堆内存

    int size = rhs._end - rhs._first;
    // _first = new T[size];
    _first = _allocator.allocate(size); // 开辟内存
    int len = rhs._last - rhs._first;
    // for(int i=0; i<len; i++) _first[i] = rhs._first[i];
    for(int i=0; i<len; i++) _allocator.consttruct(_first+i, rhs._first[i]);
    _last = _first + len;
    _end = _first + size;

    return *this;
  }

  void push_back(const T& val) {
    if(full()) expand();
    // *_last++ = val;
    // _last 指针指向的内存构造一个值为val的对象
    _allocator.construct(_last, val);
    _last++;
  }
  void pop_back() {
    if(empty()) return;
    //_last--;
    // 不仅要把_last指针--, 还需要析构删除元素
    _last--;
    _allocator.destroy(_last);
  }
  T back() const {
    return *(_last-1);
  }
  bool full() const { return _last == _end; }
  bool empty() const { return _last == _first; }
  int size() const { return _last - _first; }

  // 运算符重载
  T& operator[](int index) {
    if(index<0 || index>=size()) throw "OutOfRangeException";
    return _first[index];
    }

  // 迭代器一般实现成容器的嵌套类型
  class iterator {
  public:
    iterator(const T* p) : _p(p) {}
    // 运算符重载
    bool operator!=(const iterator& rhs)const {return _p != rhs._p;}
    void operator++() {return ++_p;}
    T& operator*() {return *_p;}
    const T& operator*()const {return *_p;}
  private:
    T* _p;
  }
  iterator begin() {return iterator(_first);}
  iterator end() {return iterator(_last);}
private:
  T *_first;  // 指向数组的起始位置
  T *_last;   // 指向数组中有效元素的后继位置
  T *_end；   // 指向数组空间的后继位置
  Alloc _aloocator; // 定义容器的空间配置器对象
  // 容器的二倍扩容
  void expand() {
    int size = _end - _first;
    // T* ptmp = new T[2 * size];
    T* ptmp = _allocator.allocate(2*size);
    // for(int i=0; i<size; i++) ptmp[i] = _first[i];
    for(int i=0; i<size; i++) _allocator.construct(ptmp+i, _first[i]);
    // delete[] _first;
    for(T *p = _first; p!=-last; ++p) _allocator.destroy(p);
    _allocator.deallocate(_first);
    _first = ptmp, _last = _first + size, _end = _first + 2*szie;
  }
};
```

## $ 运算符重载

- 使对象的运算表现得和编译器内置类型一样
- 本质上是对象对成员方法的调用，运算符左边是调用操作符的对象，右边是运算符重载函数的参数
- 编译器做对象运算的时候，会调用对象的运算符重载函数（优先调用成员方法）；如果没有成员方法，就在全局作用域找合适的运算符重载函数。
- ++、--：是单目运算符，`operator++()`前置++， `operator++(int)`带有整型参数的是后置++

### $$ 容器的迭代器

- 迭代器的功能：提供一种统一的方式，来透明的遍历容器.一般都要实现`运算符重载: !=  前置++  提领*`
- iterator -> 遍历所有容器 -> 方式都是一样的

  ```cpp
  auto it = container.begin();
  for(; it != container.end(); ++it) cout << *it << endl;
  ```

- 泛型算法 -> 给所有容器都可以使用的， 参数接收的都是容器的迭代器

### $$ 迭代器的失效问题

- C++primer 9.3.6
- 迭代器为什么会失效
  - 当容器调用`erase` 方法后，当前位置到容器末尾位置的所有迭代器全部失效
  - 当容器调用`insert`方法后，当前位置到容器末尾位置的所有迭代器全部失效
  >迭代器依然有效:[首元素， 插入点/删除点]
  >迭代器全部失效:[插入点/删除点， 末尾元素]
  - 对于`insert`来说， 如果引起了容器扩容，那么原容器的所有迭代器全部失效
- 问题怎么解决
  - 对插入/删除点的迭代器进行更新操作

## $ 继承与多态

### $$ 继承的本质和原理

- 继承的本质：代码的复用
- 类和类之间的关系：
  - 组合：a part of... ...一部分的关系
  - 继承：a kind of... ...一种的关系

|继承方式|基类的访问限定|派生类的访问限定|外部的访问限定|
|-|-|-|-|
|public|public|public|Y|
|public|protected|protected|N|
|public|private|不可见的|N|
|protected|public|protected|N|
|protected|protected|protected|N|
|protected|private|不可见的|N|
|private|public|private|N|
|private|protected|private|N|
|private|private|不可见的|N|

总结：

- 基类的成员的访问限定，在派生类里面是不可能超过继承方式的
- 外部只能访问对象public的成员，protected和private的成员无法直接访问
- 在继承结构中，派生类从基类可以继承过来private的成员，但是派生类却无法直接访问
- protected和private的区别？在基类中定义的成员，想被派生类访问，但是不想被外部访问，
那么在基类中，把相关成员定义成protected保护的；如果派生类和外部都不打算访问，那么
在基类中，就把相关成员定义成private私有的。
- 只有自己或者友元能访问私有的成员

- 默认的继承方式是什么？
要看派生类是用class定义的，还是struct定义的？
class定义派生类，默认继承方式就是private私有的
struct定义派生类，默认继承方式就是public私有的

### $$ 派生类的构造过程

- 把继承结构，也说成从上（基类）到下（派生类）的结构
  - 基类对象 <- 派生类对象          类型从下到上的转换  Y
  - 派生类对象 <- 基类对象          类型从上到下的转换  N
  - 基类指针（引用）<- 派生类对象    类型从下到上的转换  Y
  - 派生类指针（引用）<- 基类对象    类型从上到下的转换  N
- 总结：在继承结构中进行上下的类型转换，默认只支持从下到上的类型的转换
- 派生类从继承可以继承来所有的成员(变量和方法)，除过构造函数和析构函数
  - 派生类怎么初始化从基类继承来的成员变量呢？ 通过调用基类相应的构造函数来初始化
  - 派生类从基类继承来的成员，的初始化和清理由谁负责呢？ 是由基类的构造和析构函数来负责
- 派生类的构造函数和析构函数，负责初始化和清理派生类部分

**派生类对象构造和析构的过程是：**  

- 构造由内而外： 派生类调用基类的构造函数，初始化从基类继承来的成员；调用派生类自己的构造函数，初始化派生类自己特有的成员
- 析构由外而内： 调用派生类的析构函数，释放派生类成员可能占用的外部资源（堆内存，文件）；调用基类的析构函数，释放派生类内存中，从基类继承来的成员可能占用的外部资源（堆内存，文件）

### $$ 重载、 覆盖、 隐藏

- 重载关系：一组函数要重载，必须处在同一个作用域当中；而且函数名字相同，参数列表不同
- 隐藏（作用域的隐藏）的关系：在继承结构当中，派生类的同名成员，把基类的同名成员给隐藏调用了
- 覆盖关系：基类和派生类的方法，返回值、函数名以及参数列表都相同，而且基类的方法是虚函数，
那么派生类的方法就自动处理成虚函数，它们之间成为覆盖关系

**一个类添加了虚函数，对这个类有什么影响？**  

- 总结一：一个类里面定义了虚函数，那么编译阶段，编译器给这个类类型产生一个唯一的vftable虚函数
表，虚函数表中主要存储的内容就是RTTI指针和虚函数的地址。当程序运行时，每一张虚函数表
都会加载到内存的.rodata区。
- 总结二：一个类里面定义了虚函数，那么这个类定义的对象，其运行时，内存中开始部分，多存储一个
vfptr虚函数指针，指向相应类型的虚函数表vftable。一个类型定义的n个对象，它们的额vfptr
指向的都是同一张虚函数表
- 总结三：一个类里面虚函数的个数，不影响对象内存大小（vfptr），影响的是虚函数表的大小
- 总结四：如果派生类中的方法，和基类继承来的某个方法，返回值、函数名、参数列表都相同，而且基类的方法是virtual虚函数，那么派生类的这个方法，自动处理成虚函数。**重写《=》覆盖**

**哪些函数不能实现成虚函数？**

虚函数依赖：
> 1.虚函数能产生地址，存储在vftable当中
> 2.对象必须存在(vfptr -> vftable -> 虚函数地址)

构造函数：
> 1.virtual+构造函数 NO
> 2.构造函数中（调用的任何函数，都是静态绑定的）调用虚函数，也不会发生静态绑定派生类对象构造过程:先调用的是基类的构造函数,才调用派生类的构造函数

static静态成员方法 NO！ virtual + static

**什么时候把基类的析构函数必须实现成虚函数？**  
虚析构函数: 析构函数调用的时候，对象是存在的！
>基类的指针（引用）指向堆上new出来的派生类对象的时候， delete pb(基类的指针)，它调用析构函数的时候，必须发生动态绑定，否则会导致派生类的析构函数无法调用
>基类的析构函数是virtual虚函数，那么派生类的析构函数自动成为虚函数

**是不是虚函数的调用一定就是动态绑定？**  
在类的构造函数当中，调用虚函数，也是静态绑定（构造函数中调用其它函数（虚），不会发生动态绑定的）
如果不是**通过指针或者引用变量来调用虚函数**，那就是静态绑定

**静态绑定与动态绑定：**  

- 静态类型：对象在声明时采用的类型，在编译期既已确定，无法更改；
- 动态类型：通常是指一个指针或引用**目前所指**对象的类型，是在运行期决定的，可以更改；
- 静态绑定：绑定的是静态类型，所对应的函数或属性依赖于对象的静态类型，发生在编译期；
- 动态绑定：绑定的是动态类型，所对应的函数或属性依赖于对象的动态类型，发生在运行期；
- 以上，非虚函数一般都是静态绑定，只有虚函数是动态绑定（如此才可实现多态性）
- 重要：绝对不要重新定义继承而来的非虚函数（隐藏）；虚函数的调用是动态绑定的，但是**函数的缺省参数值却是静态绑定的**（函数调用，参数压栈是在编译时期就确定好的），所以绝对不要重新定义一个继承而来的虚函数的缺省参数值。effective C++ 36、37
- 重要：成员方法的访问权限（public/protected/private）是在编译阶段就确定好的，是静态绑定的，所以编译阶段只会检查方法对应静态绑定的调用是否正确。对于虚函数（通过指针或引用调用），只有运行期间的动态绑定才能确定具体调用了谁（当然，前提是必须能通过编译器的静态绑定而不报错）！

### $$ 多态  vfptr和vftable

**如何解释多态？**  

- 静态（编译时期）的多态： 函数重载、模板（函数模板和类模板）
- 动态（运行时期）的多态： 基类和派生类。 在继承结构中，基类指针（引用）指向派生类对象，通过该指针（引用）调用同名覆盖方法（虚函数），基类指针指向哪个派生类对象，就会调用哪个派生类对象的同名覆盖方法，称为多态。

**继承的好处？**  

- 可以做代码的复用  
- 在基类中给所有派生类提供统一的虚函数接口，让派生类进行重写，然后就可以使用多态了

### $$ 抽象类

**抽象类和普通类有什么区别？**  

- 拥有纯虚函数的类，叫做抽象类！
- 基类：泛指     类：抽象一个实体的类型
- 一般把什么类设计成抽象类？
  - 基类。 可以让实体类通过继承基类，实现属性的复用；给所有的派生类保留统一的覆盖/重写接口
- 抽象类不能实例化对象，但是可以定义指针和引用变量

### $$ 多重继承

- 多重继承 ：代码的复用   一个派生类有多个基类
- 好处：可以做更多代码的复用
- 菱形继承的问题：基类A派生了派生类B和C,而类D又是从B和C多继承而来，A是D的间接基类，ABCD继承关系像一个菱形。
  - **会导致派生类有多份间接基类的数据**
  - 解决办法：对间接基类采用虚继承（变为间接虚基类），这样，从虚基类继承来的成员都会放在派生类内存的最后面（只有一份，不会重复），派生类里会生成相应的vbptr和vbtable。

### $$ 虚基类  vbptr和vbtable

- 抽象类(有纯虚函数的类) | 虚基类: 被虚继承的类称作虚基类, 有vbptr和vbtable
- virtual
  1. 修饰成员方法是虚函数
  2. 可以修饰继承方式，是虚继承。被虚继承的类，称作虚基类
- 基类指针指向派生类对象，永远指向的是派生类基类部分数据的起始地址

### $$ RTTI

### $$ C++四种类型强转

|类型强转|说明|
|-|-|
|const_cast | 去掉（指针或者引用）常量属性的一个类型转换，注意只能是**指针或引用**|
|static_cast |  提供编译器认为安全的类型转换（没有任何联系的类型之间的转换就被否定了）|
|reinterpret_cast | 类似于C风格的强制类型转换 |
|dynamic_cast | 主要用在继承结构中，可以支持RTTI类型识别的上下转换|

- 基类类型 《=》 派生类类型  能不能用static_cast?  当然可以！

## $ STL

### $$ 标准容器

#### $$$ 顺序容器

1. vector:向量容器
    - 底层数据结构：动态开辟的数组，每次以原来空间大小的2倍进行扩容的
    - `vector<int> vec;`
    - 增加：
      - vec.push_back(20); 末尾添加元素 O(1)   导致容器扩容
      - vec.insert(it, 20); it迭代器指向的位置添加一个元素20  O(n)   导致容器扩容
    - 删除：
      - vec.pop_back(); 末尾删除元素 O(1)
      - vec.erase(it); 删除it迭代器指向的元素 O(n)
    - 查询：
      - operator[] 下标的随机访问vec[5]  O(1)
      - iterator迭代器进行遍历
      - find，for_each
      - foreach => 通过iterator来实现的
    - 注意：对容器进行连续插入或者删除操作(insert/erase)，一定要更新迭代器，否则第一次insert或者erase完成，迭代器就失效了
    - 常用方法介绍:
      - size()
      - empty()
      - reserve(20)：vector预留空间的   只给容器底层开辟指定大小的内存空间，并不会添加新的元素
      - resize(20)：容器扩容用的  不仅给容器底层开辟指定大小的内存空间，还会添加新的元素
      - swap ： 两个容器进行元素交换

2. deque：双端队列容器
    - 底层数据结构：动态开辟的二维数组，一维数组从2开始，以2倍的方式进行扩容，每次扩容后，原来第二维的数组，从新的第一维数组的下标oldsize/2开始存放，上下都预留相同的空行，方便支持deque的首尾元素添加
    - `deque<int> deq;`
    - 增加：
      - deq.push_back(20); 从末尾添加元素 O(1)  
      - deq.push_front(20); 从首部添加元素 O(1)   // vec.insert(vec.begin(), 20) O(n)
      - deq.insert(it, 20); it指向的位置添加元素 O(n)
    - 删除：
      - deq.pop_back(); 从末尾删除元素 O(1)  
      - deq.pop_front(); 从首部删除元素 O(1)  
      - deq.erase(it);  从it指向的位置删除元素 O(n)
    - 查询搜索：
      - iterator(连续的insert和erase一定要考虑迭代器失效的问题)
3. list：链表容器
    - 底层数据结构：双向的循环链表  pre data next
    - `list<int> mylist;`
    - 增加：
      - mylist.push_back(20); 从末尾添加元素 O(1)
      - mylist.push_front(20); 从首部添加元素 O(1)   // vec.insert(vec.begin(), 20) O(n)
      - mylist.insert(it, 20); it指向的位置添加元素 O(1) // 链表中进行insert的时候，先要进行一个query查询操作,对于链表来说，查询操作效率就比较慢了
    - 删除：
      - mylist.pop_back(); 从末尾删除元素 O(1)
      - mylist.pop_front(); 从首部删除元素 O(1)
      - mylist.erase(it);  从it指向的位置删除元素 O(1)
    - 查询搜索：
      - iterator(连续的insert和erase一定要考虑迭代器失效的问题)
4. deque和list，比vector容器多出来的增加删除函数接口：
    - push_front和pop_front
5. 容器的纵向考察：容器掌握的深度; 容器的横向考察：各个相似容器之间的对比
    - vector特点：动态数组，内存是连续的，2倍的方式进行扩容， `vector<int> vec;` 0-1-2-4-8... reserve(20)/resize
    - deque特点：动态开辟的二维数组空间，**第二维是固定长度的数组空间**，扩容的时候（第一维的数组进行2倍扩容）
    - 面经问题：deque底层内存是否是连续的？   并不是  每一个第二维是连续的.
    - vector和deque之间的区别？
      1. 底层数据结构：
      2. 前中后插入删除元素的时间复杂度： 中间和末尾 O(1)  前 deque O(1) vector O(n)
      3. 对于内存的使用效率： vector 需要的内存空间必须是连续的    deque 可以分块进行数据存储，不需要内存空间必须是一片连续的
      4. 在中间进行insert或者erase，vector和deque它们的效率谁能好一点(vector)？谁能差一点(deque)？  O(n)
    - vector和list之间的区别？
      - 数组:增加删除O(n) 查询O(n) 随机访问O(1)   链表:(考虑搜索的时间)增加删除O(1)  查询O(n)
      - 底层数据结构：数组   双向循环链表

#### $$$ 容器适配器

- 标准容器 - 容器适配器 => 设计模式，就叫做适配器模式
- 怎么理解这个适配器？
  1. 适配器底层没有自己的数据结构，它是另外一个容器的封装，它的方法全部由底层依赖的容器进行实现的
  2. 没有实现自己的迭代器

1. stack
    - 底层容器：deque
    - 方法：push入栈  pop出栈  top查看栈顶元素  empty判断栈空  size返回元素个数
2. queue
    - 底层容器：deque
    - 方法：push入队  pop出队  front查看队头元素 back查看队尾元素  empty判断队空   size返回元素个数
3. priority_queue
    - 底层容器：vector
    - 方法：push入队  pop出队  top查看队顶元素  empty判断队空  size返回元素个数  默认：大根堆
4. queue => deque  为什么不依赖vector呢？？？stack => deque  为什么不依赖vector呢？？？priority_queue => vector 为什么依赖vector？？？
    - vector的初始内存使用效率太低了！没有deque好  `queue<int>` `stack<int>`  vector 0-1-2-4-8 deque 4096/sizeof(int) = 1024
    - 对于queue来说，需要支持尾部插入，头部删除，O(1)  如果queue依赖vector，其出队效率很低
    - vector需要大片的连续内存，而deque只需要分段的内存，当存储大量数据时，显然deque对于内存的利用率更好一些
    - 对于优先队列：底层默认把数据组成一个大根堆结构  在一个**内存连续**的数组上构建一个大根堆或者小根堆的，所以只能用vector

#### $$$ 关联容器

1. 无序关联容器：底层  => 链式哈希表  增删查O(1)
    - unordered_set 单重集合
    - unordered_multiset 多重集合
    - unordered_map 单重映射表
    - unordered_multimap 多重映射表
2. 有序关联容器：底层  => 红黑树 增删查O(log2n) 2是底数(树的层数，树的高度)
    - set： 集合 key
    - multiset： 多重集合
    - map： 映射表 [key,value]
    - multimap：多重映射表

### $$ 近容器

- 数组
- string
- bitset(位容器)

### $$ 迭代器

- const_iterator: 常量的正向迭代器  只能读，而不能写了
- iterator: 普通的正向迭代器
- reverse_iterator: 普通的反向迭代器
- const_reverse_iterator: 常量的反向迭代器

- 与容器中`begin（）`，`end()`相对的：
  - rbegin()：返回的是最后一个元素的反向迭代器表示
  - rend：返回的是首元素前驱位置的迭代器的表示

### $$ 函数对象（类似C的函数指针）

- 就是仿函数：本质是一个类，用起来像个函数一样
- 通过函数对象调用operator()方法，可以省略函数的调用开销，比通过函数指针调用函数（不能够inline内联调用）效率高
- 因为函数对象是用类生成的，所以可以添加相关的成员变量，用来记录函数对象使用时更多的信息

### $$ 泛型算法

- 泛型算法 = template + 迭代器 + 函数对象
- 特点一：泛型算法的参数接收的都是迭代器
- 特点二：泛型算法的参数还可以接收函数对象（C函数指针）
  - sort,find,find_if,binary_search,for_each
- 绑定器 + 二元函数对象 =》 一元函数对象
  - bind1st：把二元函数对象的operator()(a, b)的第一个形参绑定起来
  - bind2nd：把二元函数对象的operator()(a, b)的第二个形参绑定起来

```cpp
#include <algorithm> // 包含了C++ STL里面的泛型算法
#include <functional> // 包含了函数对象和绑定器
```
