> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_43949535/article/details/97764912#sr-toc-0)

### 文章目录

*   [第一节：单例模式](#_5)
*   [第二节：线程安全的懒汉单例模式](#_150)
*   [第三节：简单工厂和工厂方法](#_368)
*   [第四节：抽象工厂](#_647)
*   [第五节：代理模式](#_787)
*   [第六节：装饰器模式](#_1097)
*   [第七节：适配器模式](#_1247)
*   [第八节：观察者模式](#_1408)

设计模式：就是在解决某一类问题场景时，有既定的优秀的代码框架可以直接使用，有以下优点可取：  
（1）代码更易于维护，代码的可读性、复用性、可移植性、健壮性更好  
（2）当软件原有需求有变更或者增加新的需求时，合理的设计模式的应用，能够做到软件设计要求的 “开 - 闭原则”，即对修改关闭，对扩展开放，使软件原有功能修改，新功能扩充非常灵活  
（3）合理的设计模式的选择，会使软件设计更加模块化，积极的做到软件设计遵循的 “高内聚，低耦合” 这一根本原则。

第一节：单例模式
========

单例模式顾名思义，保证一个类仅可以有一个实例化对象，并且提供一个可以访问它的全局接口。这是一个创建性模式（主要是指对象的创建方式）。**单例模式和多线程结合到是很紧密的。包括两种单例模式，以及线程安全的问题。非常重要**  
![](https://img-blog.csdnimg.cn/20190730125254523.png)  
![](https://img-blog.csdnimg.cn/20190730125355702.png)  
按照之前的代码，定义一个类 A。如上图所示：那么这就相当于 创建了 3 个不同的实例对象。但是在有些情况下：**一个类不管创建多少次对象，永远只能得到该类型一个对象的实例。** 在这样的情况下：有很多例子 如下：  
![](https://img-blog.csdnimg.cn/20190730130032711.png)  
第一种：日志模块  
假如 这里有一个类，提供了很多方法来封装了一个日志模块。一个软件本身可能有很多的功能模块，那么这么多的模块都要通过日志模块进行写入一些软件运行的日志信息到磁盘内。应该是把这些所有的日志信息都给到一个日志模块的对象上，日志怎么书写、往哪块写、格式是什么、分不分文件写都是由一个日志模块对象具体实现（根本不需要用户去自定义日志对象去书写日志：不方便 没必要 很麻烦）。而且软件在运行的过程中，需要写日志的地方特别多，所以说不能每次写日志的时候都要去创建一个日志对象（这样会导致日志对象特别特别多，占用大量内存且非常麻烦）。且书写日志也不是非常着急，只需要在最后能把日志写入即可。所以说 软件把尽可能多的 CPU 资源、内存资源、I/O 资源都尽可能快地给到其他的功能模块上（如 处理客户的请求等）。  
第二种：数据库模块  
作为一个个的数据库客户端，是要通过数据库模块与数据库服务器进行交互通信的。那么这个数据库 client 就可以设计成一个单例：应用软件的其他功能模块如果要与数据库服务器进行交互通信，可以调用一个数据库模块对象的某一个方法即可，不需要去创建那么多的对象。毕竟数据库的请求那么多，不可能做到 每一次处理请求都生成一个数据库对象，这样会导致数据库模块对象特别特别多和数据库的连接也特别的多，占用大量内存且非常麻烦。

如上等等都可以实现为单例模式来设计。  
**实现单例模式必须注意一下几点：**  
（1）单例类只能有一个实例化对象。  
（2）单例类必须自己提供一个实例化对象。  
（3）单例类必须提供一个可以访问唯一实例化对象的接口。  
（4）单例模式分为懒汉和饿汉两种实现方式。  
那么我们给如何实现一个类 设计成单例模式呢？具体该怎么去做呢?  
要去控制这个类 生成对象的个数，就得去控制其构造函数  
用户若可以任意访问构造函数，那么他就会去创建无数个对象  
所以就得去限制构造函数的访问方式  
**如果要用单例去设计一个类：步骤如下**  
（1）那么就需要把这个类的构造函数 私有化  
(那么这个用户在外面就没有办法去通过 new 或者 在栈上去任意创建对象)  
（2）定义一个惟一的类的实例对象：用户无法创建，所以得去给用户提供一个实例化对象使用  
（3）定义一个接口，用户可以通过这个接口来获取这个类惟一的实例对象。又因为这个接口是给用户获取这个类惟一的实例对象的，所以这个接口方法是得定义成静态的。（若是普通的成员方法，调用的时候还得去依赖一个对象。可是这个接口就是来获取这个类惟一的实例对象的，你哪来的对象使用呢）  
代码如下：

```
#include <iostream>
using namespace std;

/*
要去控制这个类 生成对象的个数，就得去控制其构造函数
用户若可以任意访问构造函数，那么他就会去创建无数个对象
所以就得去限制构造函数的访问方式
…………………………………………………………………………
如果要用单例去设计一个类：
（1）那么就需要把这个类的构造函数 私有化
(那么这个用户在外面就没有办法去通过 new 或者 在栈上去任意创建对象)
（2）定义一个惟一的类的实例对象：用户无法创建，所以得去给用户提供一个实例化对象使用
（3）定义一个接口，用户可以通过这个接口来获取这个类惟一的实例对象。又因为这个接口是
给用户获取这个类惟一的实例对象的，所以这个接口方法是得定义成静态的。（若是普通的成员
方法，调用的时候还得去依赖一个对象。可是这个接口就是来获取这个类惟一的实例对象的，
你哪来的对象使用呢）
*/
class Singleton_Pattern
{
public:
	//获取这个类惟一的实例对象的接口方法
	static Singleton_Pattern* getInstance()//第3步：类的静态方法
	{
		return &_instance;
	}
private:
	//静态的成员变量
	static Singleton_Pattern _instance;//这是第二步
	Singleton_Pattern(){}//这是第一步
};
//静态的成员变量需要在类外进行初始化
Singleton_Pattern Singleton_Pattern:: _instance;
int main()
{
	//于是在外面就可以这样去使用了，因为不能通过构造函数去创建对象的
	Singleton_Pattern* p1 = Singleton_Pattern::getInstance();
	Singleton_Pattern* p2 = Singleton_Pattern::getInstance();
	Singleton_Pattern* p3 = Singleton_Pattern::getInstance();
	cout << "p1:" << p1 << " " << " p2:" << p2 
		<< " p3:" << p3 << endl;//地址一样
	return 0;
}
```

运行截图如下：  
![](https://img-blog.csdnimg.cn/20190730141305689.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
但是上面的代码还是有点漏洞：  
![](https://img-blog.csdnimg.cn/20190730141424285.png)  
即：拷贝构造函数 和 赋值重载函数没有给 delete 掉，所以他们还是可以通过这两个函数去生成新对象的。所以在限制对象的生成方式上，不仅仅要去限制其构造函数，连拷贝构造函数 和 赋值重载函数也是要进行处理的。即在类内处理为：  
![](https://img-blog.csdnimg.cn/20190730141729110.png)  
在此时再行调用的时候：  
![](https://img-blog.csdnimg.cn/2019073014250089.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
![](https://img-blog.csdnimg.cn/20190730192703978.png)  
所以上面的代码是一个**饿汉式**单例模式的实现。因为上面定义的一个惟一的类的实例对象，在这个类中定义的类的成员变量是一个静态的对象（静态的成员变量）。静态的成员变量是在进程内存的数据段上，数据段内存的创建和初始化在 main 函数之前的。即使不调用接口方法，这个实例对象已经实例化过了，相当于是落在了类的作用域里面的**一个全局变量**。**饿汉式**单例模式一定是线程安全的。因为多线程的执行是要先执行函数的，线程函数启动才标志着这个线程开始执行的。静态的成员变量在数据段，数据段在整个程序启动 main 函数之前就创建和初始化好的了。但是**饿汉式**单例模式的缺点：对象的实例化也是要调用构造函数的，但是在实际的项目中：构造函数是要做很多事情的。例如：加载很多配置信息，读磁盘文件，访问数据库等。可是这些事情是应该放在第一次需要访问 这个实例对象的时候，才去创建这个类的实例对象的。**也有可能在整个程序运行过程中 根本就没有要去获取这个类的唯一的实例对象的。** 那么费了很大功夫 构造这个对象，真的是一点用处也没有。而且在软件启动过程中：数据段要进行内存创建和初始化 构造函数的调用中 构造函数再去做很多事情，那么这样的太多的**饿汉式**单例对象势必会影响 软件启动的时间和效率。  
**++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++**  
于是因此很受欢迎的就是**懒汉单例模式**了  
**懒汉单例模式**  
懒汉：故名思义，不到万不得已就不会去实例化类，也就是说在第一次用到类实例的时候才会去实例化一个对象。在访问量较小，甚至可能不会去访问的情况下，采用懒汉实现，这是以时间换空间。把新的唯一的实例对象的创建延迟到第一次获取该唯一的实例对象的时候，这个对象才去构造 初始化。**懒汉单例模式** 就需要考虑这个线程安全的问题了：因为用的时候才生成对象，但对象的构造又是一个非常复杂的过程，不是一个原子的操作。

在程序启动过程中，在数据段上只有一个 该类的指针（没有对象的创建）。在第一次 getInstance 方法调用的时候，想访问该类的唯一的实例对象的时候。这个指针为空，则创建这个新的唯一的实例对象的。不为空则返回第一次创建过的那个新的唯一的实例对象的指针。

```
#include <iostream>
using namespace std;
/*
要去控制这个类 生成对象的个数，就得去控制其构造函数
用户若可以任意访问构造函数，那么他就会去创建无数个对象
所以就得去限制构造函数的访问方式
…………………………………………………………………………
如果要用单例去设计一个类：
（1）那么就需要把这个类的构造函数 私有化
(那么这个用户在外面就没有办法去通过 new 或者 在栈上去任意创建对象)
（2）定义一个惟一的类的实例对象：用户无法创建，所以得去给用户提供一个实例化对象使用
（3）定义一个接口，用户可以通过这个接口来获取这个类惟一的实例对象。又因为这个接口是
给用户获取这个类惟一的实例对象的，所以这个接口方法是得定义成静态的。（若是普通的成员
方法，调用的时候还得去依赖一个对象。可是这个接口就是来获取这个类惟一的实例对象的，
你哪来的对象使用呢）
*/
class Singleton_Pattern
{
public:
	//获取这个类惟一的实例对象的接口方法
	static Singleton_Pattern* getInstance()//第3步：类的静态方法
	{
		//那么这就是为懒汉模式
		if (_instance == nullptr)
		{
			//如为空 则定义一个新的唯一的实例对象
			_instance = new Singleton_Pattern();
		}
		return _instance;//不空 返回该类的唯一的实例对象
	}
	Singleton_Pattern(const Singleton_Pattern& ) = delete;
	Singleton_Pattern& operator=(const Singleton_Pattern&) = delete;
private:
	//静态的成员变量
	static Singleton_Pattern *_instance;//这是第二步 定义成指针
	Singleton_Pattern(){}//这是第一步
};
//静态的成员变量需要在类外进行初始化
//指针初始化为空
Singleton_Pattern* Singleton_Pattern:: _instance = nullptr;
int main()
{
	//于是在外面就可以这样去使用了，因为不能通过构造函数去创建对象的
	Singleton_Pattern* p1 = Singleton_Pattern::getInstance();
	Singleton_Pattern* p2 = Singleton_Pattern::getInstance();
	Singleton_Pattern* p3 = Singleton_Pattern::getInstance();

	return 0;
}
```

运行截图如下：  
![](https://img-blog.csdnimg.cn/20190730215546985.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
遗留问题：  
（1）此时的这个 **懒汉单例模式代码** 是不是线程安全的呢？  
（2）这个类的 getInstance 方法是不是 可重入函数呢？

第二节：线程安全的懒汉单例模式
===============

可重入：这个函数还没有执行完的时候，是否还可以继续再调用一次。在单线程的场景下：不存在这个函数还没有执行完的时候，又被再调用了一次。

可重入函数：一个函数可直接在多线程环境下使用，而且也不会发生 竞态条件，那么这个函数就可以被称作是可重入的、可被多个线程不断重复调用的函数。如果会发生 竞态条件，则不能被称为可重入函数（**就需要去考虑线程安全的问题了**）。

很明显上面的代码 如下：

```
//获取这个类惟一的实例对象的接口方法
	static Singleton_Pattern* getInstance()//第3步：类的静态方法
	{
		//那么这就是为懒汉模式
		if (_instance == nullptr)
		{
			//如为空 则定义一个新的唯一的实例对象
			_instance = new Singleton_Pattern();
		}
		return _instance;//不空 返回该类的唯一的实例对象
	}
```

其中的这句代码：`_instance = new Singleton_Pattern();` 首先要去开辟内存，构造对象，给指针赋值。可是假如线程 1 调用 getInstance 方法进来了，指针首先为空。**假如线程 1 只是做到了开辟内存，构造对象，但是尚未给指针赋值。** 其时间片到，此时的指针仍然为空，下一个线程也可以进来了。那么这个单例模式 从逻辑上就不可以在多线程环境下运行了。`_instance = new Singleton_Pattern();` 这里必须是一个单例的对象，不可能进来两次。**所以线程安全的问题就很严重了。**  
![](https://img-blog.csdnimg.cn/20190730221910242.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
而且还存在一个问题：  
编译器会对高级源代码进行编译的时候，把其产生的指令进行顺序上的倒换的。为了加速代码的执行，会把步骤 2 和 步骤 3 进行顺序上的倒换的（这两个操作是没有关系的，在翻译成汇编指令的时候 顺序上发生了倒换）。还存在的竞态条件就会是如下：  
![](https://img-blog.csdnimg.cn/20190730222249663.png)  
此时 可是假如线程 1 调用 getInstance 方法进来了，指针首先为空。**假如线程 1 只是做到了开辟内存，给指针赋值，但是尚未构造对象。** 其时间片到，此时的指针不为空，下一个线程也就进不来了。然后这个线程就得到了一个没有构造对象的内存的指针，然后这个线程在访问这个对象的时候就出错了。**那么这个函数就又不是一个可重入的函数了。**

为了解决上面的两种情况的发生，就要去做 线程间的互斥操作了。一定要去保证 if 这段代码（临界区代码段）的原子操作。  
局部修改代码如下：

```
//获取这个类惟一的实例对象的接口方法
	static Singleton_Pattern* getInstance()//第3步：类的静态方法
	{
		//那么这就是为懒汉模式
		lock_guard<std::mutex>l_guard(mymutex);//加锁保证多线程间安全
		if (_instance == nullptr)
		{
			//如为空 则定义一个新的唯一的实例对象
			_instance = new Singleton_Pattern();
		}
		return _instance;//不空 返回该类的唯一的实例对象
	}
```

这样做很好的解决了多线程的线程安全，但是这么重的锁导致单线程 每次进来有必要去不停的加锁解锁码？效率上不太合适了！！！

继续修改代码：如下：

```
//获取这个类惟一的实例对象的接口方法
	static Singleton_Pattern* getInstance()//第3步：类的静态方法
	{
		//那么这就是为懒汉模式
		if (_instance == nullptr)
		{
			lock_guard<std::mutex>l_guard(mymutex);//加锁保证多线程间安全
			//如为空 则定义一个新的唯一的实例对象
			_instance = new Singleton_Pattern();
		}
		return _instance;//不空 返回该类的唯一的实例对象
	}
```

这样做的话 单线程就好多了。效率上的问题解决了。单线程在第一次生成对象的时候进行加锁解锁，后面在使用 getInstance 方法获取类的实力对象的时候，进不了 if 语句，就不需要进行加锁解锁了。但是多线程可能就不太好了：假如线程 1 调用 getInstance 方法进来了，指针首先为空。**假如线程 1 只是做到了开辟内存，构造对象，但尚未给指针赋值。** 此时的指针为空，下一个线程也就进来了。但是然后这个线程得不到互斥锁，线程阻塞了。当线程 1 开辟内存，构造对象，给指针赋值。都做完的时候，出作用域进行互斥锁的释放。线程 2 获取互斥锁，也要去执行`_instance = new Singleton_Pattern();` 所以这当然是出错的。

**所以这个时候（把锁移入里面，多个线程就都可能进入里面了），就要去加上双重判断了。** 修改代码如下：

```
//获取这个类惟一的实例对象的接口方法
	static Singleton_Pattern* getInstance()//第3步：类的静态方法
	{
		//那么这就是为懒汉模式
		if (_instance == nullptr)
		{
			lock_guard<std::mutex>l_guard(mymutex);//加锁保证单线程方便
			if (_instance == nullptr)//双重判断 解决多线程安全
			{
				//如为空 则定义一个新的唯一的实例对象
				//且下面代码只能去执行1次
				_instance = new Singleton_Pattern();
			}
		}
		return _instance;//不空 返回该类的唯一的实例对象
	}
```

这样做就是一个很完美的线程安全的懒汉单例模式。因为这个指针_instance 在数据段上，是属于同一个进程的多个线程的共享的内存。CPU 在执行线程指令的时候，为了加快指令的执行，让线程把他们共享的内存的值都拷贝一份，放在自己的线程缓存里面。（放到 CPU 的缓存里面）所以还是要在指针_instance 前面加上一个 volatile 关键字。volatile 是给指针加的，不是给指针的指向加的。  
其优点就在于：第一个线程给指针_instance 赋值的时候，其他的线程都能发现指针_instance 的值发生了改变（线程已经不能对这个共享变量：指针的值进行缓存了，大家看的值都是_instance 内存里面的值）。  
**此时的代码才算是最完美：**

```
#include <iostream>
#include <mutex>

using namespace std;

/*
要去控制这个类 生成对象的个数，就得去控制其构造函数
用户若可以任意访问构造函数，那么他就会去创建无数个对象
所以就得去限制构造函数的访问方式
…………………………………………………………………………
如果要用单例去设计一个类：
（1）那么就需要把这个类的构造函数 私有化
(那么这个用户在外面就没有办法去通过 new 或者 在栈上去任意创建对象)
（2）定义一个惟一的类的实例对象：用户无法创建，所以得去给用户提供一个实例化对象使用
（3）定义一个接口，用户可以通过这个接口来获取这个类惟一的实例对象。又因为这个接口是
给用户获取这个类惟一的实例对象的，所以这个接口方法是得定义成静态的。（若是普通的成员
方法，调用的时候还得去依赖一个对象。可是这个接口就是来获取这个类惟一的实例对象的，
你哪来的对象使用呢）
*/
std::mutex mymutex;//定义全局的互斥锁
class Singleton_Pattern
{
public:
	//获取这个类惟一的实例对象的接口方法
	static Singleton_Pattern* getInstance()//第3步：类的静态方法
	{
		//那么这就是为懒汉模式
		if (_instance == nullptr)
		{
			lock_guard<std::mutex>l_guard(mymutex);//加锁保证单线程方便
			if (_instance == nullptr)//双重判断 解决多线程安全
			{
				//如为空 则定义一个新的唯一的实例对象
				//且下面代码只能去执行1次
				_instance = new Singleton_Pattern();
			}
		}
		return _instance;//不空 返回该类的唯一的实例对象
	}
	Singleton_Pattern(const Singleton_Pattern& ) = delete;
	Singleton_Pattern& operator=(const Singleton_Pattern&) = delete;
private:
	//静态的成员变量
	static Singleton_Pattern * volatile _instance;//这是第二步 定义成指针
	Singleton_Pattern(){}//这是第一步
};//静态的成员变量需要在类外进行初始化
Singleton_Pattern * volatile Singleton_Pattern:: _instance = nullptr;
//指针初始化为空
int main()
{
	//于是在外面就可以这样去使用了，因为不能通过构造函数去创建对象的
	Singleton_Pattern* p1 = Singleton_Pattern::getInstance();
	Singleton_Pattern* p2 = Singleton_Pattern::getInstance();
	Singleton_Pattern* p3 = Singleton_Pattern::getInstance();
	cout << "p1:" << p1 << " " << " p2:" << p2
		<< " p3:" << p3 << endl;//地址一样
	return 0;
}
```

运行截图如下：  
![](https://img-blog.csdnimg.cn/20190730230917691.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
**++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++**  
下面是 函数的静态的局部对象，程序启动阶段，其内存（数据段）就有了，但是静态的对象 第一次初始化是在第一次运行这个对象的时候才进行的初始化。所以在不调用 getInstance 方法的时候，这个对象是不构造的，就不会去调用构造函数：做复杂的初始化动作。因此这个_instance 也是在第一次调用 getInstance 方法的时候才进行的构造。所以**这个也是懒汉式单例模式**

```
class Singleton_Pattern
{
public:
	//获取这个类惟一的实例对象的接口方法
	static Singleton_Pattern* getInstance()//第3步：类的静态方法
	{
		static Singleton_Pattern  _instance;//这是第二步 
		return &_instance;
	}	
private:
	Singleton_Pattern()//这是第一步
	{
		//很多初始化的代码动作;
	}
	Singleton_Pattern(const Singleton_Pattern&) = delete;
	Singleton_Pattern& operator=(const Singleton_Pattern&) = delete;
};
```

但是这是不是一个线程安全的呢？因为在多线程场景下会不会发生如下问题呢？  
线程 1 调用 getInstance 方法的时候，这个唯一的实例对象要去进行初始化。也就是去调用构造函数，可是调用构造函数：如果里面有很多事情要去初始化。就会发生构造函数执行了一部分的时候，线程 2 也有可能调用 getInstance 方法。线程 2 发现这个对象没有构造完，于是又去初始化 （又去构造），所以这就 违反了 **函数的静态的局部变量只应该去初始化一次。** 在单线程环境下 这是串行调用的，当然不存在多个线程同时调用 getInstance 方法多次。但是在多线程下，就会不会发生多个线程去调用 getInstance 方法呢？  
![](https://img-blog.csdnimg.cn/20190730233505888.png)  
因此在这里 这个**函数的静态局部对象的初始化**，已经是一个线程安全的操作了。（不用去担心有线程安全的问题了）。对于 static 静态局部对象的初始化，编译器会自动对它的初始化进行加锁和解锁控制，使静态局部对象的初始化成为线程安全的操作，不用担心多个线程都会初始化静态局部对象，因此上面的懒汉单例模式是线程安全的单例模式！  
最终的源代码如下：

```
//非常精简的线程安全的懒汉单例模式
class Singleton_Pattern
{
public:
	//获取这个类惟一的实例对象的接口方法
	static Singleton_Pattern* getInstance()//第3步：类的静态方法
	{
		/*
		下面是 函数的静态的局部对象
		程序启动阶段，其内存（数据段）就有了
		但是静态的对象 第一次初始化是在第一次运行
		这个对象的时候才进行的初始化
		所以在不调用getInstance方法的时候，这个对象是不构造的
		就不会去调用构造函数：做复杂的初始化动作。
		因此这个_instance也是在第一次调用getInstance方法的时候
		才进行的构造。所以这个也是懒汉式单例模式
		*/
		static Singleton_Pattern  _instance;//这是第二步 
		return &_instance;
	}
	
private:
	Singleton_Pattern()//这是第一步
	{
		//很多初始化的代码动作;
	}
	Singleton_Pattern(const Singleton_Pattern&) = delete;
	Singleton_Pattern& operator=(const Singleton_Pattern&) = delete;
};
int main()
{
	//于是在外面就可以这样去使用了，因为不能通过构造函数去创建对象的
	Singleton_Pattern* p1 = Singleton_Pattern::getInstance();
	Singleton_Pattern* p2 = Singleton_Pattern::getInstance();
	Singleton_Pattern* p3 = Singleton_Pattern::getInstance();
	cout << "p1:" << p1 << " " << " p2:" << p2
		<< " p3:" << p3 << endl;//地址一样
	return 0;
}
```

![](https://img-blog.csdnimg.cn/20190730235014107.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
在 Linux 环境中，通过 g++ 编译上面的代码，命令如下：  
g++ -o main main.cpp -g  
生成可执行文件 main，用 gdb 进行调试，到 getInstance 函数，并打印该函数的汇编指令，如下：  
请见博客：[https://blog.csdn.net/QIANGWEIYUAN/article/details/88544524](https://blog.csdn.net/QIANGWEIYUAN/article/details/88544524)

第三节：简单工厂和工厂方法
=============

![](https://img-blog.csdnimg.cn/20190731000439290.png)  
简单工厂：它不属于标准的 OOP 设计模式中，而后面两种是包含在标准的 OOP 的 23 种设计模式中的。  
**为什么要工厂模式：主要是封装了对象的创建过程。** 创建性模式本身就是体现了：对象的创建过程的封装和隐藏。没有工厂模式的封装就是：对象的 new 和 new 等。当代码里面出现很多的类，每次创建在对象的时候，都需要通过 new 类名称的方式来生成对象。这样的话，就不得不需要记忆很多类的名称，难记住不说且这样的设计使得代码很难维护，类名一旦发生了改变，那么所有使用类名称的地方都需要去修改。其耦合性太强，不符合软件设计的思想，于是简单工厂就是在这样的需求下应用而生的。

例如：有了简单工厂的方式：当需要对象的时候，就向工厂申请需要一个什么样子的对象，工厂就返回这样的一个对象即可。类比较多的时候，使用工厂模式比较 OK。（就几个类，没必要搞上工厂模式）  
**++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++**  
之前的做法如下：

```
#include <iostream>
#include <string>

class Car
{
public :
	//在构造函数的初始化列表里面给name初始化
	Car(std::string name):_name(name){}

	//提供一个给派生类（具体的汽车）重写的接口
	virtual void show() = 0;//纯虚函数
protected:
	std::string _name;
};

class BMW:public Car
{
public:
	BMW (std::string name):Car(name){}
	//重写从基类基类继承过来的纯虚函数
	void show()
	{
		std::cout << "宝马  " << _name << std::endl;
	}
};
class Audi :public Car
{
public:
	Audi(std::string name) :Car(name) {}
	//重写从基类基类继承过来的纯虚函数
	void show()
	{
		std::cout << "奥迪  " << _name << std::endl;
	}
};

int main()
{
	Car* p1 = new BMW("x1");
	Car* p2 = new Audi("A6");
	p1->show();
	p2->show();
	delete p1;
	delete p2;
	/*
	上面的缺点在于：
	1 需要记住派生类的名字，不然没法new对象
	2 创建对象的时候，直接使用了对象

	其实我们不需要去了解对象创建的具体内容
	只要给我汽车就OK了，劳资不需要去知道怎么造车的
	*/
	return 0;
}
```

![](https://img-blog.csdnimg.cn/20190731205036444.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
所以 解决办法为：把这些对象都给 封装到一个简单的工厂里面。如下：  
通过下面这个简单的工厂把所有对象的创建给封装起来了，下面造汽车劳资不用管，我只是想要一辆车，通过传入不同的参数，得到不同的对象。

```
#include <iostream>
#include <string>

class Car
{
public :
	//在构造函数的初始化列表里面给name初始化
	Car(std::string name):_name(name){}

	//提供一个给派生类（具体的汽车）重写的接口
	virtual void show() = 0;//纯虚函数
protected:
	std::string _name;
};

class Bmw:public Car
{
public:
	Bmw(std::string name):Car(name){}
	//重写从基类基类继承过来的纯虚函数
	void show()
	{
		std::cout << "宝马  " << _name << std::endl;
	}
};
class Audi :public Car
{
public:
	Audi(std::string name) :Car(name) {}
	//重写从基类基类继承过来的纯虚函数
	void show()
	{
		std::cout << "奥迪  " << _name << std::endl;
	}
};

enum CarType
{
	BMW,AUDI
};
//通过下面这个简单的工厂把所有对象的创建给封装起来了
class SimpleFactory
{
public:
	//用户调用这个方法的时候，根据传入的参数的不同，创建相应对象
	Car* createOneCar(CarType type)
	{
		switch (type)
		{
		case BMW:
			return new Bmw("x1");

		case AUDI:
			return new Audi("A6");
		default:
			std::cerr << "传入的参数不正确" << type << std::endl;
		}
		return nullptr;
	}
};
int main()
{
	SimpleFactory* factory = new SimpleFactory();
	Car* p1 = factory->createOneCar(BMW);
	Car* p2 = factory->createOneCar(AUDI);
	/*
	上面造汽车劳资不用管，我只是想要一辆车
	通过传入不同的参数，得到不同的对象
	*/
	p1->show();
	p2->show();
	delete p2;
	delete p1;
	delete factory;
	return 0;
}
```

![](https://img-blog.csdnimg.cn/20190731211629112.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
这里也可以使用上智能指针，帮助我们释放资源。修改上面代码如下：

```
int main()
{
	std::unique_ptr<SimpleFactory> factory(new SimpleFactory());
	std::unique_ptr < Car> p1(factory->createOneCar(BMW));
	std::unique_ptr < Car> p2(factory->createOneCar(AUDI));
	
	/*
	上面造汽车劳资不用管，我只是想要一辆车
	通过传入不同的参数，得到不同的对象
	*/
	p1->show();
	p2->show();
	return 0;
}
```

但是简单工厂也有其缺点：  
同一个工厂既建宝马，也整奥迪。不可能用一个工厂把所有对象的创建都封装起来（这个工厂做的事情太多了，一个工厂负责一个产品的创建）。这里应该是两个工厂比较好一点，再者工厂里面的设计也不符合 **开闭原则**

```
//通过下面这个简单的工厂把所有对象的创建给封装起来了
class SimpleFactory
{
public:
	//用户调用这个方法的时候，根据传入的参数的不同，创建相应对象
	Car* createOneCar(CarType type)
	{
		switch (type)
		{
		case BMW:
			return new Bmw("x1");

		case AUDI:
			return new Audi("A6");
		default:
			std::cerr << "传入的参数不正确" << type << std::endl;
		}
		return nullptr;
	}
};
```

**对修改关闭，对扩展开放。**  
但是上面的代码：对修改 扩展都是开放的。在增加新的对象和删除这个对象的时候，需要重写这个接口（接口都没有封闭）。

鉴于简单工厂这样的缺点：也就有了  
**工厂方法**

```
#include <iostream>
#include <string>
#include <memory>

class Car
{
public :
	//在构造函数的初始化列表里面给name初始化
	Car(std::string name):_name(name){}

	//提供一个给派生类（具体的汽车）重写的接口
	virtual void show() = 0;//纯虚函数
protected:
	std::string _name;
};

class Bmw:public Car
{
public:
	Bmw(std::string name):Car(name){}
	//重写从基类基类继承过来的纯虚函数
	void show()
	{
		std::cout << "宝马  " << _name << std::endl;
	}
};
class Audi :public Car
{
public:
	Audi(std::string name) :Car(name) {}
	//重写从基类基类继承过来的纯虚函数
	void show()
	{
		std::cout << "奥迪  " << _name << std::endl;
	}
};

//工厂方法
class FactoryMethod
{
public:
	//工厂方法
	virtual Car* createOneCar(std::string) = 0;
};
//下面是宝马工厂，负责创建宝马
class BMWFactoryMethod :public FactoryMethod
{
	Car* createOneCar(std::string name)
	{
		return new Bmw(name);
	}
};
//下面是奥迪工厂，负责创建奥迪
class AUDIFactoryMethod :public FactoryMethod
{
	Car* createOneCar(std::string name)
	{
		return new Audi(name);
	}
};

/*
这里相较于简单工厂：使用一个工厂，通过传入不同的标识 来创建不同的对象
相当于给了一个工厂的基类，然后通过实现具体的产品的工厂。用这个具体产品的工厂
来创建具体的产品对象。

*/
int main()
{
	std::unique_ptr<FactoryMethod> bmwfactorymethod
	(new BMWFactoryMethod());
	std::unique_ptr<FactoryMethod> audifactorymethod
	(new AUDIFactoryMethod());
	std::unique_ptr < Car> p1(bmwfactorymethod->createOneCar("x1"));
	std::unique_ptr < Car> p2(audifactorymethod->createOneCar("A5"));

	p1->show();
	p2->show();
	return 0;
}
```

![](https://img-blog.csdnimg.cn/20190731215000680.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
把工厂划分为一个继承结构：封装了一个工厂方法（纯虚函数）。派生类代表具体的工厂，产生具体的产品。（相应的工厂创建相应的产品），达到了一个工厂其相应的产品。

在这种情况下：假如要增加一个 奔驰的工厂，不需要动奥迪和宝马的派生类。只需要加上一个奔驰继承来的派生类即可（在里面重写 createOneCar 方法即可）。**这就是对已有的功能进行封闭。开闭原则：对修改关闭，对扩展开放** 此时要是删除一种产品的工厂，直接删除相应的派生类即可（也不会改动其他的类）。在调用的时候，想要获取一个对象，只需要调用 工厂相应的工厂方法就可以了，不需要去了解派生类叫什么名字、也不需要知道对象是怎么创建的（这些细节由工厂进行维护）。

第四节：抽象工厂
========

简单工厂的缺点如下：  
比如说是 一类有关联关系的产品：手机、手机上的耳机等。根据上面的工厂方法，创建耳机，也得整上一个相应的耳机工厂，这样做 就有些不太现实（具体的一个个工厂类就太多了）。**也就是这些一系列有关联关系的产品 应该放在一个工厂进行创建的。** 毕竟有关联关系的产品太多了，不可能各个产品都创建相对应的工厂（工厂类将会极为庞大）。

**解决方法：将工厂方法升级为抽象工厂。抽象工厂也是需要工厂方法的，抽象工厂：对一系列有关联关系的产品簇提供产品对象的统一创建。**

就可以把一组有关联关系的产品簇提供产品对象的统一创建 放到一个工厂里面做了。  
这里产品不是只有一个产品了，而是一组有关联关系的产品。提供多个产品创建的抽象接口，**在一个工厂里面：统一创建一组有关联关系的产品簇**。（而不是一个产品一个工厂，这样会造成工厂类特别多。不方便维护，那我们使用工厂的初衷也就不存在了。）具体代码如下：

```
#include <iostream>
#include <string>
#include <memory>

//一系列产品1 汽车
class Car
{
public :
	//在构造函数的初始化列表里面给name初始化
	Car(std::string name):_name(name){}

	//提供一个给派生类（具体的汽车）重写的接口
	virtual void show() = 0;//纯虚函数
protected:
	std::string _name;
};
class Bmw:public Car
{
public:
	Bmw(std::string name):Car(name){}
	//重写从基类基类继承过来的纯虚函数
	void show()
	{
		std::cout << "宝马  " << _name << std::endl;
	}
};
class Audi :public Car
{
public:
	Audi(std::string name) :Car(name) {}
	//重写从基类基类继承过来的纯虚函数
	void show()
	{
		std::cout << "奥迪  " << _name << std::endl;
	}
}; 
//一系列产品2  车灯
class CarLight
{
public:
	virtual void light() = 0;
};
class BmwLight :public CarLight
{
public:
	//重写从基类基类继承过来的纯虚函数
	void light()
	{
		std::cout << "宝马车灯  "  << std::endl;
	}
};
class AudiLight :public CarLight
{
public:
	//重写从基类基类继承过来的纯虚函数
	void light()
	{
		std::cout << "奥迪车灯  " << std::endl;
	}
};
//工厂方法 升级为 抽象工厂
//抽象工厂：对有一组关联关系的产品簇提供产品对象的统一创建
class AbstractFactory
{
public:
	//下面是工厂方法,提供多个产品创建的抽象接口
	//车子产品方法 创建车子
	virtual Car* createOneCar(std::string) = 0;
	//车灯产品方法 创建车灯
	virtual CarLight* createCarLight() = 0;
};
//下面是宝马工厂，负责创建宝马系列的产品
class BMWAbstractFactory :public AbstractFactory
{
	Car* createOneCar(std::string name)//创建车子
	{
		return new Bmw(name);
	}
	CarLight* createCarLight()//创建车灯
	{
		return new BmwLight();
	}
};
//下面是奥迪工厂，负责创建奥迪系列的产品
class AUDIAbstractFactory :public AbstractFactory
{
	Car* createOneCar(std::string name)//创建车子
	{
		return new Audi(name);
	}
	CarLight* createCarLight()//创建车灯
	{
		return new AudiLight();
	}
};

/*
这里相较于简单工厂：使用一个工厂，通过传入不同的标识 来创建不同的对象
相当于给了一个工厂的基类，然后通过实现具体的产品的工厂。用这个具体产品的工厂
来创建具体的产品对象。

*/
int main()
{
	std::unique_ptr<AbstractFactory> bmwAbstractFactory
	(new BMWAbstractFactory());
	std::unique_ptr<AbstractFactory> audiAbstractFactory
	(new AUDIAbstractFactory());
	std::unique_ptr < Car> p1(bmwAbstractFactory->createOneCar("x1"));
	std::unique_ptr < Car> p2(audiAbstractFactory->createOneCar("A5"));

	std::unique_ptr<CarLight>p3(bmwAbstractFactory->createCarLight());
	std::unique_ptr<CarLight>p4(audiAbstractFactory->createCarLight());
	p1->show();
	p2->show();

	p3->light();
	p4->light();
	return 0;
}
```

运行截图如下：  
![](https://img-blog.csdnimg.cn/20190801001105588.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
这样做，工厂类也就大大减少了。但是抽象工厂真的是完美的吗？  
其实其缺点也是存在的，如下：  
假如宝马工厂比别的汽车工厂多做了一项业务，其他工厂没有。如果我们直接在 类 AbstractFactory 里面加上这项业务 不太合适。  
![](https://img-blog.csdnimg.cn/20190801001805774.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
这个是基类，在里面添加一项 纯虚函数显然是不合理的。加了的话，则所有工厂类都要去重写这个纯虚函数（必须得重写，不然这个派生类就成了一个抽象类了）。

总结：  
![](https://img-blog.csdnimg.cn/20190801002543870.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)

第五节：代理模式
========

结构型模式：（不是关注于对象的产生，关注于最后通过类与类的组合之后，功能上该怎么使用？以及对问题场景的符合与否？） **这些设计模式关注于类和对象的组合。继承的概念被用来组合接口和定义组合对象获得新功能的方式。**

代理模式：Proxy Pattern 。主要体现的是 对象访问权限的控制。这个 Proxy 代理可以把那些访问对象的权限不够的用户 都给挡回去。  
![](https://img-blog.csdnimg.cn/20190801004154854.png)

```
//代理模式
class VedioWebSite//抽象类
{
public:
	virtual void freeMovie() = 0;//免费电影
	virtual void vipMovie() = 0;//VIP电影
	virtual void ticketMovie() = 0;//VIP + 券电影

};

//实际的 操作视频服务器后台的所有电影
class OperatorMovieBoss:public VedioWebSite  //这个是 委托类
{
public:
	virtual void freeMovie()//免费电影
	{
		std::cout << "你可以看免费电影" << std::endl;
	}
	virtual void vipMovie() //VIP电影
	{
		std::cout << "你可以看VIP电影" << std::endl;
	}
	virtual void ticketMovie()//VIP + 券电影
	{
		std::cout << "你可以看VIP + 券电影" << std::endl;
	}
};
```

OperatorMovieBoss 是委托类，相当于那个老板，拥有各种各样的功能。但是能不能调用这个老板的功能（委托类的方法），是要通过代理类来控制一下对委托类的访问权限的。  
如果 OperatorMovieBoss 不被委托，就会出现如下问题：

```
int main()
{
	VedioWebSite* p = new OperatorMovieBoss();
	p->freeMovie();
	p->ticketMovie();
	p->vipMovie();

	delete p;
	return 0;
}
```

![](https://img-blog.csdnimg.cn/2019080101023384.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
这个把视频网站的三种电影都给看了。但是视频网站已经对 不同的用户分了不同的角色，这三种用户是不一样的。（**分为 3 种用户：普通用户、VIP 用户、VIP 有券用户**）可是 如上，是没有进行判断身份的，所以都可以看了。但是到处都加上判断就会显得非常麻烦。（如果只是普通用户，是不可以调用这个对象的 vipMovie 这个方法的。）

具体实现如下：

```
#include <iostream>
#include <string>
#include <memory>


//代理模式
class VedioWebSite//抽象类
{
public:
	virtual void freeMovie() = 0;//免费电影
	virtual void vipMovie() = 0;//VIP电影
	virtual void ticketMovie() = 0;//VIP + 券电影
};

//实际的 操作视频服务器后台的所有电影
class OperatorMovieBoss:public VedioWebSite  //这个是 委托类
{
public:
	virtual void freeMovie()//免费电影
	{
		std::cout << "你可以看免费电影" << std::endl;
	}
	virtual void vipMovie() //VIP电影
	{
		std::cout << "你可以看VIP电影" << std::endl;
	}
	virtual void ticketMovie()//VIP + 券电影
	{
		std::cout << "你可以看VIP + 券电影" << std::endl;
	}
};
//代理OperatorMovieBoss的代理类
class FreeMovieProxy :public VedioWebSite//免费电影代理类
{
public:
	FreeMovieProxy()
	{
		//指针指向代理的对象
		pvedio = new OperatorMovieBoss();
	}
	~FreeMovieProxy()
	{
		delete pvedio;
	}
	virtual void freeMovie()
	{
		//通过代理对象的freeMovie来访问真正委托类对象的freeMovie方法
		pvedio->freeMovie();
	}
	virtual void vipMovie() //VIP电影
	{
		std::cout << "你只是普通用户，不可以看VIP电影,请升级为会员" 
		<< std::endl;
	}
	virtual void ticketMovie()//VIP + 券电影
	{
		std::cout <<"你没有电源券，不可以观看电影,请先升级为会员然后购买影券" 
		<< std::endl;
	}
private:
	//这里需要一个组合的对象
	/*
	基类指针指向派生类对象，只是这里不直接指向委托类对象
	而是直接访问代理对象，用代理对象来控制用户对委托类对象
	访问权限的问题。既然也是从抽象类继承来的，3个纯虚函数都要去
	重写一下。不然这个代理类也成了抽象类
	*/
	VedioWebSite* pvedio;
};
//代理OperatorMovieBoss的代理类
class VIPMovieProxy :public VedioWebSite//VIP电影代理类
{
public:
	VIPMovieProxy()
	{
		//指针指向代理的对象
		pvedio = new OperatorMovieBoss();
	}
	~VIPMovieProxy()
	{
		delete pvedio;
	}
	virtual void freeMovie()
	{
		//通过代理对象的freeMovie来访问真正委托类对象的freeMovie方法
		pvedio->freeMovie();
	}
	virtual void vipMovie() //VIP电影
	{
		pvedio->vipMovie();
	}
	virtual void ticketMovie()//VIP + 券电影
	{
		std::cout<<"你没有电源券，不可以观看电影,请先购买影券"<<std::endl;
	}
private:
	//这里需要一个组合的对象
	/*
	基类指针指向派生类对象，只是这里不直接指向委托类对象
	而是直接访问代理对象，用代理对象来控制用户对委托类对象
	访问权限的问题。既然也是从抽象类继承来的，3个纯虚函数都要去
	重写一下。不然这个代理类也成了抽象类
	*/
	VedioWebSite* pvedio;
};
void WhoWatchMovie(std::unique_ptr<VedioWebSite>& ptr)
{
	ptr->freeMovie();
	ptr->vipMovie();
	ptr->ticketMovie();
}
int main()
{
	//基类指针 指向 普通用户级别的代理类
	std::unique_ptr<VedioWebSite> p1(new FreeMovieProxy());
	WhoWatchMovie(p1);
	std::cout << "*****************************" << std::endl;
	//基类指针 指向 VIP用户级别的代理类
	std::unique_ptr<VedioWebSite> p2(new VIPMovieProxy());
	WhoWatchMovie(p2);

	return 0;
}
```

![](https://img-blog.csdnimg.cn/20190801014130388.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
这样就可以 不同身份的用户可以通过访问代理对象，就可以对最终的 OperatorMovieBoss 类的方法的调用。因为电源能不能看，以及三种用户怎么看的功能都是在 OperatorMovieBoss 委托类里面实现的。因为不是所有的用户都能够直接去访问 OperatorMovieBoss 对象，所以给三种身份的用户都提供了 不同的代理类。**代理类和委托类之间是一个 组合的关系。都是从抽象类继承而来，主要就是因为在代码上用的都是 (多态) 基类指针指向派生类对象** 通过成员变量 指针 pvedio 在构造函数里面指向了一个委托类对象，通过代理对象的 freeMovie 虚函数方法来访问真正委托类对象的 freeMovie 方法 虚函数方法。

```
std::unique_ptr<VedioWebSite> p1(new FreeMovieProxy());
std::unique_ptr<VedioWebSite> p2(new VIPMovieProxy());
```

因为这里使用的是代理类对象 替换原来的委托类对象，而且还想继续让抽象类（基类）指针指向。**原来基类的指针指向的是委托类对象，现在是直接指向了代理类对象**。所以这个代理类也只能从这个基类继承而来。这就是结构型模式：关注于类和对象的组合。

总结：**代理模式涉及到了 抽象类（公共类），委托类（需要从抽象类继承而来），代理类（需要从抽象类继承而来），以组合的方式 使用代理对象，在实际的使用中客户直接访问的是代理对象。**

也可以做如下更改：

```
#include <iostream>
#include <string>
#include <memory>


//代理模式
class VedioWebSite//抽象类
{
public:
	virtual void freeMovie() = 0;//免费电影
	virtual void vipMovie() = 0;//VIP电影
	virtual void ticketMovie() = 0;//VIP + 券电影
};

//实际的 操作视频服务器后台的所有电影
class OperatorMovieBoss:public VedioWebSite  //这个是 委托类
{
public:
	virtual void freeMovie()//免费电影
	{
		std::cout << "你可以看免费电影" << std::endl;
	}
	virtual void vipMovie() //VIP电影
	{
		std::cout << "你可以看VIP电影" << std::endl;
	}
	virtual void ticketMovie()//VIP + 券电影
	{
		std::cout << "你可以看VIP + 券电影" << std::endl;
	}
};
//代理OperatorMovieBoss的代理类
class FreeMovieProxy :public VedioWebSite//免费电影代理类
{
public:
	virtual void freeMovie()
	{
		//通过代理对象的freeMovie来访问真正委托类对象的freeMovie方法
		freemovieproxy.freeMovie();
	}
	virtual void vipMovie() //VIP电影
	{
		std::cout << "你只是普通用户，不可以看VIP电影,请升级为会员" 
		<< std::endl;
	}
	virtual void ticketMovie()//VIP + 券电影
	{
		std::cout << "你没有电源券，不可以观看电影,请先升级为会员然后购买影券"
		 << std::endl;
	}
private:
	OperatorMovieBoss freemovieproxy;//直接定义一个委托类对象
};
//代理OperatorMovieBoss的代理类
class VIPMovieProxy :public VedioWebSite//VIP电影代理类
{
public:
	VIPMovieProxy()
	{
		//指针指向代理的对象
		pvedio = new OperatorMovieBoss();
	}
	~VIPMovieProxy()
	{
		delete pvedio;
	}
	virtual void freeMovie()
	{
		//通过代理对象的freeMovie来访问真正委托类对象的freeMovie方法
		pvedio->freeMovie();
	}
	virtual void vipMovie() //VIP电影
	{
		pvedio->vipMovie();
	}
	virtual void ticketMovie()//VIP + 券电影
	{
		std::cout << "你没有电源券，不可以观看电影,请先购买影券" 
		<< std::endl;
	}
private:
	//这里需要一个组合的对象
	/*
	基类指针指向派生类对象，只是这里不直接指向委托类对象
	而是直接访问代理对象，用代理对象来控制用户对委托类对象
	访问权限的问题。既然也是从抽象类继承来的，3个纯虚函数都要去
	重写一下。不然这个代理类也成了抽象类
	*/
	VedioWebSite* pvedio;
};
/*
下面的都是通用的函数接口，使用的都是基类的指针或者引用
不用关心ptr值的是委托类还是代理类对象。通过多态，访问对象的
虚函数表访问相应的虚函数即可。
*/
void WhoWatchMovie(std::unique_ptr<VedioWebSite>& ptr)
{
	ptr->freeMovie();
	ptr->vipMovie();
	ptr->ticketMovie();
}
int main()
{
	//基类指针 指向 普通用户级别的代理类
	std::unique_ptr<VedioWebSite> p1(new FreeMovieProxy());
	WhoWatchMovie(p1);
	std::cout << "*******************************" << std::endl;
	//基类指针 指向 VIP用户级别的代理类
	std::unique_ptr<VedioWebSite> p2(new VIPMovieProxy());
	WhoWatchMovie(p2);

	return 0;
}
```

FreeMovieProxy 类的成员变量是一个委托类对象，委托类对象自己调用自己的方法。  
VIPMovieProxy 类的成员变量是基类的指针，通过指针的方式 来访问委托类方法。

当然上面的第一种就不如第二种灵活：因为第一种只能去代理这一种委托类对象了，不可以再去代理其他的委托类对象。第二种可以继续通过共同的基类指针指向其他的委托类对象（当然也是从基类继承来的）。  
当然这两种也是不影响代码的结果：  
![](https://img-blog.csdnimg.cn/20190801092754734.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)

第六节：装饰器模式
=========

也属于结构型模式的一种：Decorator Pattern 。在代码结构上和代理模式很相似  
![](https://img-blog.csdnimg.cn/20190801100228586.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
代理模式：从抽象类里面派生出一个具体的委托类，但是用户是不可以直接使用这个委托类的。因为委托类里面包含了整个系统额所有功能，但是这并意味着用户可以直接得到这些功能服务，需要对其访问的权限进行控制。同样从基类派生出 代理类，用户直接访问的是代理类。在代理类里面，它有一个基类的指针，指向被委托的委托类对象。因为代理类和委托类都是从基类继承而来的，其中的方法都是重写了抽象类里面定义的纯虚函数的了。代理类最终 如果可以访问到委托类对象则在其方法里面，调用的还是委托类对象的同名方法；如果代理类发现权限不够，则不调用委托类对象的相应的同名方法。  
![](https://img-blog.csdnimg.cn/20190801101349172.png)  
![](https://img-blog.csdnimg.cn/20190801101658794.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
现在为了增加这一个功能，就需要在代码上 每个工厂类共增加三个逻辑功能一致的新类（且这个类的逻辑功能和具体的汽车没有太大关系）。假如说，如果给这三种汽车再增加一个自动刹车功能，则需要再整上 3 个新类（且这个类的逻辑功能和具体的汽车也没有太大关系）。

每次一增加一个新的功能，就得去给这 3 个实体类增加一个派生类。如上，就 2 个功能，最后增加了 6 个类。产生的问题如下：  
![](https://img-blog.csdnimg.cn/20190801102451823.png)  
通过子类扩展功能如下：  
![](https://img-blog.csdnimg.cn/20190801102600704.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
通过使用装饰器模式增强功能如下：  
![](https://img-blog.csdnimg.cn/20190801103001546.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
从 Car 这个类里面继承来一个 派生类 CarDecorator 。（为了代码不做改变，都是通过基类指针指向派生类对象的）在 CarDecorator 里面，需要持有一个 Car 指针（需要一个被装饰的对象，这个基类指针可以指向任何的派生类对象的）。然后从 CarDecorator 去扩展更多的功能（定义具体的功能性装饰器）。也就是说 增加几个功能就增加几个类。而不用去给所有的实体类都增加一遍 这些功能类。

在这里 那几个具体的功能性装饰器类最终访问的也是汽车实体类的方法，但是通过访问其方法，还要去其方法进行功能性的扩展。

```
#include <iostream>
#include <string>
#include <memory>

using namespace std;
//装饰器模式
class Car//抽象基类
{
public :
	virtual void show() = 0;
};
//下面是三个 汽车实体类
class Bmw :public Car
{
public:
	void show()
	{
		cout << "这是宝马汽车的配置为:" ;
	}
};
class Audi :public Car
{
public:
	void show()
	{
		cout << "这是奥迪汽车的配置为:" ;
	}
};
class Bnze :public Car
{
public:
	void show()
	{
		cout << "这是奔驰汽车的配置为:" ;
	}
};
/*进行功能增强了，如下：*/
class CarBaseDecorator:public Car//装饰器的基类
{
public:
	//装饰器里面，先把基本的功能装饰写好，给谁添加 先不重要
	
	//装饰器的基类，具体的功能装饰还没有出来呢
	virtual void show() = 0;
};

/*
其实对于装饰器基类，如果要有一些装饰器公共的方法的话，
可以在装饰器的基类里面进行实现。
这里各个装饰器类没有一些公有的方法需要在统一地在基类书写的
情况下。

例如这里 就没有公共的方法，这种情况下就可以把装饰器的
基类CarBaseDecorator给省略掉。直接让下面的CarChildDecorator们 
直接继承于Car类。

但是我选择不省略
*/

//功能装饰器1：定速巡航功能
class CarChildDecorator1 :public CarBaseDecorator
{
public:
	CarChildDecorator1(Car*p)
	{
		_ptr = p;
	}
	void show()
	{
		_ptr->show();
		cout << " 定速巡航功能";
	}
private:
	Car* _ptr;
};

//功能装饰器2：自动驾驶功能
class CarChildDecorator2 :public CarBaseDecorator
{
public:
	CarChildDecorator2(Car* p)
	{
		_ptr = p;
	}
	void show()
	{
		_ptr->show();
		cout << " 自动驾驶功能";
	}
private:
	Car* _ptr;
};
//功能装饰器3：车道偏离功能
class CarChildDecorator3 :public CarBaseDecorator
{
public:
	CarChildDecorator3(Car* p)
	{
		_ptr = p;
	}
	void show()
	{
		_ptr->show();
		cout << " 车道偏离功能";
	}
private:
	Car* _ptr;
};
int main()
{
	Car* p1 = new CarChildDecorator1(new Bmw());
	p1 = new CarChildDecorator2(p1);
	p1= new CarChildDecorator3(p1);
	p1->show();
	cout << endl;
	cout << "************************************" << endl;
	Car* p2 = new CarChildDecorator2(new Audi());
	p2->show();
	cout << endl;
	cout << "************************************" << endl;
	Car* p3 = new CarChildDecorator3(new Bnze());
	p3->show();
	cout << endl;
	return 0;
}
```

![](https://img-blog.csdnimg.cn/20190801115426693.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
装饰器模式 可以以比较小的代价，增强类的功能。和代理模式很相似，在具体的功能装饰类里面（不是装饰器基类）用一个指针 来保留所装饰对象的信息。最终通过访问装饰器方法的时候，还是访问具体对象的方法。访问完方法之后，再在装饰器类里面添加新的功能。而且这样的话，还可以实现任意功能的组装。

**本节比较让人恶心的一点：那个装饰器基类 最好不要用了。直接从基类 Car 继承来就可以了。但是为了以后的拓展，最好还是学习一下，这个装饰器基类的使用。**  
至于最后的函数调用，真的特别有意思。希望大家在学习的时候，可以在这里 打开调试，看一下代码运行流程。这个将会对 继承与多态 大有帮助。

第七节：适配器模式
=========

设计模式除了比较重要的单例模式，工厂方法之外，比较重要的就是适配器模式了。  
例如，在项目中 使用到第三方的插件或者库，但是因为**接口不兼容** 就需要去添加很多的适配器类。当然解决方法还有 重构第三方的库，重新设计统一的接口去使用，但是这个明显成本和难度就很大了。  
![](https://img-blog.csdnimg.cn/2019080122035562.png)  
例如场景：电脑使用一个**接口**，把桌面演示投影到 投影仪上。  
假如：现在有三种类型的接口：VGA HDMI TypeC 。

（1）接口兼容的情况有：电脑和投影仪都是同一种 VGA 接口。这是最简单的一种情况 如下：

```
//适配器模式：让不兼容的接口可以在一起工作
//电脑 = 》   投影到 = 》   投影仪上   VGA  HDMI  TypeC

//VGA接口的电脑，(TV)投影仪也是VGA接口
class VGA // VGA接口   抽象类
{
public:
	virtual void play() = 0;
};
// TV01表示支持VGA接口的投影仪
class TV01 : public VGA
{
public:
	void play()
	{
		cout << "通过VGA接口连接投影仪，进行视频播放" << endl;
	}
};

// 实现一个电脑类(只支持VGA接口)
class Computer
{
public:
	// 由于电脑只支持VGA接口，所以该方法的参数也只能支持VGA接口的指针/引用
	void playVideo(VGA* pVGA)
	{
		pVGA->play();
	}
};
int main()
{
	Computer computer;
	computer.playVideo(new TV01());	
	return 0;
}
```

![](https://img-blog.csdnimg.cn/20190802082336660.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
（2）第二种情况：  
进了一批新的投影仪，但是新的投影仪都是只支持 HDMI 接口。但是 电脑还是只支持 VGA 接口。此时 投影仪自带的连接线的接口 和 电脑的这个接口就不兼容了。

```
// 进了一批新的投影仪，但是新的投影仪都是只支持HDMI接口
class HDMI
{
public:
	virtual void play() = 0;
};
class TV02 : public HDMI
{
public:
	void play()
	{
		cout << "通过HDMI接口连接投影仪，进行视频播放" << endl;
	}
};
```

所以这下，就不能够 直接在参数调用中 如下操作：

```
Computer computer;
	computer.playVideo(new TV02());
```

不可以这样做，因为第二行的 参数传递不可以传入一个 TV02 的类型对象指针。由于电脑只支持 VGA 接口，所以该 playVideo 方法的参数也只能支持 VGA 接口的指针 / 引用。然后使用基类指针 指向派生类对象。

那么如果解决接口不兼容呢？  
方法 1：换一个支持 HDMI 接口的电脑，这个就叫代码重构。成本高，而且在实际的项目开发中，已经开发了大量的组件。这些组件还是使用着大量的旧接口，代码重构所以更换起来非常麻烦。  
方法 2：买一个转换头（适配器），转换头 VGA 一头插在电脑上，另一头插在投影仪自带的连接线的接口 上，这样就能够把 VGA 信号转成 HDMI 信号，这个叫添加适配器类。这样旧接口和新接口就可以在一起去工作了。

此时的电脑还是只需要支持 VGA 接口即可，至于投影仪自带的连接线的接口 是哪种类型 不需要关系。因为这些是封装在转换头里面了，它负责 VGA 的一头插在电脑上，另一头插在投影仪自带的连接线的接口 上，这样就能够把 VGA 信号转成 HDMI 信号。

由于电脑（VGA 接口）和投影仪（HDMI 接口）无法直接相连，所以需要添加适配器类。**这个类是一定要支持旧接口的，所以必须从旧类 VGA 继承过来。** 如下：

```
#include <iostream>
#include <string>
#include <memory>

using namespace std;
//适配器模式：让不兼容的接口可以在一起工作
//电脑 = 》   投影到 = 》   投影仪上   VGA  HDMI  TypeC

//VGA接口的电脑，(TV)投影仪也是VGA接口
class VGA // VGA接口   抽象类
{
public:
	virtual void play() = 0;
};
// TV01表示支持VGA接口的投影仪
class TV01 : public VGA
{
public:
	void play()
	{
		cout << "通过VGA接口连接投影仪，进行视频播放" << endl;
	}
};

// 实现一个电脑类(只支持VGA接口)
class Computer
{
public:
	// 由于电脑只支持VGA接口，所以该方法的参数也只能支持VGA接口的指针/引用
	void playVideo(VGA* pVGA)
	{
		pVGA->play();
	}
};
/*
方法1：换一个支持HDMI接口的电脑，这个就叫代码重构
方法2：买一个转换头（适配器），能够把VGA信号转成HDMI信号，这个叫添加适配器类
*/

// 进了一批新的投影仪，但是新的投影仪都是只支持HDMI接口
class HDMI
{
public:
	virtual void play() = 0;
};
class TV02 : public HDMI
{
public:
	void play()
	{
		cout << "通过HDMI接口连接投影仪，进行视频播放" << endl;
	}
};


// 由于电脑（VGA接口）和投影仪（HDMI接口）无法直接相连，所以需要添加适配器类
class VGAToHDMIAdapter : public VGA
{
public:
	VGAToHDMIAdapter(HDMI* p) :pHdmi(p) {}
	void play() // 该方法相当于就是转换头，做不同接口的信号转换的
	{
		pHdmi->play();
	}
private:
	HDMI* pHdmi;
};
int main()
{
	Computer computer;
	//computer.playVideo(new TV01());
	//不可以直接在里面传入TV02对象的指针，所以需要加上一层转换适配
	//因为playVideo方法接收的还是VGA接口，而适配器也是从VGA 继承而来的
	//适配器类的构造函数接收的是HDMI接口的对象指针 所以这里传入的是newTV02()对象指针
	computer.playVideo(new VGAToHDMIAdapter(new TV02()));
	return 0;
}
```

![](https://img-blog.csdnimg.cn/20190802181301633.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)  
不可以直接在里面传入 TV02 对象的指针，所以需要加上一层转换适配。因为 playVideo 方法接收的还是 VGA 接口，而适配器也是从 VGA 继承而来的。VGAToHDMIAdapter 适配器类的构造函数接收的是 HDMI 接口的指针 所以这里传入的是 newTV02() 对象指针。（基类指针指向派生类对象，pHdmi 就指向的是这个新生成的 newTV02() 对象）当 computer 调用 playVideo 方法的时候，在 computer 类里面调用的是 VGA 的 play 方法。根据多态，这个 playVideo 方法接收的是一个适配器对象指针，最后就调用的是适配器类的 play 方法。那么底层就是通过转换调用的是，通过 pHdmi 指针调用的是 HDMI 的 play 方法。

第八节：观察者模式
=========

也即：观察者 - 监听者模式（发布订阅模式），隶属于行为型模式。  
行为型模式：主要关注的是对象之间的通信，例如：对象 A 调用对象 B 的成员方法等。

观察者 - 监听者模式：主要处理 对象的一对多的关系，也就是多个对象都依赖于一个对象。**当该对象的状态发生改变时，其他对象都可以接受到相应的通知**。假如现在一个类表示了一组数据，生成一个对象。通过这同一个数据对象， 可以得到一组 圆饼图、柱状图、条形图等。  
![](https://img-blog.csdnimg.cn/20190802175823390.png)  
**此时的图形对象 依赖于 数据对象，当然 数据对象发生改变的时候，这 3 个图形对象也应该及时的接收通知，然后图形上进行调整。这就是适用于发布订阅模式的典型示例。**

这里的**通知**：并不是要求所有的图形对象，自己去观察 所依赖的数据对象（很麻烦）。而是由所发生改变的数据对象，进行广播。这也就是为什么也叫做 发布订阅模式，由图形对象进行订阅 数据对象是否发生改变，当数据对象发生改变的时候，由数据对象自己发布即可（进行通知）。  
![](https://img-blog.csdnimg.cn/20190802180759313.png)  
![](https://img-blog.csdnimg.cn/20190802181013902.png)

```
#include <iostream>
#include <string>
#include <unordered_map>
#include <list>
using namespace std;
//发布订阅模式
//观察者接收到消息之后，就要去处理消息，更新图像
class Observer //观察者抽象类
{
public:
	//统一的处理事件,messageid表示消息    回调函数
	virtual void upData(int messageid) = 0;
};

//以下全部是观察者实例
class Observer1 :public Observer
{
public:
	void upData(int id)
	{
		switch (id)
		{
		case 1:
			cout << "观察者1 对1 和 2号消息感兴趣.1号消息发生改变" << endl;
			break;
		case 2:
			cout << "观察者1 对1 和 2号消息感兴趣.2号消息发生改变" << endl;
			break;
		}
	}
};
class Observer2 :public Observer
{
public:
	void upData(int id)
	{
		switch (id)
		{
		case 1:
			cout << "观察者2 对1 和 3号消息感兴趣.1号消息发生改变" << endl;
			break;
		case 3:
			cout << "观察者2 对1 和 3号消息感兴趣.3号消息发生改变" << endl;
			break;
		}
	}
};
class Observer3 :public Observer
{
public:
	void upData(int id)
	{
		switch (id)
		{
		case 2:
			cout << "观察者3 对2 和 3号消息感兴趣.2号消息发生改变" << endl;
			break;
		case 3:
			cout << "观察者3 对2 和 3号消息感兴趣.3号消息发生改变" << endl;
			break;
		}
	}
};

//主题类
//需要存储一下 每个Observer对哪个id感兴趣
class Subject
{
public :
	//向list里面添加一个Observer 还有其感兴趣消息id
	//给主题添加观察者对象
	void addObserver(Observer*obser, int messageid)
	{
		//mapping[messageid].push_back(obser);
		/*
		上面的这一句话
		map的【】运算符重载函数
		messageid存在 则返回键对应值的引用，插入到list。
		键不存在的话，则默认插入一对。（新增加一个messageid，其值为
		默认构造的list，把obser添加进去）
		*/
		unordered_map<int, list<Observer*>>::iterator it 
		= mapping.find(messageid);
		if (it != mapping.end())
		{
			it->second.push_back(obser);
		}
		else
		{
			list<Observer*>mylist;
			mylist.push_back(obser);
			mapping.insert({ messageid, mylist });
		}
	}

	//主题发生改变，给相应的观察者进行广播
	void dispatch(int messageid)
	{
		//看看  谁对这个消息主题感兴趣
		auto it = mapping.find(messageid);
		if (it != mapping.end())
		{
			//有人对这个消息感兴趣，进行查看list 分发
			for (Observer* pObserver : it->second)
			{
				pObserver->upData(messageid);//最核心的代码
			}
		}
	}
private:
	//消息id 具体感兴趣的Observer
	//但是对于一个id的消息，可能有多个Observer
	//所以 很多个观察者串成一个列表
	unordered_map<int, list<Observer*>>mapping;
};
int main()
{
	Subject subject;//消息主题对象

	//观察者1 对1 和 2号消息感兴趣
	subject.addObserver(new Observer1(), 1);
	subject.addObserver(new Observer1(), 2);
	
	//观察者2 对1 和 3号消息感兴趣
	subject.addObserver(new Observer2(), 1);
	subject.addObserver(new Observer2(), 3);
	
	//观察者3 对2 和 3号消息感兴趣
	subject.addObserver(new Observer3(), 2);
	subject.addObserver(new Observer3(), 3);
	cout << "************************************" << endl;

	int changeMessid;
	cout << "请输入一个发生改变的消息  输入-1结束" << endl;
	cout << "哪个消息改变了：";
	while (cin >> changeMessid && changeMessid != -1)
	{
		subject.dispatch(changeMessid);
		cout << "哪个消息改变了：";
	}
	return 0;
}
```

![](https://img-blog.csdnimg.cn/20190802193024417.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk0OTUzNQ==,size_16,color_FFFFFF,t_70)