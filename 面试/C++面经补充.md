# C++面经补充

## 1 单例模式

- 含义：保证一个类仅可以有一个实例化对象，并且提供一个可以访问它的全局接口；
- 应用场景：日志模块、数据库模块
- 饿汉单例模式：还没有访问实例对象的时候，实例对象就已经事先产生了；在进入main函数前，就产生好了
- 懒汉单例模式：唯一的实例对象，直到第一次访问它的时候才产生

```cpp
1.构造函数私有化：类外不能新建对象。
2.定义一个唯一的实例化对象：私有的静态成员变量
3.提供访问唯一实例对象的接口：公共的静态成员方法（无需对象就能访问）

懒汉单例模式
在第一次用到类实例的时候才会去实例化一个对象
要考虑线程安全问题：因为用的时候才生成对象，但对象的构造又是一个非常复杂的过程，不是一个原子的操作。
#include <iostream>
#include <mutex>
using namespace std;

mutex mymutex;  // 定义全局互斥锁
class singleton{
public:
    // 静态成员方法，访问唯一实例对象的接口
    static singleton* getInstence() {
        if(_instence == nullptr) {
            // 双重if + 互斥锁 保证线程安全
            lock_guard<mutex> guard(mymutex);
            if(_instence == nullptr) {
                _instence = new singleton();
            }
        }
        return _instence;
    }
private:
    // 静态成员变量
    static singleton* volatile _instence;
    // 构造函数私有化
    singleton() {}
    // 禁用拷贝构造函数和赋值操作符函数
    singleton(const singleton &) = delete;
    singleton& operator=(const singleton &) = delete;
};
// 静态成员变量类内定义，类外初始化
singleton* volatile singleton::_instence = nullptr; 

int main() {
    singleton *p1 = singleton::getInstence();
    singleton *p2 = singleton::getInstence();
    singleton *p3 = singleton::getInstence();
    cout << p1 << '\n' << p2 << '\n' << p3 << endl;
}

饿汉单例模式
一定是线程安全的。因为多线程的执行是要先执行函数的，线程函数启动才标志着这个线程开始执行的。
静态的成员变量在数据段，数据段在整个程序启动 main 函数之前就创建和初始化好的了。
缺点：主函数之前就完成了唯一对象的实例化，可是万一我整个过程中都没用到该实例化对象呢？这就造成了程序额外的开销，根本就没必要实例化一个不会用到的对象。
class singleton{
public:
    static singleton* getInstence() {
        return &_instence;
    }
private:
    static singleton _instence;
    singleton() {}
    singleton(const singleton &) = delete;
    singleton& operator=(const singleton &) = delete;
};
singleton singleton::_instence;

函数局部对象实现的懒汉单例模式
函数的静态的局部对象程序启动阶段，其内存（数据段）就有了，但是静态的对象 第一次初始化是在第一次运行这个对象的时候才进行的初始化，所以在不调用getInstance方法的时候，这个对象是不构造的，就不会去调用构造函数：做复杂的初始化动作。所以这个也是懒汉式单例模式
线程安全吗？
对于 static 静态局部对象的初始化，编译器会自动对它的初始化进行加锁和解锁控制，使静态局部对象的初始化成为线程安全的操作
class singleton {
public:
    static singleton* getInstence() {
        static singleton _instence;
        return &_instence;
    }
private:
    singleton() {}
    singleton(const singleton &) = delete;
    singleton& operator=(const singleton &) = delete;
};
```